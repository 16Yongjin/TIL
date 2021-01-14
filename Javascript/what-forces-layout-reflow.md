# reflow를 일으키는 것들

리플로우를 작동시키는 자바스크립트 프로퍼티와 메서드를 소개합니다. 리플로우는 브라우저가 동기적으로 스타일과 레이아웃을 계산하게 합니다. 레이아웃 쓰레싱이라고도 하며, 흔한 성능 병목입니다.

일반적으로, 레이아웃을 동기적으로 측정하는 API에서 리플로우(또는 레이아웃)이 일어납니다. 자세한 설명과 예시는 아래에 나와있습니다.

## Element API들

##### 박스 크기 재기

- `elem.offsetLeft`, `elem.offsetTop`, `elem.offsetWidth`, `elem.offsetHeight`, `elem.offsetParent`
- `elem.clientLeft`, `elem.clientTop`, `elem.clientWidth`, `elem.clientHeight`
- `elem.getClientRects()`, `elem.getBoundingClientRect()`

##### Scroll 관련

- `elem.scrollBy()`, `elem.scrollTo()`
- `elem.scrollIntoView()`, `elem.scrollIntoViewIfNeeded()`
- `elem.scrollWidth`, `elem.scrollHeight`
- `elem.scrollLeft`, `elem.scrollTop`과 이를 설정할 때

##### focus 설정

- `elem.focus()` ([출처](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/dom/element.cc;l=4206-4225;drc=d685ea3c9ffcb18c781bc3a0bdbb92eb88842b1b))

##### 이외에..

- `elem.computedRole`, `elem.computedName`
- `elem.innerText` ([출처](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/editing/element_inner_text.cc;l=462-468;drc=d685ea3c9ffcb18c781bc3a0bdbb92eb88842b1b))

### window 차원 가져오기

- `window.scrollX`, `window.scrollY`
- `window.innerHeight`, `window.innerWidth`
- window.visualViewport.height / width / offsetTop / offsetLeft ([출처](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/frame/visual_viewport.cc;l=435-461;drc=a3c165458e524bdc55db15d2a5714bb9a0c69c70?originalUrl=https:%2F%2Fcs.chromium.org%2F))

### document

- `document.scrollingElement` 스타일링만 발생시킵니다.
- `document.elementFromPoint`

### Forms: selection과 focus 설정하기

- `inputElem.focus()`
- `inputElem.select()`, `textareaElem.select()`

### Mouse events: offset 데이터 읽기

- `mouseEvt.layerX`, `mouseEvt.layerY`, `mouseEvt.offsetX`, `mouseEvt.offsetY` ([출처](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/events/mouse_event.cc;l=476-487;drc=52fd700fb07a43b740d24595d42d8a6a57a43f81))

### getComputedStyle() 호출

`window.getComputedStyle()`은 스타일 재계산을 일으킵니다.

`window.getComputedStyle()`은 때로 리플로우를 일으킵니다..

<details>
<summary>`gCS()`가 리플로우를 일으키는 조건</summary>

`window.getComputedStyle()`는 다음 3개 중 하나의 조건이 만족하면 리플로우가 일어납니다:

