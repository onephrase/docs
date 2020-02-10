# Getting Started

## Installation

### Embed as script

```html
<script src="https://unpkg.com/@onephrase/commons"></script>

<script>
// The above tag loads Commons into a global "OnePhrase" object.
const Commons = window.OnePhrase.Commons;
</script>
```

### Install via npm

```shell
$ npm i -g npm
$ npm i --save @onephrase/commons
```

#### Import
Commons is written in and distributed as standard JavaScript modules, and is thus imported only with the `import` keyword.

Commons works both in browser and server environments.

```js
// Node-style import
import Commons from '@onephrase/commons';

// Standard JavaScript import. (Actual path depends on where you installed Commons to.)
import Commons from './node_modules/@onephrase/commons/src/index.js';
```

### Basic Usage

```js
// Arr.flatten
var cities = ['New York City', 'Lagos', 'Berlin', ['two', 'more', 'cities'],];
console.log(Arr.flatten(cities));
```

```js
// Obj.each
var cities = {city1: 'New York City', city2: 'Lagos', city3: 'Berlin'};
Obj.each(cities, (key, val) => {
	console.log(key, val);
	// Returning false stops further iteration
	return false;
});
```
