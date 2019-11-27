# Should You Use Kubernetes and Docker In Your Next Project?

출처: https://youtu.be/u8dW8DrcSmo

## Monolith 서비스를 Auth, API, Worker, FrontEnd로 쪼개면

- 개발이 빨라짐
- 한 가지 기능에 집중해서 개발하니 업데이트가 빠름
- 컴포넌트 하나가 망가지면 그 컴포넌트만 망가지고 전체 시스템에 영향이 안감

그래서 가상머신 하나에 컴포넌트 하나 올려서 서비스를 돌렸었음

VM을 쓰면 서비스의 크기는 다양한데 가상머신의 크기는 그대로니까 가상머신의 오버헤드가 커짐

VM 오버헤드가 10%라면 서버비용 10%를 그대로 날리는 셈

## 리눅스에 Control Groups와 Namespace 기능이 있음

Control Groups로 프로세스에 할당할 CPU, 메모리, 네트워크를 제한할 수 있음

Namespace로 프로세스의 권한을 설정할 수 있음

그레서 이 두 기능으로 Docker를 만들었음

도커를 사용하면

- 컨테이너가 몇 초 내로 부팅이 되고
- HW 에뮬레이션이 필요 없고
- 자원을 동적으로 할당이 가능함
- 컨테이너 안에 뭐가 들어있는 밖에서는 그저 컨테이너로만 보여서 어디서든 실행가능함

## 도커 덕분에 VM 오버헤드가 줄었지만 그래도 관리할 컨테이너가 너무 많음

그래서 쿠버네티스를 사용함

## 쿠버네티스의 장점

- 여러 컴퓨터를 하나의 클러스터로 묶어 서버를 추상화할 수 있음
- 요구자원이 다른 컨테이너들을 노드 컴퓨터에 알맞에 배분해서 실행함
- 인프라를 API로 다룰 수 있음(노드 생성, 로드 밸런싱 설정 등)

## 쿠버네티스 레이어 예시

```
인그레스(외부 로드밸런서)
   |             |
  서비스         서비스
 |  |  |          |
팟1 팟2 팟3       팟5
```

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

위 명령어로 각 레이어를 실행하고

```bash
kubectl get pods
kubectl get service
kubectl get ingress
```

위 명령어로 pods, service, ingress 상태를 확인할 수 있음

추가로, 간단한 yaml 파일 설정으로 앱 레플리카를 여러 개 띄울 수 있고

앱 버전 업데이트 시 자체적으로 롤링 업데이트를 실행해서 무중단 배포가 가능함

OpenAI에서도 쿠버네티스를 통해 노드 2500개를 모델 훈련에 사용하고 있음

일반 개발자도 256개의 TCU와 128000개의 선점형 vCPU를 클릭 몇 번으로 사용할 수 있음 (대신 사용료가 한 시간에 200만원임)
