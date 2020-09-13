# 엄청 빠른 블로그

깃헙에 올라와 있는 [eleventy-high-performance-blog](https://github.com/google/eleventy-high-performance-blog) 레포는 [Eleventy](https://www.11ty.dev/) 정적 사이트 생성기에 성능 모범 사례를 추가해서 엄청 빠른 블로그 생성기를 구현했다.

![image](https://user-images.githubusercontent.com/22253556/93019254-c6984900-f610-11ea-9e52-fe1a83a140ac.png)

라이트하우스를 돌리면 성능, 접근성, 모범 사례, SEO 모두 만점이 나온다.

`READMD.md`에 적힌 특징을 살펴보면서 어떻게 이런 결과를 얻을 수 있는지 확인해본다.

## 특징

### 성능 결과

- 라이트하우스 검사가 모두 만점이 나온다.
- 첫 HTTP에서 [First Contenful Paint](https://web.dev/first-contentful-paint/)가 발생한다.
- [Largest Contentful Paint](https://web.dev/lcp/)가 매우 최적화되어 있다. (이 점수는 이미지에 달려 있는데 이미지가 최적화되어 있다는 뜻)
- [Cumulative Layout Shift](https://web.dev/cls/)이 0
- [First Input Delay](https://web.dev/fid/)가 0임

### 성능 최적화 방법

#### 이미지

- 다양한 크기의 이미지를 생성하고 `srcset`에서 사용한다.
- 각 이미지마다 **blurry placeholder**를 생성한다.(HTML 요소를 추가하거나 JS를 사용하지 않음)
- 이미지를 [AVIF](<https://en.wikipedia.org/wiki/AV1#AV1_Image_File_Format_(AVIF)>) (인코더가 안정화되지 않아서 현재는 꺼져있음)와 [webp](https://developers.google.com/speed/webp)로 트랜스코딩하고 `picture` 요소를 생성한다.
- 이미지들을 **지연 로딩**한다. ([내장 `loading=lazy`](https://web.dev/native-lazy-loading/) 사용).
- 이미지를 **비동기 디코딩**한다. (`decoding=async` 사용).
- 이미지의 **CLS**를 막기 위해 이미지의 높이와 너비를 예측해서 제공함 (Chrome, Firefox, Safari 14+에서 지원함).
- 원격 이미지를 다운로드해서 로컬에 저장하고 서빙함.
- 불변 URL.

#### CSS

- 용량이 적은 "클래스가 없는" [Bahunya CSS 프레임워크](https://kimeiga.github.io/bahunya/) 기본 사용.
- CSS 인라인함.
- 안 쓰는 코드 제거 / 트리 쉐이킹 / [PurgeCSS](https://purgecss.com/)를 사용해서 페이지 마다 안 쓰는 CSS를 제거함.
- [csso](https://www.npmjs.com/package/csso)로 CSS 축소.

#### 기타

- JS URL은 불변하게.
- 이미지, 폰트, JS에 불변 캐싱 해더를 설정함(CSS는 인라인되어 있음). 현재는 Netlify `_headers` file만 구현되어 있음.
- HTML를 축소하고 압축을 위해 최적화함. [html-minifier](https://www.npmjs.com/package/html-minifier)에 aggressive 옵션을 넣어서 사용.
- [rollup](https://rollupjs.org/)로 JS를 번들링하고 [terser](https://terser.org/)로 축소함.
- 내비게이션이 일어날 거 같을 때 Orgin이 같은 내비게이션을 미리 불러옴.
- AMP 파일이 있으면 [최적화 함](https://amp.dev/documentation/guides-and-tutorials/optimize-and-measure/optimize_amp/).

#### Fonts

- 같은 오리진에서 폰트를 서빙함.
- fonts에 `display:swap` 옵션 적용.

#### Analytics

- Google Analytics의 JS 파일을 로컬에서 제공하고 Hit 요청을 Netlify 프록시로 보냄.
- noscript hit 요청 지원.
- 동기적 analytics 요청을 하지 않음.
- 이를 위해 `metadata.json`의 `googleAnalyticsId`를 명시함.

### DX features

- Uses 🚨 as favicon during local development.
- Supports a range of default tests.
- Runs build and tests on `git push`.
- Sourcemap generated for JS.

### SEO & Social

- Share button prefering `navigator.share()` and falling back to Twitter. Using OS-like share-icon.
- Support for OGP metadata.
- Support for Twitter metadata.
- Support for schema.org JSON-LD.
- Sitemap.xml generation.

### Largely useless glitter

- Read time estimate.
- Animated scroll progress bar…
- …with an optimized implementation that should never cause a layout.

### Security

Generates a strong CSP for the base template.

- Default-src is self.
- Disallows plugins.
- Generates hash based CSP for the JS used on the site.

### Build performance

- Downloaded remote images, and generated sizes are cached in the local filesystem…
- …and SHOULD be committed to git.
- `.persistimages.sh` helps with this.

## Disclaimer

This is not an officially supported Google product, but rather [Malte's](https://twitter.com/cramforce) private best-effort open-source project.
