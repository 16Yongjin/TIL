# Get better at JavaScript with just JavaScript

### 출처: https://youtu.be/pws4qzGn5ak

# Intersection Observer

한 요소가 화면 안으로 들어왔는지 확인할 때 사용한다.

- 스크롤 시 요소에 애니메이션 넣기
- 스크롤 시 비디오 재생
- 이미지 지연로딩
- 광고 노출수 세기
- 스티키 헤더

등에 사용할 수 있다.

### 1. 옵션 설정

```javascript
const options = {
  root: document.querySelector(".scrollingDiv"),
  rootMargin: "100px",
  // 화면에 없을 때, 반만, 완전히 들어왔을 때 알려줌
  threshold: [0, 0.5, 1.0]
};
```

### 2. 빈 Observer 생성

```javascript
const observer = new IntersectionObserver(callback, options);
```

### 3. 콜백 전달하기

```javascript
const callback = (entries, observer) => {
  entries.forEach(entry => {
    console.log(entry);
    // threshold가 1일 때(요소가 완전히 들어왔을 때)만 보이게 하기
    if (entry.isIntersecting && entry.intersectionRatio >= 1) {
      entry.target.classList.add("visible");
    } else {
      entry.target.classList.remove("visible");
    }
    observer.unobserve(entry.target);
  });
};
```

### 4. 관찰 시작

```javascript
const boxes = document.querySelectorAll(".box");
boxes.forEach(box => observer.observe(box));
```

---

# Resize Observer

요소의 크기 변경을 확인할 수 있다.

### 기본 예제

```javascript
const callback = (entries, observer) => {
  entries[0].target.innerHTML = `
    <pre>
      ${JSON.stringify(entries[0].contentRect, null, " ")}
    </pre>
  `;
};

const observer = new ResizeObserver(callback);

const element = document.querySelector(".resize");
observer.observe(element);
```

요소 크기를 Viewport에 맞추는게 아니라 다른 요소가 얼마나 큰지, 어디에 있는 지 등, 내가 원하는 대로 지정하고 싶을 때 사용할 수 있다.

```javascript
const callback = (entries, observer) => {
  const { width } = entries[0].contentRect;
  if (width > 400) {
    size = "large";
  } else if (width > 300) {
    size = "medium";
  } else {
    size = "small";
  }

  entries[0].target.classList.remove("small", "medium", "large");
  entires[0].target.classList.add(size);
};

const observer = new ResizeObserver(callback);
const element = document.querySelector(".videos__list");
observer.observe(element);
```

---

# DOM Element Methods

## 1. .closest()

입력받은 선택자에 일치하는 가장 가까운 조상요소를 찾는다.

```HTML
<div class="cards">
  <div class="card">
    <p>I'm a Card</p>
    <button>Delete</button>
  </div>
  <div class="card">
    <p>I'm a Card</p>
    <button>Delete</button>
  </div>
</div>
```

2개의 카드가 있는 div요소가 있고 각 카드는 삭제 버튼이 있다.

```javascript
document.querySelectorAll(".card button").forEach(button => {
  button.addEventListener("click", e => {
    e.currentTarget.closest(".card").remove();
  });
});
```

closest 메서드로 버튼의 조상인 .card div를 찾아서 삭제한다.

```javascript
const p = document.querySelector("p");

document.addEventListener("click", e => {
  const isOutsize = !e.target.closest(".modal-inner");
  p.textContent = `You Clicked ${isOutsize ? "Outsize" : "Insize"}!`;
});
```

요소 밖을 클릭했는지 안을 클릭했는지 확인할 때도 유용하다.

## 2. .matches()

한 요소가 선택자와 일치하는지 확인한다.

```javascript
button.addEventListener("click", e => {
  if (e.currentTarget.matches(".available[data-price]")) {
    // .. 처리
  }
});
```

버튼이 data-price 라는 애트리뷰트를 갖고 있는지 확인한다.

```javascript
// list는 JS로 아이템을 추가/삭제하는 빈 요소임
list.addEventListener("click", e => {
  if (e.target.matches("button")) {
    deleteItem(parseFloat(e.target.value));
  }
});
```

Event Delegation에도 사용한다.
모든 리스트 내의 아이템에 이벤트 리스너를 붙일 필요가 없다.

## 3. .contains()

```javascript
const modal = document.querySelector(".modal");
modal.contains(button); // true

modal.querySelector("button"); // <button></button>
// contains 메서드와 같음
!!modal.querySelector("button"); // true
```

---

# Bling.js

바닐라 JS앱을 만들 때 있으면 유용한 11줄짜리 라이브러리

```javascript
window.$ = document.querySelector.bind(document);
window.$$ = document.querySelectorAll.bind(document);

Node.prototype.on = window.on = function(name, fn) {
  this.addEventListener(name, fn);
};

NodeList.prototype.__proto__ = Array.prototype;

NodeList.prototype.on = function(name, fn) {
  this.forEach(function(elem, i) {
    elem.on(name, fn);
  });
};
```

입력하기 힘든 document.querySelector를 \$ 한 문자로 줄임

NodeList에서 Array의 메서드를 사용할 수 있게 함

---

# 데이터 다루기

## 1. Array.from()

```javascript
Array.from({ length: 3 });
// => [undefined, undefined, undefined]

Array.from({ length: 3 }, () => `💰`);
// => ["💰", "💰", "💰"]
```

iterable을 array로 변환한다.

```javascript
Array.from({ length: 3 }, (_, i) => `day-${i + 1}`);
// => [ "day-1", "day-2", "day-3" ]
```

하드코딩을 피하고 원하는 데이터가 담긴 리스트 생성할 때 유용하다.

```javascript
Array.from({ length: 12 }, (x, index) =>
  new Date(0, index + 1, 0).toLocaleDateString("en-US", { month: "short" })
);
// => [ "Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]

Array.from({ length: 12 }, (x, index) =>
  new Date(0, index + 1, 0).toLocaleDateString("ko-KR", { month: "short" })
);

// => [ "1월", "2월", "3월", "4월", "5월", "6월", "7월", "8월", "9월", "10월", "11월", "12월"]
```

달 이름 생성할 때 사용할 수 있다.

---

# 중복 요소 제거하기

```javascript
const values = [1, 2, 3, 1, 2]

Array.from(new Set(values)) // [1, 2, 3]

[...new Set(values)] // [1, 2, 3]
```

---

# ...spread and ...rest

```javascript
// 1:1 복사
const copyOfHuman = { ...human };

// 복사 후 덮어쓰기
const copyNewId = {
  ...human,
  id: "das-vader"
};

// id를 뺀 모든 것
const { id, ...withoutId } = human;
```

객체 다루기

```javascript
// 기존 배열를 변경하지 않고 배열 뒤집기
const reversed = [...names].reverse();
```

---

# 에러 핸들링

```javascript
process.on("unhandledRejection", error => {
  console.log("unhandledRejection", error);
});
```

근데 위와 같이 에러를 무시할 바엔 unhandledRejection 발생 시 프로세스를 종료하고 다시 시작하는게 낫다고 노드 베스트 프랙티스에 나와있다.

---

# Web 음성인식

SpeechRecognition API로 음성인식이 가능하다.

---

# Shape Detection

얼굴, 바코드, 텍스트를 인식할 수 있다.
