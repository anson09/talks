---
theme: dracula
title: Performance Optimization
titleTemplate: "%s"
favicon: /logo-256.png
transition: slide-left
hideInToc: true
---

# 7 Talks on Performance Optimization

<div class="absolute right-30px bottom-30px">
<p class="author">🥸 Author: Anson, speaker from futu</p>
<sub>online slide: <a>https://talks.anson.ltd/performance-optimization/</a></sub>
</div>

<style>
h1 {
  background-image: linear-gradient(45deg, #d3f0ff 10%, #9b6ba5 70%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

p.author {
  margin: 0;
  text-align: right;
}

</style>

<!--
general methods: network、cache、resource、runtime、api
-->

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

Let's pack 3 files and see what happens.

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
import { add } from "./sublib";
add();
```

</v-click>

<v-click>

`sublib.js`

```js
export function add() {
  console.log("sublib add");
}
export function sub() {
  console.log("sublib sub");
}
```

</v-click>

---

First, pack with webpack@3.0.0 <twemoji-grinning-face-with-sweat />

`npx webpack index.js bundle.js`

```js {0-87|88-101|102-111|112-128} {maxHeight:'350px'}
/******/ (function (modules) {
  // webpackBootstrap
  /******/ // The module cache
  /******/ var installedModules = {};
  /******/
  /******/ // The require function
  /******/ function __webpack_require__(moduleId) {
    /******/
    /******/ // Check if module is in cache
    /******/ if (installedModules[moduleId]) {
      /******/ return installedModules[moduleId].exports;
      /******/
    }
    /******/ // Create a new module (and put it into the cache)
    /******/ var module = (installedModules[moduleId] = {
      /******/ i: moduleId,
      /******/ l: false,
      /******/ exports: {},
      /******/
    });
    /******/
    /******/ // Execute the module function
    /******/ modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__,
    );
    /******/
    /******/ // Flag the module as loaded
    /******/ module.l = true;
    /******/
    /******/ // Return the exports of the module
    /******/ return module.exports;
    /******/
  }
  /******/
  /******/
  /******/ // expose the modules object (__webpack_modules__)
  /******/ __webpack_require__.m = modules;
  /******/
  /******/ // expose the module cache
  /******/ __webpack_require__.c = installedModules;
  /******/
  /******/ // define getter function for harmony exports
  /******/ __webpack_require__.d = function (exports, name, getter) {
    /******/ if (!__webpack_require__.o(exports, name)) {
      /******/ Object.defineProperty(exports, name, {
        /******/ configurable: false,
        /******/ enumerable: true,
        /******/ get: getter,
        /******/
      });
      /******/
    }
    /******/
  };
  /******/
  /******/ // getDefaultExport function for compatibility with non-harmony modules
  /******/ __webpack_require__.n = function (module) {
    /******/ var getter =
      module && module.__esModule
        ? /******/ function getDefault() {
            return module["default"];
          }
        : /******/ function getModuleExports() {
            return module;
          };
    /******/ __webpack_require__.d(getter, "a", getter);
    /******/ return getter;
    /******/
  };
  /******/
  /******/ // Object.prototype.hasOwnProperty.call
  /******/ __webpack_require__.o = function (object, property) {
    return Object.prototype.hasOwnProperty.call(object, property);
  };
  /******/
  /******/ // __webpack_public_path__
  /******/ __webpack_require__.p = "";
  /******/
  /******/ // Load entry module and return exports
  /******/ return __webpack_require__((__webpack_require__.s = 0));
  /******/
})(
  /************************************************************************/
  /******/ [
    /* 0 */
    /***/ function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
      /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__lib__ =
        __webpack_require__(1);

      function add() {
        console.log("index add");
      }
      add();

      /***/
    },
    /* 1 */
    /***/ function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__sublib__ =
        __webpack_require__(2);

      __WEBPACK_IMPORTED_MODULE_0__sublib__["a" /* add */]();

      /***/
    },
    /* 2 */
    /***/ function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      /* harmony export (immutable) */ __webpack_exports__["a"] = add;
      /* unused harmony export sub */
      function add() {
        console.log("sublib add");
      }
      function sub() {
        console.log("sublib sub");
      }

      /***/
    },
    /******/
  ],
);
```

**💡 wrapping each module in a function, which is called when the module is imported**

---

Then, pack with rollup@4.0.0 <twemoji-grinning-face/>

`npx rollup index.js`

```js
function add$1() {
  console.log("sublib add");
}

