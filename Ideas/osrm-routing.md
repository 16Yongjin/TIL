# OSRM으로 장소 간 이동거리, 시간 구하기 구하기

![image](https://user-images.githubusercontent.com/22253556/95537287-4525b200-0a28-11eb-8a30-bd585157dfaa.png)

출처: [OSRM Project](http://map.project-osrm.org/?z=11&center=33.385586%2C126.654167&loc=33.331102%2C126.216431&loc=33.499033%2C126.538467&hl=en&alt=0&srv=1)

네이버나 구글 지도 API로 두 장소 간 경로의 거리와 이동시간을 구할 수 있다.

하지만 문제는 비용. 1건 당 5원 정도 비용이 든다.

장소가 많으면 많을수록 두 장소 간 경로 수는 제곱으로 늘어난다.

장소가 1000개면 경로는 백만 개이므로 API를 사용하면 500만 원이 든다.

하지만, [오픈 스트리트 맵](https://www.openstreetmap.org/)과 [오픈 소스 라우팅 머신](https://github.com/Project-OSRM/osrm-backend/)를 활용해 약간의 수고들 들이면 비용이 무료다.

## 1. 지도 준비하기

[Geofabrik](http://download.geofabrik.de/asia/south-korea.html)에서 한국 지도인 `south-korea-latest.osm.pbf` 파일을 다운로드한다.

![image](https://user-images.githubusercontent.com/22253556/95537144-e6603880-0a27-11eb-93f1-5c160bf41ced.png)

## 2. 도커 설치

[도커 허브](https://hub.docker.com/editions/community/docker-ce-desktop-windows/)에서 도커를 다운로드하고 설치한다.

## 3. 지도 파일 전처리

`south-korea-latest.osm.pbf` 파일이 있는 디렉터리에서 다음 명령어를 실행한다.

```
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/south-korea-latest.osm.pbf
```

`-v "${PWD}:/data"` 플래그는 도커 컨테이너의 `/data` 디렉터리를 만들고 현재 작업 중인 경로인 `${PWD}`를 사용할 수 있게 한다.

컨테이너의 `/data/south-korea-latest.osm.pbf` 파일은 호스트의 `"${PWD}/south-korea-latest.osm.pbf"`를 가리키게 된다.

`/opt/car.lua` 플래그로 차량이 다닐 수 있는 길만 가져온다. `car` 대신 `bicycle`이나 `foot` 옵션을 넣을 수 있다.

다음 두 명령어도 실행한다.

```
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/south-korea-latest.osrm
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/south-korea-latest.osrm
```

## 4. 경로 탐색 HTTP 서버 실행하기

```
docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld /data/berlin-latest.osrm
```

## 5. HTTP 요청 보내기

```
curl http://localhost:5000/route/v1/driving/126.3731575012207,33.38400957065941;126.33230209350586,33.384296245188274
```

경도, 위도 순으로 데이터를 보낸다.

그러면 다음과 같은 결과가 나온다. 여기서 `routes[0].distance`와 `routes[0].duration` 데이터를 가져와서 쓴다.

```json
{
  "code": "Ok",
  "routes": [
    {
      "geometry": "cjwjEkaibWcWkFkCrJiK`Iyb@fMyLnIkIvAaEzKeJvIgGl@gF|BcVr[sGzNiEnEi@nHgK`LkBjWoJ~FjIhIrGjDf[vIhMrFdv@xo@~PzKrMjFfItBlCmQbOmLzBeAvFnD",
      "legs": [
        {
          "steps": [],
          "distance": 9089.2,
          "duration": 762.3,
          "summary": "",
          "weight": 762.3
        }
      ],
      "distance": 9089.2,
      "duration": 762.3,
      "weight_name": "routability",
      "weight": 762.3
    }
  ],
  "waypoints": [
    {
      "hint": "JgkKgP___38GAAAAKAAAAGAAAAArAAAA1_ZvQSOEn0KyWmJD2HDJQgYAAAAoAAAAYAAAACsAAAByBAAAfUmIB_dm_QEmTYgHSmb9AQEAzwDpqsV4",
      "distance": 89.281362,
      "name": "평화로",
      "location": [126.372221, 33.384183]
    },
    {
      "hint": "TwYKgFIGCoAAAAAAJQAAAAUDAADLAAAAAAAAAHUdzUEoGgZEYeQMQwAAAAAlAAAABQMAAMsAAAByBAAAHqqHBxls_QGOrYcHaGf9ASEAjwPpqsV4",
      "distance": 156.360005,
      "name": "",
      "location": [126.331422, 33.385497]
    }
  ]
}
```

## Node.js 바인딩

Node.js에서도 사용 가능한데, 최신 버전 바이너리 파일 다운로드 서버에 이상이 있어서 못 해봤다.

## 위도와 경도를 사용해 직선거리 구하기

가까운 장소라도 도로가 없으면 차량 거리 2시간이 나올 때가 있다.

그럴 때는 직선거리를 구해 도보 이동시간을 구하고 차량과 도보 중 빠른 쪽을 선택한다.

다음 자바스크립트 함수는 두 위경도를 사용해 직선거리를 구한다.

```js
function distance(lon1, lat1, lon2, lat2, unit) {
  if (lat1 === lat2 && lon1 === lon2) return 0

  const radlat1 = (Math.PI * lat1) / 180
  const radlat2 = (Math.PI * lat2) / 180
  const theta = lon1 - lon2
  const radtheta = (Math.PI * theta) / 180
  let dist =
    Math.sin(radlat1) * Math.sin(radlat2) +
    Math.cos(radlat1) * Math.cos(radlat2) * Math.cos(radtheta)

  if (dist > 1) dist = 1
  dist = Math.acos(dist)
  dist = (dist * 180) / Math.PI
  dist = dist * 60 * 1.1515

  if (unit == "K") dist = dist * 1.609344

  if (unit == "N") dist = dist * 0.8684

  return dist
}
```

기본 단위는 마일이므로 `unit` 파라미터에 `K`를 넣으면 킬로미터 단위로 구해진다.

## 참고

[OSRM HTTP server 문서](https://github.com/Project-OSRM/osrm-backend/blob/master/docs/http.md)
