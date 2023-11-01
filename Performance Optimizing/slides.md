---
# theme: dracula
colorSchema: light
title: Performance Optimizing
titleTemplate: '%s'
favicon: /logo-256.png
transition: slide-left
lineNumbers: true
hideInToc: true
---

# Performance Optimizing 9 Tips You May Not Know

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

WHEN?
- DOM manipulation
- Log Something
- Gloabl variable assignment
- And so on ...

**sideEffects types: false | string | array\<string\>**

</div>
</div>


---
layout: quote
---

Reference Links:

> https://parceljs.org/features/scope-hoisting/

> https://blog.developer.adobe.com/optimizing-javascript-through-scope-hoisting-2259ef7f5994


---
layout: section
---

# Cascading Cache

打包粒度
级联缓存


---
layout: section
---

# Polyfill


---
layout: section
---

# Font
字体


---
layout: section
---

# Prexxx 
preconnect prefetch


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
layout: end
---

End
