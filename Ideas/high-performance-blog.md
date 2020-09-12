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

- Generates multiple sizes of each image and uses them in **`srcset`**.
- Generates a **blurry placeholder** for each image (without adding an HTML element or using JS).
- Transcodes images to [AVIF](<https://en.wikipedia.org/wiki/AV1#AV1_Image_File_Format_(AVIF)>) (currently off-by-default due to instabillity of the encoder) and [webp](https://developers.google.com/speed/webp) and generates `picture` element.
- **Lazy loads** images (using [native `loading=lazy`](https://web.dev/native-lazy-loading/)).
- **Async decodes** images (using `decoding=async`).
- **Avoids CLS impact** of images by inferring and providing width and height (Supported in Chrome, Firefox and Safari 14+).
- Downloads remote images and stores/serves them locally.
- Immutable URLs.

#### CSS

- Defaults to the compact "classless" [Bahunya CSS framework](https://kimeiga.github.io/bahunya/).
- Inlines CSS.
- Dead-code-eliminates / tree-shakes / purges (pick your favorite word) unused CSS on a per-page basis with [PurgeCSS](https://purgecss.com/).
- Minified CSS with [csso](https://www.npmjs.com/package/csso).

#### Miscellaneous

- Immutable URLs for JS.
- Sets immutable caching headers for images, fonts, and JS (CSS is inlined). Currently implements for Netlify `_headers` file.
- Minifies HTML and optimizes it for compression. Uses [html-minifier](https://www.npmjs.com/package/html-minifier) with aggressive options.
- Uses [rollup](https://rollupjs.org/) to bundle JS and minifies it with [terser](https://terser.org/).
- Prefetches same-origin navigations when a navigation is likely.
- If an AMP files is present, [optimizes it](https://amp.dev/documentation/guides-and-tutorials/optimize-and-measure/optimize_amp/).

#### Fonts

- Serves fonts from same origin.
- Makes fonts `display:swap`.

#### Analytics

- Supports locally serving Google Analytics's JS and proxying it's hit requests to a Netlify proxy (other proxies could be easily added).
- Support for noscript hit requests.
- Avoids blocking onload on analytics requests.
- To turn this on, specify `googleAnalyticsId` in `metadata.json`.

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