1. 요소가 그림자 트리에 있을 때
2. 뷰 포트 관련 미디어 쿼리가 있을 때. 특히, 다음 중 하나일 때: ([출처](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/css/media_query_exp.cc;l=240-256;drc=4c8db70889f2d2fae8338b16f553c646dd20bf78)
   - `min-width`, `min-height`, `max-width`, `max-height`, `width`, `height`
   - `aspect-ratio`, `min-aspect-ratio`, `max-aspect-ratio`
   - `device-pixel-ratio`, `resolution`, `orientation` , `min-device-pixel-ratio`, `max-device-pixel-ratio`
3. 요청한 프로퍼티가 다음 중 하나일 때: ([source](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/css/properties/css_property.h;l=69;drc=d685ea3c9ffcb18c781bc3a0bdbb92eb88842b1b))
   - `height`, `width`
   - `top`, `right`, `bottom`, `left`
   - 마진이 고정인 경우 `margin` [`-top`, `-right`, `-bottom`, `-left`, 또는 *축약표현*]
   - 패딩이 고정인 경우 `padding` [`-top`, `-right`, `-bottom`, `-left`, 또는 *축약표현*]
   - `transform`, `transform-origin`, `perspective-origin`
   - `translate`, `rotate`, `scale`
   - `grid`, `grid-template`, `grid-template-columns`, `grid-template-rows`
   - `perspective-origin`
   - 옛날에 해당됐지만 지금은 아닌 것들: (18년 2월 이후): `motion-path`, `motion-offset`, `motion-rotation`, `x`, `y`, `rx`, `ry`

</details>

### `Range` 차원 가져오기

- `range.getClientRects()`, `range.getBoundingClientRect()`

### SVG

많은 프로퍼티와 메서드가 레이아웃을 일으키지만 다 안 나와 있습니다:

- SVGLocatable: `computeCTM()`, `getBBox()`
- SVGTextContent: `getCharNumAtPosition()`, `getComputedTextLength()`, `getEndPositionOfChar()`, `getExtentOfChar()`, `getNumberOfChars()`, `getRotationOfChar()`, `getStartPositionOfChar()`, `getSubStringLength()`, `selectSubString()`
- SVGUse: `instanceRoot`

아래의 크로미엄 소스 트리 링크에서 찾아보세요.

### contenteditable

- 이미지를 클립보드에 복사하는 것을 포함해서 많은 것들이 리플로우를 일으킵니다. ([출처](https://source.chromium.org/search?q=UpdateStyleAndLayout%20-f:test&ss=chromium%2Fchromium%2Fsrc:third_party%2Fblink%2Frenderer%2Fcore%2Fediting%2F))

## \* 색인

- 리플로우는 도큐먼트가 변경되고 스타일이나 레이아웃에서 뭔가가 잘못됐을 때만 비용이 발생합니다. DOM 변경(클래스 변경, 노드 추가/삭제, `:focus` 같은 의사 클래스 추가 등)으로 인한 비용 발생이 일반적입니다.
- 리플로우를 방지하는 방법을 아래에 간단하게 소개합니다. 자세한 내용은 `리플로우 더 알아보기`를 참조하세요.
  1. `for` 내에서 리플로우와 DOM을 변경하지 마세요.
  2. 개발자 도구의 성능 패널에서 리플로우가 발생하는 곳을 찾아보세요. 생각보다 본인의 코드와 라이브러리에서 리플로우가 자주 발생해서 놀랄 수도 있습니다.
  3. DOM 읽고 쓰기를 한 번에 하세요.([FastDOM](https://github.com/wilsonpage/fastdom)이나 가상 DOM을 사용). 프레임의 첫 부분에서 레이아웃을 측정하세요.(`requesetAnimationFrame`이나 스크롤 처리 함수의 시작 부분 등). 이 첫 부분이 최근 리폴로우가 끝났을 때와 숫자들이 같습니다.

<center>
<img src="https://cloud.githubusercontent.com/assets/39191/10144107/9fae0b48-65d0-11e5-8e87-c9a8e999b064.png">
 <i>Guardian으로 타임라인 추적한 사진. Outbrain이 계속 리플로우를 일으키는데 아마 루프 안이라서 그런 것 같습니다.</i>
</center>

#### 리플로우 더 알아보기

- **[Avoiding layout thrashing — Web Fundamentals](https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing)** The **best** resource on identifying and fixing this topic.
- [CSS Triggers](http://csstriggers.com/) - covers what operations are required as a result of setting/changing a given CSS value. The above list, however, are all about what forces the purple/green/darkgreen circles synchronously from JavaScript.
- [Fixing Layout thrashing in the real world | Matt Andrews](https://mattandre.ws/2014/05/really-fixing-layout-thrashing/)
- [Timeline demo: Diagnosing forced synchronous layouts - Google Chrome](https://developer.chrome.com/devtools/docs/demos/too-much-layout)
- [Preventing &apos;layout thrashing&apos; | Wilson Page](http://wilsonpage.co.uk/preventing-layout-thrashing/)
- [wilsonpage/fastdom](https://github.com/wilsonpage/fastdom)
- [Rendering: repaint, reflow/relayout, restyle / Stoyan](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/)
- [We spent a week making Trello boards load extremely fast. Here’s how we did it. - Fog Creek Blog](http://blog.fogcreek.com/we-spent-a-week-making-trello-boards-load-extremely-fast-heres-how-we-did-it/)
- [Minimizing browser reflow | PageSpeed Insights | Google Developers](https://developers.google.com/speed/articles/reflow?hl=en)
- [Optimizing Web Content in UIWebViews and Websites on iOS](https://developer.apple.com/videos/wwdc/2012/?id=601)
- [Accelerated Rendering in Chrome](http://www.html5rocks.com/en/tutorials/speed/layers/)
- [web performance for the curious](https://www.igvita.com/slides/2012/web-performance-for-the-curious/)
- [Jank Free](http://jankfree.org/)

---

2020년 4월에 약간 업데이트 됐습니다.

## [원문 출처](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)
