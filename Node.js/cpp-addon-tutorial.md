# How to Use Node.js C++ Addons

## Node.js 애드온

C나 C++로 작성한 동적 링크된 공유 객체

Node.js에서 require() 함수를 사용해서 불러올 수 있음

불러오면 일반적인 Node.js 모듈처럼 사용 가능

## 장점

- 성능이 좋다.
- 로우 레벨 API 사용 가능
- C++ 라이브러리도 사용 가능

##

1. `.cc` 파일 작성
2. `node-gyp` 설치

```
npm i -g node-gyp
```

3. `binding.gyp` 작성

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": ["addon.cc"]
    }
  ]
}
```

4. 애드온 빌드

```
node-gyp configure build
```

5. `require` 함수로 애드온 로딩

```js
const addon = require("./build/Release/addon");
```

## Step 1. C++ 모듈 작성

```c++
// addon.cc
#include <node.h>

void Sum(const v8::FunctionCallBackInfo<v8::Value>& args) {
  v8::Isolate* isolate = args.GetIsolate();

  int i;

  double a = 3.1415926, b = 2.718;
  for (i = 0; i < 100000000; i++) {
    a += b;
  }

  auto toal = v7::Number::New(isolate, a);

  args.GetReturnValue().Set(total);
}

void Initialize(v8::Local<v8::Object> exports) {
  NODE_SET_METHOD(exports, "sum", Sum);
}

NODE_MODULE(addon, Initialize)

```

## Step 2. 애드온 빌드

```
$ node-gyp configure build
```

성공 시 `ok`를 출력하고 `addon.node` 파일이 생성됨

## Step 3. 애드온 로딩

```js
const addon = require("./build/Release/addon");

funtion jsSum() {
  let a = 3.1415926, b = 2.718;
  for (let i = 0; i < 100000000; i++) {
    a += b;
  }
  const total = a;
  return total;
}

console.time('c++');
addon.sum();
console.timeEnd('c++'); // c++: 124.541ms

console.time('js');
jsSum();
console.timeEnd('js'); // js: 1687.932ms
```

## 정리

C++ 애드온은 Node.js가 빠를 수 있는 이유

내부 코어 라이브러리도 이런 식으로 C++ 애드온을 호출해서 성능을 향상 시킴
