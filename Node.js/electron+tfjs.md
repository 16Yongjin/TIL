# 일렉트론에서 `@tensorflow/tfjs-node` 사용하기

일렉트론에서 사용하는 Node.js 버전과 운영체제에 설치된 Node.js 버전이 같아야함

그래야 tfjs-node 내부 C 바이너리 빌드 시 오류가 발생하지 않는다.

## 버전 확인방법

```javascript
console.log(process.version);
```
