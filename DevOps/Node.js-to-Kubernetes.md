# Node.js to Kubernetes

## Docker 설치

1. Docker Desktop 설치
2. Docker 설정의 Resources 탭에서 File Sharing 옵션 설정

## Node.js 서버 생성

### 프로젝트 생성
```
npm init -y
```

### `server.js` 생성
```javascript
const express = require('express');
const PORT = 5000;
const HOST = '0.0.0.0';
const app = express();

app.get('/', (req, res) => {
  res.send('Hello world \n');
});

app.listen(PORT, HOST);

console.log(`Running version 3 on http://${HOST}:${PORT}`);
```

## `dockerfile` 생성
```dockerfile
FROM node:12.4.0-alpine

WORKDIR /

COPY ./package.json /package.json

RUN npm install

COPY ./ /

CMD node . 
```

도커파일의 각 명령어는 레이어로 캐싱되므로 코드가 바뀌어도 디펜던시 설치 과정을 건너뛸 수 있다.

## docker 이미지 빌드

```
docker build . -t username/nodejs:v1
```

## docker 이미지 푸시

```
docker push username/nodejs:v1
```

## Kubernetes 설정

docker 설정의 Kubernetes 탭에서 Enable Kubernetes 옵션 설정

## Kubernetes 노드 확인

```
kubectl get nodes
```

Docker Desktop이 실행한 노드를 확인할 수 있음

```
NAME             STATUS  ROLES  AGE VERSION
docker-desktop	Ready	master	3D	v1.15.5
```

## Kubernetes 네임스페이스 생성

```
kubectl create ns example-app
```

네임스페이스는 쿠버네티스 클러스터내 분리 단위

네임스페이스 별로 CPU나 RAM 같은 리소스 할당량을 조절할 수 있다.

## `deployment.yaml` 생성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deploy
  labels:
    app: example-app
  annotations:
spec:
  selector:
    matchLabels:
      app: example-app
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-app
        image: aimvector/nodejs:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        resources:
          requests:
            memory: "64Mi"
            cpu: "10m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

이 deployment의 이름은 `example-deploy`이고 `example-app` 앱을 2개 띄운다.

`template/spec/container`에 실행할 이미지와 포트 번호, 할당할 리소스를 설정한다.

## 네임스페이스에 Deployment 적용하기

```
kubectl apply -n example-app -f .\deployment.yaml
```

## `example-app`의 Deployment 확인

```
kubectl -n example-app get deploy
```

replicas를 2개로 설정해서 READY, UP-TO-DATE, AVAILABLE이 2로 뜬다. 

## `example-app`의 Pod 확인

```
kubectl -n example-app get pods
```

replicas를 2개로 설정해서 팟도 두 개가 뜬다. 

## 팟의 로그 확인하기

```
kubectl -n example-app logs example-deploy-6d56dff57c-8bfh5
```

## `service.yaml` 생성

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
  labels:
    app: example-app
spec:
  type: LoadBalancer
  selector:
    app: example-app
  ports:
    - protocol: TCP
      name: http
      port: 80
      targetPort: 5000
```

`metadate/labeles/app`으로 서비스와 Deployment를 연결한다.

`example-service`서비스의 80번 포트로 접속하면 컨테이너의 5000번 포트에 접속할 수 있다.

## 서비스 실행

```
kubectl apply -n example-app -f .\service.yaml
```

`service.yaml`의 `spec/type`이 `LoadBalancer`이므로 EKS, GCP, AZURE에서 서비스 실행 시 로드밸런서가 자동으로 설정된다.

`spec/type`이 `CluterIP`인 경우 내부 아이피만 생성돼서 외부에서 접속할 수 없다.

이 경우 아래 명령어를 실행해야만 외부에서 접속이 가능하다. (5000번 포트로 접속 가능) 

```
kubectl -n example-app port-forward example-deploy-6d56dff57c-8bfh5 5000
```

## 출처

[Youtube - Node.js to Kubenetes](https://youtu.be/J_kU7O8OCOA)