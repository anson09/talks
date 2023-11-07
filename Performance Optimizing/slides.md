---
theme: dracula
title: Performance Optimizing
titleTemplate: '%s'
favicon: /logo-256.png
transition: slide-left
hideInToc: true
---

# 7 Talks on Performance Optimization

<p class="absolute right-30px bottom-30px">
 🥸Author: Anson, speaker from futu
</p>

<style>
h1 {
  background-image: linear-gradient(45deg, #d3f0ff 10%, #9b6ba5 70%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
} 
</style>


---
layout: intro
hideInToc: true
---

# Index

<Toc columns="2" class="gap-20" maxDepth="1"/>


---
layout: section
---

# Scope Hoisting


---

Let's pack 3 files, See what happened.

`index.js`
```js
import "./lib";
function add() {
  console.log("index add");
}
add();
```

<v-click>

`lib.js`
```js
import { add } from './sublib';
add();
```
</v-click>

<v-click>

`sublib.js`
```js
export function add() {
  console.log('sublib add');
}
```

</v-click>


---

First pack with webpack@3.0.0 <twemoji-grinning-face-with-sweat />

`npx webpack index.js bundle.js`

```js {0-66|67-82|83-92|93-104} {maxHeight:'350px'}
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, {
/******/ 				configurable: false,
/******/ 				enumerable: true,
/******/ 				get: getter
/******/ 			});
/******/ 		}
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__lib__ = __webpack_require__(1);


function add() {
  console.log("index add");
}

add();


/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__sublib__ = __webpack_require__(2);

__WEBPACK_IMPORTED_MODULE_0__sublib__["a" /* add */]();


/***/ }),
/* 2 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony export (immutable) */ __webpack_exports__["a"] = add;
function add() {
  console.log('sublib add');
}


/***/ })
/******/ ]);

```

**💡 wrapping each module in a function, which is called when the module is imported**


---

Then pack with rollup@4.0.0 <twemoji-grinning-face/>



`npx rollup index.js`

```js
function add$1() {
  console.log('sublib add');
}

add$1();

function add() {
  console.log("index add");
}

add();
```

**💡 the top-level variables of each module are renamed to ensure they are unique**


---

Last pack with parcel@2.0.0 <twemoji-grinning-squinting-face />


`npx parcel build index.js --no-source-maps`

```js
console.log("sublib add"),console.log("index add");
```

**💡 collaborating with tree-shaking futher**


---
layout: two-cols
---

**Without Scope Hoisting 🧐**

- Separate isolated scope
- Side effects run at the expected time
- HMR
- Code that cannot be statically analyzed 
- Development Mode

::right::

**With Scope Hoisting 🧐**

- Single scope
- Download size
- Runtime performance without object lookups
- Production Mode


---

## Side Effects

<div class="grid grid-cols-2 gap-8">
<div>

`app.js`
```js
import {add} from 'math';
console.log(add(2, 3));
```

`node_modules/math/index.js:`
```js {3|all}
export {add} from './add.js';
export {multiply} from './multiply.js';
let loaded = Date.now();
export function elapsed() {
  return Date.now() - loaded;
}
```

`node_modules/math/package.json:`
```js{all|3}
{
  "name": "math"
  "sideEffects": false
}
```

</div>
<div>

**WHEN?**
- DOM manipulation
- Log Something
- Gloabl variable assignment
- And so on ...

**sideEffects types: false | string | array\<string\>**

</div>
</div>


---
layout: section
---

# Cascading Cache


---

## What's Good Cache Strategy

<div v-click class="mt-4">

- Add revision information to the filenames of your assets
- Set max-age caching headers

👉 `filename: '[name]-[contenthash].js'`

- Code splitting such as making node_modules to splited vendor chunk

</div>

<div v-click class="text-5xl mt-20">

But there is a problem ⚠️

</div>


---
layout: two-cols-header
clicks: 1
---

**when making a patch in `vendor.mjs`** <span v-click=1>**, 80% caches are invalid 😨**</span>

`dep2.mjs`/`dep3.mjs` 👇🏼
```js {monaco-diff}
import {...} from '/vendor-5e6f.mjs';
~~~
import {...} from '/vendor-d4a1.mjs';
```

`main.mjs` 👇🏼
```js {monaco-diff}
import {...} from '/dep2-3c4d.mjs';
import {...} from '/dep3-d4e5.mjs';
~~~
import {...} from '/dep2-2be5.mjs';
import {...} from '/dep3-3c6f.mjs';
```

::left::

![caching-module-dependency-graph-before](/caching-module-dependency-graph-before-b10a36a36e.svg)

::right::

<div v-click=1>

![caching-module-dependency-graph-after](/caching-module-dependency-graph-after-b6afbdd237.svg)

</div>


---

**Best Solution - Import Maps**

<div class="grid grid-cols-[1fr_30%] gap-2">

<div>

Code in bunlde references `/vendor.mjs` but loads `/vendor-5e6f.mjs`.
```js
import {...} from '/vendor.mjs';
```

<v-click>

When bundle updates, just change the import map 🤩
```js
<script type="importmap">
{
  "imports": {
    "/main.mjs": "/main-1a2b.mjs",
    "/dep1.mjs": "/dep1-b2c3.mjs",
    "/dep2.mjs": "/dep2-3c4d.mjs",
    "/dep3.mjs": "/dep3-d4e5.mjs",
    "/vendor.mjs": "/vendor-5e6f.mjs",
  }
}
</script>
```
</v-click>

<v-click>

Cache is efficient now 🚀

</v-click>

</div>

<div>
<img v-click alt="caniuse-import-maps" src='/caniuse-import-maps.png' />
</div>

</div>


---

<div class="grid grid-cols-[1fr_40%] gap-10">
<div>


**Compatible Solution 1- Service Worker**

Map update with service worker version
```js
const importMap = {
  '/main.mjs': '/main-1a2b.mjs',
  '/dep1.mjs': '/dep1-b2c3.mjs',
  '/dep2.mjs': '/dep2-3c4d.mjs',
  '/dep3.mjs': '/dep3-d4e5.mjs',
  '/vendor.mjs': '/vendor-5e6f.mjs',
};

addEventListener('fetch', (event) => {
  const oldPath = new URL(event.request.url, location).pathname;
  if (importMap.hasOwnProperty(oldPath)) {
    const newPath = importMap[oldPath];
    event.respondWith(fetch(new Request(newPath, event.request)));
  }
});
```
<v-click>

Before service worker has installed and activated, the un-revisioned files will be requested on the first load

</v-click>

</div>
<div v-click>

**Compatible Solution 2 - Custom Script Loader**

Uses a manifest in each entry bundle, that's what kinds of bundler do now

</div>
</div>


---
layout: section
---

# Font


---

<div class="grid grid-cols-[1fr_40%] gap-2">
<div>

**Font render period**

If the font face is not loaded

PERIOD 1️⃣. **BLOCK**

Render with an **invisible** fallback font face, waiting for updating

PERIOD 2️⃣. **SWAP**

Render with a fallback font face, waiting for updating

PERIOD 3️⃣. **FAILURE**

Render fallback, won't update

</div>
<div v-click>

**Perfomance Impact**

FCP/LCP - Delay text rendering

CLS - Layout shift


<!-- <video controls autoplay loop >
  <source src="/overlap.mp4" type="video/mp4">
</video> -->


![overlap](/overlap.png)


</div>
</div>


---

## font-display

| Value | Block period | Swap period |
|-------|--------------|-------------|
| Auto | Varies by browser |	Varies by browser |
| Block | 2-3 seconds | Infinite |
| Swap | 0ms | Infinite |
| Fallback | 100ms | 3 seconds |
| Optional | 100ms | None |

<div v-click>

**first look in web font 💄: block**

**better for FCP/LCP 🚀: swap**

**better for CLS 🫨: optional**

</div>


---

## More Optimizing Methods

- Inline font declarations
```html
<head>
  <style>
    @font-face {
        font-family: "Open Sans";
        src: url("/fonts/OpenSans-Regular-webfont.woff2") format("woff2");
    }
    body {
        font-family: "Open Sans";
    }
  </style>
</head>
```

- PreConnect/PreLoad
```html
<head>
  <link rel="preconnect" href="https://fonts.com" crossorigin>
</head>
```


---

- WOFF2, 30% smaller than WOFF
- subset fonts*
```css
@font-face {
    font-family: "Open Sans";
    src: url("/fonts/OpenSans-Regular-webfont.woff2") format("woff2");
    unicode-range: U+0025-00FF;
}
```

- size-adjust
```css
@font-face {
  font-family: "Adjusted Typeface";
  size-adjust: 150%;
  src: url(some/path/to/typeface.woff2) format('woff2');
}

```

<p class="absolute bottom-0 text-xs">* subset font can't used in preload</p>


---
layout: section
---

# Pre-xxx


---
layout: intro
---

*As a website owner, you are the one who knows which resources are most crucial and what the actual user journey on your site is. And to improve the overall performance, perceived speed, and user experience of your website, you could use that knowledge to help the browsers load your pages faster.* 

*That’s where the resource hints come in.*


---

## preconnect

[speed up 100–500 ms according to Google](https://web.dev/articles/preconnect-and-dns-prefetch#:~:text=You%20can%20speed%20up%20the%20load%20time%20by%20100%E2%80%93500%20ms%20by%20establishing%20early%20connections%20to%20important%20third%2Dparty%20origins)

```html
<link rel="preconnect" href="https://example.com">
```

Performs DNS lookup, TCP handshake, and [TLS](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/) negotiation with the origin specified in the href attribute


<img alt="[with-preconnect" src='/with-preconnect.png' class="w-1/2"/>


<v-click>

Cons: If an established connection is not used quickly (within 10 seconds on Chrome), it would automatically be closed by the browser

</v-click>


---

## prefetch

**low priority**, executed as the browser sees fit, which is used for improving the load time of subsequent pages, such as you can apply the prefetch directive during the authentication of a use

[speed up 30% TTI in NETFLIX](https://medium.com/dev-channel/a-netflix-web-performance-case-study-c0bcde26a9d9#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6ImY1ZjRiZjQ2ZTUyYjMxZDliNjI0OWY3MzA5YWQwMzM4NDAwNjgwY2QiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTY1MzQ3MjI0MTMwOTI0NDkxOTEiLCJlbWFpbCI6ImZpbmFsc29uZzZAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5iZiI6MTY5OTE3NDM4NiwibmFtZSI6ImFuc29uIC53IiwicGljdHVyZSI6Imh0dHBzOi8vbGgzLmdvb2dsZXVzZXJjb250ZW50LmNvbS9hL0FDZzhvY0lDNGJiVEhLcVIzSERDbnBkRnVnMndfbi13V0VrSHFaemt1cHhpc0FWVzI5az1zOTYtYyIsImdpdmVuX25hbWUiOiJhbnNvbiIsImZhbWlseV9uYW1lIjoiLnciLCJsb2NhbGUiOiJlbiIsImlhdCI6MTY5OTE3NDY4NiwiZXhwIjoxNjk5MTc4Mjg2LCJqdGkiOiIzOWNhM2Y1NDBhOGY0YTZhZDE2ODRhNTYwZjlmMDhmN2RiN2E4NDNlIn0.iwGAZpy0_20xzs6s5zt4TfjXkcNBsdHF783CU6eVyMkfa-xmPa6wA8mYIUXLwISYdT4h3Web7BQV9bKh-iho_uwZKC-8S3yj5fb9tuDib037c__0D5kvpv4m7blR-wlSlHt0tlezNfBf-Lylml4L5Vut-xqFsilXEQHwUMawdPiqVY0DTNrDAZ3Iil0Q8dEvba56j-FCJqY-abnUsqVRFpta-kpFUKh6gesjYRcTXUcBbv-k2pHPJ8DVpAvvUa4O1XQATV6-aMBMp-DwNaji1wBAyo80i4U4e26Md4IkxFKztXjqWNXqJKFhgmWiHbh2i8zPhvdO7XtTF8iCpe6jpQ:~:text=Prefetching%20HTML%2C%20CSS%20and%20JavaScript%20(React)%20reduced%20Time%2Dto%2DInteractive%20by%2030%25%20for%20future%20navigations)

**take two request**, the second use preftech cache of the first


<img alt="prefetch-cache" src='/prefetch-cache.png' class="w-80%"/>

<v-click>

Cons: might increase the data consumption

</v-click>


---
layout: two-cols
---

## preload

Preload is a declarative fetch, and it’s **mandatory** for the browsers

works best on resources that are part of the [critical rendering path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path) in current page, such as fonts, css, or critical javascript

```html
<link rel="preload" as="image" href="header-logo.svg">
```

**only take one request**, warning if not used in 3 seconds

::right::

<img alt="value-of-as" src='/value-of-as.png' class="w-3/4 m-auto"/>


---
layout: two-cols
---

preload before :
![preload-before](/preload-before.png)

::right::
preload after 🚀:
![preload-after](/preload-after.png)


---
layout: section
---

# requestIdleCallback


---

## Methods used to schedule jobs

<v-click>

- SetTimeout / SetInterval / postTask
- rAF
- queueMicrotask

</v-click>

<v-click>

But they are not really idle schedule which suitable for **non-critical** tasks, such as: 

- Analytics
- Logging
- Prefetching
- And so on ...

More important works like **rendering** and **user interaction** will be blocked

Scheduling non-essential work yourself is very difficult to do. It’s impossible to figure out exactly how much frame time remains because after requestAnimationFrame callbacks execute there are style calculations, layout, paint, and other browser internals that need to run

</v-click>

---

<div class="grid grid-cols-2 gap-2">

<div>

**Free time at the end of a frame:**

rest time of (**16.67ms** in 60fps) each frame
<img  alt="ric-busy" src='/ric-busy.webp' class="m-auto"/>
</div>
<div>

**User is inactive:**

at most **50ms** when no after render work
<img  alt="ric-idle" src='/ric-idle.png' class="m-auto"/>

**100ms** is the maximum time human can feel the delay

</div>
</div>


---

## Code Example

```js
let handler = null;

// When browser is busy, rIC won't be called, using timeout to force trigger
handler = requestIdleCallback(myNonEssentialWork, { timeout: 2000 });

function myNonEssentialWork (deadline) {
  // deadline.timeRemaining() is the amount of time left in the current idle period
  // If the callback function is executed due to a timeout，deadline.didTimeout is true
  while ((deadline.timeRemaining() > 0 || deadline.didTimeout) &&
         tasks.length > 0) {
       doWorkIfNeeded();
    }
  if (tasks.length > 0) {
    handler = requestIdleCallback(myNonEssentialWork);
  }
}

```

cancel the callback
```js
cancelIdleCallback(handle);

```


---

## Best Practice

**Recommended:**

- Lower priority tasks
- Use timeouts only when needed

**Not Recommended:**

- DOM Manipulation (could build Document Fragment, then push to raF)
- Add microtask
- Tasks which could take an unpredictable amount of time
- Overrun the deadline


---
layout: section
---

# Image


---

## Understand Format

- jpeg/jpg: lossy image format, good for photos, but **does not support transparency or lossless compression**
- png: lossless image format, much **larger** than jpegs or other lossy image formats, but **support transparency** and offer much **higher quality** for fine details
- gif: lossless compression and can be used for animations, up to 8 bits per pixel and a **maximum of 256 colors** from the 24-bit color space

<v-click>

**More Recommended:**

- **webp**: supports both **lossy and lossless** compression as well as **animation and transparency**，offers better compression for the same quality as jpegs and pngs
- heic/heif: supports both **lossy and lossless** compression, **better compression** than webp, jpeg, png and gif, but **only apple system** because it is complex and expensive to license
- avif:  **lossy** image format based on the AV1 video format.,  offers **significant compression** and quality improvements over jpeg and webp

</v-click>


---

## Compatibility

<br>

<div class="grid grid-cols-4 gap-2">
  <img alt="animated-webp-supported" src='/animated-webp-supported.webp'/>
  <img alt="caniuse-webp" src='/caniuse-webp.png'/>
  <img alt="caniuse-heic" src='/caniuse-heic.png'/>
  <img alt="caniuse-avif" src='/caniuse-avif.png' class="relative bottom-2px"/>
</div>

<v-click>

```html
<picture>
  <source type="image/heic" srcset="/image.heic">
  <source type="image/avif" srcset="/image.avif">
  <source type="image/webp" srcset="/image.webp">
  <source type="image/jpeg" srcset="/image.jpeg">
  <img src="/image.jpeg" alt="Description">
</picture>
```

</v-click>


---
layout: section
---

# Compositing Layer


---



---
layout: quote
---

Reference Links:

> https://blog.developer.adobe.com/optimizing-javascript-through-scope-hoisting-2259ef7f5994

> https://philipwalton.com/articles/cascading-cache-invalidation/

> https://web.dev/articles/font-best-practices

> https://nitropack.io/blog/post/resource-hints-performance-optimization

> https://developer.chrome.com/blog/using-requestidlecallback/

---
layout: end
---

Thank you! 🙏🏼
