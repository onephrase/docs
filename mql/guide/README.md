# Getting Started

## Installation

### Embed as script

```html
<script src="https://unpkg.com/@onephrase/mql"></script>

<script>
// The above tag loads Mql into a global "OnePhrase" object.
const Mql = window.OnePhrase.Mql;
</script>
```

### Install via npm

```shell
$ npm i -g npm
$ npm i --save @onephrase/mql
```

#### Import
Mql is written in and distributed as standard JavaScript modules, and is thus imported only with the `import` keyword.

Mql works both in browser and server environments.

```js
// Node-style import
import Mql from '@onephrase/mql';

// Standard JavaScript import. (Actual path depends on where you installed Mql to.)
import Mql from './node_modules/@onephrase/mql/src/index.js';
```