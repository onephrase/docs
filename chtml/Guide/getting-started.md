# Getting Started

## Installation

### Embed as script

```html
<script src="https://unpkg.com/@onephrase/chtml"></script>

<script>
// The above tag loads CHTML into a global "OnePhrase" object.
const Chtml = window.OnePhrase.Chtml;
</script>
```

### Install via npm

```shell
$ npm i -g npm
$ npm i --save @onephrase/chtml
```

#### Import
CHTML is written in and distributed as standard JavaScript modules, and is thus imported only with the `import` keyword.

To import CHTML for a project that renders on the browser...
(That is, in the context of the browser's `window` object.)

```js
// Node-style import
import Chtml from '@onephrase/chtml';

// Standard JavaScript import. (Actual path depends on where you installed CHTML to.)
import Chtml from './node_modules/@onephrase/chtml/src/index.js';
```

To import CHTML for a project that renders only on the server...
(Requirements for setting up server-side rendering coming soon.)

```js
// Node-style import
import Chtml from '@onephrase/chtml/src/server-entry';

// Standard JavaScript import. (Actual path depends on where you installed CHTML to.)
import Chtml from './node_modules/@onephrase/chtml/src/server-entry.js';
```

## Next Steps
Learn the concepts and write your first CHTML component.

*	[Strucutural Concepts](/chtml/guide/strucutural-concepts/)
*	[Behavioural Concepts](/chtml/guide/behavioural-concepts/)
*	[Compositional Concepts](/chtml/guide/compositional-concepts/)