add$1();

function add() {
  console.log("index add");
}

add();
```

**💡 scope hoisting + tree-shaking**  
**💡 the top-level variables of each module are renamed to ensure they are unique**

---

Lastly, pack with parcel@2.0.0 <twemoji-grinning-squinting-face />

`npx parcel build index.js --no-source-maps`

```js
(console.log("sublib add"), console.log("index add"));
```

**💡 minimizing the size of the bundle further**

---
layout: two-cols
---

**Without Scope Hoisting 🧐**

- Development Mode
- Separate isolated scope
- Side effects run at the expected time
- HMR
- Code which can't be statically analyzed

::right::

**With Scope Hoisting 🧐**

- Production Mode
- Single scope
- Download size
- Runtime performance without object lookups

---

## Side Effects

<div class="grid grid-cols-2 gap-8">
<div>

`app.js`

```js
import { add } from "math";
console.log(add(2, 3));
```

`node_modules/math/index.js:`

```js {3|all}
export { add } from "./add.js";
export { multiply } from "./multiply.js";
let loaded = Date.now();
export function elapsed() {
  return Date.now() - loaded;
}
```

`node_modules/math/package.json:`

```js{all|3}
{
  "name": "math",
  "sideEffects": false
}
```

</div>
<div>

**WHEN?**

- DOM manipulation
- Log something
- Global variable assignment
- And so on ...

**sideEffects types: false | string | array\<string\>**

</div>
</div>

---
layout: section
---

# Cascading Cache

---

## What's a Good Caching Strategy

<div v-click class="mt-4">

- Add revision information to the filenames of your assets
- Set max-age caching headers

👉 `filename: '[name]-[contenthash].js'`

- Code splitting, such as splitting `node_modules` into a vendor chunk

</div>

<div v-click class="text-5xl mt-20">

But there is a problem ⚠️

</div>

---
layout: two-cols
---

`dep2.mjs`/`dep3.mjs` 👇🏼

<div class="pr-6">

```js
- import {...} from '/vendor-5e6f.mjs';
+ import {...} from '/vendor-d4a1.mjs';
```

`main.mjs` 👇🏼

```js
- import {...} from '/dep2-3c4d.mjs';
- import {...} from '/dep3-d4e5.mjs';
+ import {...} from '/dep2-2be5.mjs';
+ import {...} from '/dep3-3c6f.mjs';
```

![caching-module-dependency-graph-before](/caching-module-dependency-graph-before-b10a36a36e.svg)

</div>

::right::

<div class="pl-6">

<v-click>

**when making a patch in `vendor.mjs`** <span text-2xl>**, 80% of the cache is invalidated 😨**</span>

![caching-module-dependency-graph-after](/caching-module-dependency-graph-after-b6afbdd237.svg)

</v-click>

</div>

---

**Modern Option - Import Maps**

<div class="grid grid-cols-[1fr_30%] gap-2">

<div>

In modern browsers with native ESM, code in bundle references `/vendor.mjs` but loads `/vendor-5e6f.mjs`.

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

**Compatible Solution - Service Worker**

Map update with service worker version

```js
const importMap = {
  "/main.mjs": "/main-1a2b.mjs",
  "/dep1.mjs": "/dep1-b2c3.mjs",
  "/dep2.mjs": "/dep2-3c4d.mjs",
  "/dep3.mjs": "/dep3-d4e5.mjs",
  "/vendor.mjs": "/vendor-5e6f.mjs",
};

