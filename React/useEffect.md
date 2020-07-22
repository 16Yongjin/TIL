# `useEffect`

훅을 이용하면 클래스 컴포넌트를 작성하지 않아도 상태 같은 다양한 리액트의 기능을 이용할 수 있다.

그 중 하나가 *이펙트*다.

이펙트 훅을 사용하면 함수 컴포넌트에서도 부수 효과를 일으킬 수 있다.

```jsx
import React, { useState, useEffect } from "react";
function Example() {
  const [count, setCount] = useState(0);

  // componentDidMount나 componentDidUpdate와 비슷하다.
  useEffect(() => {
    // 브라우저 API로 DOM을 변경한다.
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

위의 코드는 카운트를 도큐먼트 타이틀에 보여준다.

부수 효과의 예로는 데어터를 불러오거나 구독 설정하기, DOM을 직접 조작하는 것들이 있다.

> `useEffect`는 `componentDidMount`와 `componentDidUpdate`, `componentWillUnmount`를 합친거라고 봐도 된다.

리액트에서 사용하는 사이드 이펙트는 정리가 필요한 것과 필요하지 않은 것으로 나뉜다.

## 정리가 필요 없는 이펙트

실행한 뒤에 신경쓸 필요 없는 것들이다.

- 리액트의 DOM 업데이트 후 코드를 실행
- 네트워크 요청
- DOM 직접 조작
- 로깅

### 클래스 컴포넌트 사용

클래스 컴포넌트를 사용 시 렌더링이 끝날 때마다 코드를 실행시키려면 `componentDidMount`와 `componentDidUpdate`에 동일한 코드를 집어넣어야 헸다.

### 훅 사용

`useEffect`는 렌더링 후에 무조건 실행된다.

> `useEffect`는 브라우저를 막지 않도록 비동기로 스케줄링된다. 이펙트가 동기적으로 실행되길 원하면 `useLayoutEffect`를 사용한다.

## 정리가 필요한 이펙트

웹소켓 같이 구독이 필요한 자원은 사용 후 반드시 정리해서 메모리 누수를 막아야 한다.

### 클래스 컴포넌트 사용

클래스 컴포넌트 사용 시 구독은 `componentDidMount`와 `componentDidUpdate`에서, 구독 해제는 `componentWillUnmount`에서 따로 해야 한다.

이 경우 개념적으로 관련 있는 코드를 강제로 나눠서 훅에 등록해야 한다.

### 훅 사용

`useEffect`를 사용하면 관련 있는 코드를 한 곳에 다 집어넣을 수 있다.

```jsx
import React, { useState, useEffect } from "react";

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    // 이펙트를 정리하는 법을 명시함
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}
```

랜더링이 다시 될 때마다 이펙트가 실행된다.

리액트는 다음 이펙트 실행 전에 그전 이펙트를 정리한다.

## 이펙트 스킵으로 성능 최적화

매 랜더링 시 이펙트를 정리하고 실행하면 성능 저하가 일어날 수 있다.

클래시 컴포넌트 사용 시 `componentDidUpdate`에서 `prevProps`나 `prevState`를 비교해서 성능 향상을 했다.

이런 경우가 많으므로 랜더링 사이에 값의 변화가 없으면 `useEffect`가 이펙트 실행을 스킵하도록 할 수 있다.

`useEffect`의 두 번째 인자에 값을 배열로 전달한다.

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // count가 변했을 때만 이펙트를 재실행함
```

> 이펙트를 한번만 실행하고 정리하고 싶으면 훅에 두 번째 인자로 빈 배열(`[]`)을 넘긴다.
