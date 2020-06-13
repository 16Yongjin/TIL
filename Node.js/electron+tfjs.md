# 일렉트론에서 `@tensorflow/tfjs-node` 사용하기

일렉트론에서 사용하는 Node.js 버전과 운영체제에 설치된 Node.js 버전이 같아야함

그래야 tfjs-node 내부 C 바이너리 빌드 시 오류가 발생하지 않는다.

## 버전 확인방법

```javascript
console.log(process.version);
```

## COCO SSD 모델 오프라인으로 사용하기

웹에서 모델 로딩 시 인터넷 속도가 느리면 1분 정도 걸리고 메인 스레드가 막힌다.

Web Worker를 사용하면 되지만 그래도 브라우저에 의한 CPU 성능 제한이 있으므로

오프라인 사용이 가능한 일렉트론의 기능을 최대한 활용해야 한다.

### `node-fetch` 모듈 추가

tfjs는 구글 API 스토리지에서 모델을 가져오기 위해 fetch API를 사용한다.

Node.js에는 fetch API가 없으므로 `node-fetch` 모듈을 사용한다.

```
yarn add node-fetch
```

### 모델 파일 다운로드 받기

`https://storage.googleapis.com/tfjs-models/savedmodel/ssdlite_mobilenet_v2/model.json`에서 `model.json`를 다운받는다.

`model.json` 내부의 `.weightsManifest[].paths[0]`에 나와있는 가중치가 저장된 `group1-shard{1..5}of5` 파일을 다운받는다.

잘 모르겠으면 브라우저에서 모델을 로딩하고 개발자툴의 네트워크 탭을 살펴본다.

파일들을 프로젝트 루트의 `public/` 폴더에 넣는다.

`public/` 루트는 `__static` 전역변수로 접근가능하다.

다만 타입스크립트는 `__static`을 모르므로 `declare const __static: string` 이 코드를 추가한다.

### fetch API에 file:// 프로토콜 지원 추가하기

브라우저와 node.js 모두에서 사용할 수 있는 전역공간인 `globalThis`에 `fetch`를 정의한다.

`url`의 프로토콜이 `http`라면 `node-fetch`의 `fetch`를 사용하고

`file`이라면 해당 `url`에서 읽은 파일을 `fetch` API처럼 `Response` 객체에 담아서 반환한다.

이렇게 정의한 `fetch`는 `tfjs` 내부에서 사용될 것이다.

### 전체 코드

```typescript
import * as cocoSsd from "@tensorflow-models/coco-ssd";
import fetch, { Response, RequestInit } from "node-fetch";
import fs from "fs";
import { promisify } from "util";
import path from "path";
declare const __static: string;

const readFile = promisify(fs.readFile);

// 앞에 붙는 :// 을 지운다.
const removeProtocol = (url: string) => url.replace(/(^\w+:|^)\/\//, "");

// 파일을 읽고 fetch의 Response 형식으로 반환함
const fetchFile = async (url: string) => {
  const path = removeProtocol(url)
  const file = await readFile(path)
  const contentType =
    url.indexOf('.json') !== -1
      ? 'application/json'
      : 'application/octet-stream'
  const headers = { 'content-type': contentType }
  return new Response(file, { headers })
}

// @ts-ignore
// 이제 fetch 시 file 프로토콜도 지원함
globalThis.fetch = async (url: string, init?: RequestInit | undefined) =>
  url.startsWith('file://') ? fetchFile(url) : fetch(url, init)

// /public/cocossd/model.json이 모델 파일 위치임
const modelUrl = `file://${__static}/cocossd/model.json`;

cocoSsd.load({ modelUrl });
```

브라우저에선 1분 걸렸던 모델 로딩이 일렉트론에서는 3초 이내로 줄었다.
