# Getting Started

## Installation

### Embed as script

```html
<script src="https://unpkg.com/@onephrase/roseui"></script>

<script>
// The above tag loads RoseUI into a global "OnePhrase" object.
const RoseUI = window.OnePhrase.RoseUI;
</script>
```

### Install via npm

```shell
$ npm i -g npm
$ npm i --save @onephrase/roseui
```

#### Import
RoseUI is written in and distributed as standard JavaScript modules, and is thus imported only with the `import` keyword.

RoseUI works both in browser and server environments.

```js
// Node-style import
import RoseUI from '@onephrase/roseui';

// Standard JavaScript import. (Actual path depends on where you installed RoseUI to.)
import RoseUI from './node_modules/@onephrase/roseui/src/index.js';
```