# cache-component [![stability][0]][1]
[![npm version][2]][3] [![build status][4]][5] [![test coverage][6]][7]
[![downloads][8]][9] [![js-standard-style][10]][11]

Cached [bel][bel] components. Makes rendering elements _very fast™_. Analogous to
React's `.shouldComponentUpdate()` method, but only using native DOM methods.

Runs a `_render` function whenever arguments changed according to an `_update` function.  If the `_update` function determines an update is needed, a newly `_render`ed bel element is returned.  If an update is not needed, a `_proxy` element is returned instead.

## Features
- makes rendering elements _very fast™_
- implemented in only a few lines
- only uses native DOM methods
- Class based components, which offer a great place to store methods for re-use.

## Usage

```js
// Implementer API
var CacheComponent = require('cache-component')
var html = require('bel')

function CachedButton () {
  if (!(this instanceof CachedButton)) return new CachedButton()
  this._color = null
  CacheComponent.call(this)
}
CachedButton.prototype = Object.create(CacheComponent.prototype)

CachedButton.prototype._render = function (color) {
  this._color = color
  return html`
    <button style="background-color: ${color}">
      Click Me
    </button>
  `
}

// Override default shallow compare _update function
CachedButton.prototype._update = function (newColor) {
  return newColor !== this._color
}

var element = CachedButton()

let el = element('red') // creates new element
let el = element('red') // returns cached element (proxy)
let el = element('blue') // creates new element

```

```js
// Consumer API
var CachedButton = require('./cached-button.js')
var cachedButton = CachedButton()
document.body.appendChild(cachedButton.render('green'))
```

## API

### `CacheComponent.prototype()`
Inheritable CachedComponent prototype. Should be inherited from using
`CacheComponent.call(this)` and `prototype =
Object.create(CacheComponent.prototype)`.

Internal properties are:

- `this._proxy`: proxy element that's returned on subsequent
  `render()` calls that don't pass the `._update()` check.
- `this._isProxied`: used to keep track if the proxy element has been created or not.
- `this._element`: rendered element that should be returned from the
  `._render()` call.
- `this._hasWindow`: boolean if `window` exists. Can be used to create
  elements that render both in the browser and in Node.
- `this._args`: a reference to the arguments array that was used during the last `_render()` call.

### `CacheComponent.prototype._render([arguments])`
__Must be implemented.__ Render an HTML node with arguments. The Node that's returned is cached as
`this._element`.  Only called on first render and whenever you return `true` from `prototype._update()`.  You must return a DOM node from this function on every call.

### `CacheComponent.prototype._update([arguments])`
Return a boolean to determine if `prototype._render()`
should be called.  Not called on the first render.  Defaults to the following shallow compare function:

```js
CacheElement.prototype._update = function () {
  var length = arguments.length
  if (length !== this._args.length) return true

  for (var i = 0; i < length; i++) {
    if (arguments[i] !== this._args[i]) return true
  }
  return false
}
```

## Installation
```sh
$ npm install cache-component
```

## FAQ
### Where does this run?
Make sure you're running a diffing engine that checks for `.isSameNode()`, if
it doesn't you'll end up with super weird results because proxy nodes will
probably be rendered which is not what should happen. Probably make sure you're
using [morphdom][md] or [nanomorph][nm]. Seriously.

### What's a proxy node?
It's a node that overloads `Node.isSameNode()` to compare it to another node.
This is needed because a given DOM node can only exist in one DOM tree at the
time, so we need a way to reference mounted nodes in the tree without actually
using them. Hence the proxy pattern, and the recently added support for it in
certain diffing engines:
```js
var html = require('bel')

var el1 = html`<div>pink is the best</div>`
var el2 = html`<div>blue is the best</div>`

// let's proxy el1
var proxy = html`<div></div>`
proxy.isSameNode = function (targetNode) {
  return (targetNode === el1)
}

el1.isSameNode(el1)   // true
el1.isSameNode(el2)   // false
proxy.isSameNode(el1) // true
proxy.isSameNode(el2) // false
```

### How does it work?
[Morphdom][md] is a diffing engine that diffs real DOM trees. It runs a series
of checks between nodes to see if they should either be replaced, removed,
updated or reordered. This is done using a series of property checks on the
nodes.

Since [v2.1.0][210] `morphdom` also runs `Node.isSameNode(otherNode)`. This
allows us to override the function and replace it with a custom function that
proxies an existing node. Check out the code to see how it works. The result is
that if every element in our tree uses `cache-component`, only elements that have
changed will be recomputed and rerendered making things very fast.

`nanomorph`, which saw first use in choo 5, has supported `isSameNode` since it's conception.

### What's the exact difference between cache-element and nanocomponent?
- `cache-component` returns a proxy node if the arguments were the same. If arguments
  change, it'll rerender and return a new node.
- `nanocomponent` will render a new node initially and always return a proxy node on subsequent calls to `prototype.render`.  This means the component is responsible for mutating any internal changes. It also listens for the node to be
  unmounted from the DOM so it can clean up internal references, making it more
  expensive to use.

## License
[MIT](https://tldrlegal.com/license/mit-license)

[0]: https://img.shields.io/badge/stability-experimental-orange.svg?style=flat-square
[1]: https://nodejs.org/api/documentation.html#documentation_stability_index
[2]: https://img.shields.io/npm/v/cache-element.svg?style=flat-square
[3]: https://npmjs.org/package/cache-element
[4]: https://img.shields.io/travis/yoshuawuyts/cache-element/master.svg?style=flat-square
[5]: https://travis-ci.org/yoshuawuyts/cache-element
[6]: https://img.shields.io/codecov/c/github/yoshuawuyts/cache-element/master.svg?style=flat-square
[7]: https://codecov.io/github/yoshuawuyts/cache-element
[8]: http://img.shields.io/npm/dm/cache-element.svg?style=flat-square
[9]: https://npmjs.org/package/cache-element
[10]: https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square
[11]: https://github.com/feross/standard
[bel]: https://github.com/shama/bel
[md]: https://github.com/patrick-steele-idem/morphdom
[210]: https://github.com/patrick-steele-idem/morphdom/pull/81