addEventListener("fetch", (event) => {
  const oldPath = new URL(event.request.url, location).pathname;
  if (importMap.hasOwnProperty(oldPath)) {
    const newPath = importMap[oldPath];
    const newUrl = new URL(newPath, self.location.origin);
    event.respondWith(
      fetch(newUrl, { method: "GET", credentials: "same-origin" }),
    );
  }
});
```

<v-click>

Before the service worker is installed and activated, the un-revisioned files are requested on the first load

</v-click>

---
layout: section
---

# Font

---

<div class="grid grid-cols-[1fr_40%] gap-2">
<div>

**Font rendering timeline**

If the web font is not loaded

PERIOD 1️⃣. **BLOCK**

Render text as **invisible** (FOIT) while waiting for the web font to load

PERIOD 2️⃣. **SWAP**

Render with a fallback font first, then swap to the web font when available

PERIOD 3️⃣. **FAILURE**

Render with a fallback font and keep it (no swap)

</div>
<div v-click="1">

**Performance Impact**

FCP/LCP - Delay text rendering

CLS - Layout shift

<v-switch>
  <template #1>
    <video controls autoplay loop >
      <source src="/overlap.mp4" type="video/mp4">
    </video>
  </template>
  <template #2>
    <img alt="overlap" src="/overlap.png" />
  </template>
</v-switch>

</div>
</div>

---

## font-display

| Value    | Block period      | Swap period       |
| -------- | ----------------- | ----------------- |
| Auto     | Varies by browser | Varies by browser |
| Block    | 2-3 seconds       | Infinite          |
| Swap     | 0ms               | Infinite          |
| Fallback | 100ms             | 3 seconds         |
| Optional | 100ms             | None              |

<div v-click>

**default behavior for web fonts 💄: auto (often block-like)**

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

- preconnect/preload

```html
<head>
  <link rel="preconnect" href="https://fonts.com" crossorigin />
</head>
```

---

- WOFF2, 30% smaller than WOFF
- Subset fonts

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
  src: url(some/path/to/typeface.woff2) format("woff2");
}
```

---
layout: section
---

# Pre-xxx

---
layout: intro
---

_As a website owner, you are the one who knows which resources are most crucial and what the actual user journey on your site is. And to improve the overall performance, perceived speed, and user experience of your website, you could use that knowledge to help the browsers load your pages faster._

_That’s where the resource hints come in._

---

## preconnect

