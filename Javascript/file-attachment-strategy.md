# 다양한 파일 첨부 방식

일반적인 파일 첨부 방식과 편리한 UX를 위한 드래그 & 드롭, 붙여넣기 파일 첨부 방식 구현하기

## 1. 인풋 태그 사용

파일 선택 시 인풋태그에서 파일을 가져온다.

```vue
<template>
  <input multiple type="file" accept="image/*" @change="onChangeFile" />
</template>

<script>
function selectFiles(e) {
  const files = e.target.files

  if (!files.length) return

  Array.from(files).forEach((file) => addFile(file))

  e.target.value = null
}
</script>
```

## 2. 드래그 & 드롭

`drop` 시 전송된 파일을 추가해주고 `drag` 시 `dragging` 변수로 스타일링을 한다.

```vue
<template>
  <div
    :class="{ dragging }"
    @drop.stop.prevent="onDropFile"
    @dragover.stop.prevent="dragging = true"
    @dragleave.stop.prevent="dragging = false"
  ></div>
</template>

<script>
function onDropFile(e) {
  const items = e.dataTransfer.items

  if (!items?.length) return

  Array.from(items)
    .filter((item) => item.kind === "file")
    .map((item) => item.getAsFile())
    .forEach((file) => addFile(file))

  this.dragging = false
}
</script>

<style>
.dragging {
  outline: 1px dashed black;
}
</style>
```

## 붙여넣기

`paste`시 클립보드에서 파일을 가져온다.

```vue
<template>
  <textarea @paste="onPasteFile"></textarea>
</template>

<script>
function onPasteFile(e) {
  const items = (e.clipboardData || e.originalEvent.clipboardData).items

  if (!items?.length) return

  Array.from(items)
    .filter((item) => item.kind === "file")
    .map((item) => item.getAsFile())
    .forEach((file) => addFile(file))
}
</script>
```
