# 콘솔에서 외부 라이브러리 테스트하기

## 라이브러리 불러오기

```js
const injectScript = (src) =>
  new Promise((resolve, reject) => {
    const script = document.createElement('script')
    script.src = src
    script.addEventListener('load', resolve)
    script.addEventListener('error', (e) => reject(e.error))
    document.head.appendChild(script)
  })
```

## blob 다운로드하기

```js
const saveBlob = (() => {
  const a = document.createElement('a')
  document.body.appendChild(a)
  a.style = 'display: none'
  return (blob, fileName) => {
    const url = window.URL.createObjectURL(blob)
    a.href = url
    a.download = fileName
    a.click()
    window.URL.revokeObjectURL(url)
  }
})()
```

## 예시) `html2canvas` 라이브러리 테스트

```js
await injectScript('https://html2canvas.hertzen.com/dist/html2canvas.min.js')
const canvas = await html2canvas(document.querySelector('.container'))
const file = await canvas.toBlob((blob) => saveData(blob, 'test.jpeg'))
```