[speed up 100–500 ms according to Google](https://web.dev/articles/preconnect-and-dns-prefetch#:~:text=You%20can%20speed%20up%20the%20load%20time%20by%20100%E2%80%93500%20ms%20by%20establishing%20early%20connections%20to%20important%20third%2Dparty%20origins)

```html
<link rel="preconnect" href="https://example.com" />
```

Performs DNS lookup, TCP handshake, and [TLS](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/) negotiation with the origin specified in the href attribute

<img alt="with-preconnect" src='/with-preconnect.png' class="w-1/2"/>

<v-click>

Cons: If an established connection is not used soon, the browser may close it (timing is browser-dependent; Chrome is often around 10s)

</v-click>

---

## prefetch

**lowest priority**, executed as the browser sees fit, used to improve the load time of subsequent pages, for example during user authentication flows

[speed up 30% TTI in Netflix](https://medium.com/dev-channel/a-netflix-web-performance-case-study-c0bcde26a9d9)

Can appear as **two entries in DevTools** (one prefetch, one real fetch), and the second often reuses the prefetched cache entry

<img alt="prefetch-cache" src='/prefetch-cache.png' class="w-80%"/>

<v-click>

Cons: might increase the data consumption

</v-click>

---
layout: two-cols
---

## preload

Preload is a declarative fetch, and browsers generally treat it as a **high-priority required fetch**; priority still depends on the value of the `as` attribute (and browser scheduling)

Works particularly well on resources that are part of the [critical rendering path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path) on the current page, such as fonts, CSS, or critical JavaScript

```html
<link rel="preload" as="image" href="header-logo.svg" />
```

Ideally **one network transfer**, with a warning if not used shortly after load (browser-dependent)

::right::

<img alt="value-of-as" src='/value-of-as.png' class="w-3/4 m-auto"/>

---
layout: two-cols
---

**preload before :**
![preload-before](/preload-before.png)

::right::
**preload after 🚀:**
![preload-after](/preload-after.png)

---
layout: section
---

# requestIdleCallback

---

## Methods used to schedule jobs

<v-click>

- setTimeout / setInterval / postTask
- rAF
- queueMicrotask

</v-click>

<v-click>

But they are not true idle scheduling mechanisms suitable for **non-critical** tasks, such as:

- Analytics
- Logging
- Prefetching
- And so on ...

More important works like **rendering** and **user interaction** will be blocked

Scheduling non-essential work yourself is very difficult. It’s hard to know exactly how much frame time remains because, after `requestAnimationFrame` callbacks execute, there are style calculations, layout, paint, and other browser internals that still need to run

</v-click>

---

<div class="grid grid-cols-2 gap-2">

<div>

**Free time at the end of a frame:**

remaining time of each frame (**16.67ms** at 60fps)
<img  alt="ric-busy" src='/ric-busy.webp' class="m-auto"/>

</div>
<div>

**User is inactive:**

at most **50ms** when there is no post-render work
<img  alt="ric-idle" src='/ric-idle.png' class="m-auto"/>

**~100ms** is a commonly cited threshold where users start to notice delay

</div>
</div>

---

## Code Example

```js
let handler = null;

// When the browser is busy, rIC won't be called, so use timeout to force a trigger
handler = requestIdleCallback(myNonEssentialWork, { timeout: 2000 });

function myNonEssentialWork(deadline) {
  // deadline.timeRemaining() is the amount of time left in the current idle period
  // If the callback function is executed due to a timeout，deadline.didTimeout is true
  while (
    (deadline.timeRemaining() > 0 || deadline.didTimeout) &&
    tasks.length > 0
  ) {
    doWorkIfNeeded();
  }
  if (tasks.length > 0) {
    handler = requestIdleCallback(myNonEssentialWork);
  }
}
```

Cancel the callback

```js
cancelIdleCallback(handler);
```

---

## Recommended Practice

**Recommended:**

- Lower priority tasks
- Use timeouts only when needed

**Not Recommended:**

- DOM Manipulation (could build Document Fragment, then push to rAF)
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
- png: lossless image format, much **larger** than JPEGs or other lossy image formats, but **supports transparency** and offers much **higher quality** for fine details
- gif: lossless compression and can be used for animations, up to 8 bits per pixel and a **maximum of 256 colors** from the 24-bit color space

<v-click>

**More Recommended:**

- **webp**: supports both **lossy and lossless** compression as well as **animation and transparency**, and offers better compression for the same quality as JPEGs and PNGs
- heic/heif: supports both **lossy and lossless** compression, and can often achieve better compression than WebP/JPEG/PNG/GIF in many scenarios, but is mostly limited to the Apple ecosystem because it is complex and expensive to license
- avif: an image format based on AV1 that supports both **lossy and lossless** compression, often with better compression than JPEG/WebP at similar quality

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
  <source type="image/heic" srcset="/image.heic" />
  <source type="image/avif" srcset="/image.avif" />
  <source type="image/webp" srcset="/image.webp" />
  <img src="/image.jpeg" alt="Description" />
</picture>
```

</v-click>

---
layout: section
---

# Compositing Layer

---
layout: intro
---

_During painting/compositing, browsers may split elements into multiple compositing layers. Promoting content to a composited layer can improve performance for some cases (especially transform/opacity animations) by reducing repaint work and relying more on compositor operations, but it is not always faster and can increase memory/rasterization cost. Layer promotion is browser- and situation-dependent: elements such as \<video\>, \<canvas\>, and styles like 3D transforms or will-change often trigger it, while properties like opacity may create a stacking context but do not always guarantee a separate composited layer._

---

**Before optimizing**

<div class="grid grid-cols-2 gap-2">
<div>

```html
<html>
  <body>
    <h1>title</h1>
    <p>content</p>
  </body>
  <style>
    @keyframes move {
      from {
        left: 0;
      }
      to {
        left: 600px;
      }
    }
    p {
      position: absolute;
      animation: move 5s ease-in-out infinite alternate;
    }
  </style>
</html>
```

</div>
<div>

![composite-before](/composite-before.png)

</div>
</div>

---

**After optimizing**

<div class="grid grid-cols-2 gap-2">
<div>

```html
<html>
  <body>
    <h1>title</h1>
    <p>content</p>
  </body>
  <style>
    @keyframes move {
      from {
        transform: translateX(0);
      }
      to {
        transform: translateX(600px);
      }
    }
    p {
      position: absolute;
      animation: move 5s ease-in-out infinite alternate;
    }
  </style>
</html>
```

</div>
<div>

![composite-after](/composite-after.png)

</div>
</div>

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
