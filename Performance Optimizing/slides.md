---
colorSchema: light
title: Performance Optimizing
titleTemplate: '%s'
favicon: /logo-256.png
transition: slide-left
hideInToc: true
---

# Performance Optimizing 8 Tips You May Not Know

<p class="absolute right-30px bottom-30px">
  Author: Anson
</p>

<style>
h1 {
  background-image: linear-gradient(45deg, #4EC5D4 20%, #146b8c 60%);
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

**font-display**

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

**More Optimizing Methods**

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

    ...etc.

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

# Prexxx 
preconnect prefetch preload


---
layout: section
---

# JS load and execute
js 加载执行机制


---
layout: section
---

# requestIdleCallback


---
layout: section
---

# Image
图片


---
layout: section
---

# Compositing Layer
合成层优化


---
layout: quote
---

Reference Links:

> https://blog.developer.adobe.com/optimizing-javascript-through-scope-hoisting-2259ef7f5994
> https://philipwalton.com/articles/cascading-cache-invalidation/
> https://web.dev/articles/font-best-practices


---
layout: end
---

End
