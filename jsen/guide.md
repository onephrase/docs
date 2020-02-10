# Guide

## Installation

### Embed as script

```markup
<script src="https://unpkg.com/@onephrase/jsen/dist/main.js"></script>

<script>
// The above tag loads JSEN into a global "OnePhrase" object.
const Jsen = window.OnePhrase.Jsen;
</script>
```

### Install via npm

```text
$ npm i -g npm
$ npm i --save @onephrase/jsen
```

#### Import

JSEN is written in and distributed as standard JavaScript modules, and is thus imported only with the `import` keyword.

JSEN works both in browser and server environments.

```javascript
// Node-style import
import Jsen from '@onephrase/jsen';

// Standard JavaScript import. (Actual path depends on where you installed JSEN to.)
import Jsen from './node_modules/@onephrase/jsen/src/index.js';
```

## Examples

### Math expression

```javascript
// parse() a Math expression
var expr = '7 + 8';
var exprObj = Jsen.parse(expr);

// get result with eval()
console.log(exprObj.eval());

// convert to string anytime
console.log(exprObj.toString());
```

### Primitives and object notations

```javascript
// parse() a value
var expr = '10';
var exprObj = Jsen.parse(expr);

// get result with eval()
console.log(exprObj.eval());

// ------------

// parse() an array expression
var expr = '["New York City", "Lagos", "Berlin"]';
var exprObj = Jsen.parse(expr);

// get result with eval()
console.log(exprObj.eval());

// ------------

// parse() an object expression
var expr = '{city1: "New York City", city2: "Lagos", city3: "Berlin"}';
var exprObj = Jsen.parse(expr);

// get result with eval()
console.log(exprObj.eval());

// ------------

// convert to string anytime
console.log(exprObj.toString());
```

### Eval\(\) with a context object

```javascript
// parse() a logical expression with references to properties of a context object
var expr = 'age < 18 ? fname + " " + lname + " does not meet the age requirement!" : fname + " " + lname + " is old enough!"';
var exprObj = Jsen.parse(expr);

// eval() with a context object
var context = {fname: "John", lname: "Doe", age: 24};
console.log(exprObj.eval(context));

// eval() same exprObj with another context object
var context2 = {fname: "John2", lname: "Doe2", age: 10};
console.log(exprObj.eval(context2));

// convert to string anytime
console.log(exprObj.toString());
```

### Call a method

```javascript
// parse() a logical expression with embedded calls
var expr = '"Today is: " + date().toString()';
var exprObj = Jsen.parse(expr);

// eval() with the given context
var context = {date:() => (new Date)};
console.log(exprObj.eval(context));

// convert to string anytime
console.log(exprObj.toString());
```

### ES6-style function declaration supported

```javascript
// parse() a function declaration that will materialize into a real function
var expr = '(arg1, arg2) => {return arg1 + arg2 + argFromContext}';
var exprObj = Jsen.parse(expr);

// eval() to materialize the declared function
var context = {argFromContext: 10};
var sumFunction = exprObj.eval(context);

// Call materialized function now
console.log(sumFunction(30, 20));

// convert to string anytime
console.log(exprObj.toString());
```

### Assignment operation supported with the "mutates" flag set to true

```javascript
// parse() an assignment expression.
// Multiple expressions supported.
var expr = 'prop2 = "val2"; prop3 = "val3"';
var exprObj = Jsen.parse(expr, {mutates: true});

// eval() with a context object
var context = {prop1: "val1"};
console.log(exprObj.eval(context));

// Confirm the new properties have been set
console.log(context);

// convert to string anytime
console.log(exprObj.toString());
```

### Delete operation supported with the "mutates" flag set to true

```javascript
// parse() a delete expression.
var expr = 'delete prop1; delete prop1';
var exprObj = Jsen.parse(expr, {mutates: true});

// eval() with a context object to mutate
var context = {prop1: "val1", prop2: "val2", prop3: "val3"};
console.log(exprObj.eval(context));

// Confirm the new property has been deleted
console.log(context);

// convert to string anytime
console.log(exprObj.toString());
```

### Multiple expressions and Comments supported

```javascript
// Line comments
var expr = `

/**
 * Block comments
 */

// Single line comments
delete prop1; delete /*comment anywhere*/prop3;
`;

var exprObj = Jsen.parse(expr, {mutates: true});
// To obtain the comments...
let {comments} = Jsen.lex(expr, {mutates: true});
console.log(comments);

// eval() with a context object to mutate
var context = {prop1: "val1", prop2: "val2", prop3: "val3"};
console.log(exprObj.eval(context));

// Confirm the new properties have been deleted
console.log(context);

// convert to string anytime
console.log(exprObj.toString());
```

## Extending JSEN

### Utility functions

It is obviously possible to call methods of an object in an expression. And fortunately, some host langauges like JavaScript happen to implement their primitives as objects internally. This is why we can invoke methods on primitives like strings and numbers; e.g: "bannana".toUpperCase\(\). We are aslo able to extend the prototype of these objects with whatever custom method we want.

Unfortunately, this luxury isn't possible with some other host languages like PHP where primitives are simply primitives. So while "bannana".toUpperCase\(\) would evaluate in a JavaScript environment, it wouldn't in a PHP environment. To achieve universality of expressions, JSEN allows us to supply these function definitions separately. These functions are called utility functions and are used on encountering expressions like "bannana".toUpperCase\(\) where primitives do not live as objects or where a certain method is not defined on the value's prototype.

Utility functions for a value type are defined together in the format below. And each function recieves the encoutered value as its first parameter. So the string functions, for example, will recieve "bannana" as their first parameter.

```javascript
// Define utilities...
var utility = {
    // String functions
    Str: {
        toUpperCase: (inputString) => {
        },
        toLowerCase: (inputString) => {
        },
    },
    // Array functions
    Arr: {
        flatten: (inputArray) => {
        },
    },
    // Object functions
    Obj: {
        each: () => {
        },
    },
    // Number functions
    Num: {...},
};

// Use in an expression...
// We will now assign utility to JSEN's call parser...
// That's where it is needed.
import {Call} from '@onephrase/jsen';
Call.utils = utility;

// Using the utility.Arr.flatten method now...
// parse() a nested array notation and flatten.
var cities = {cities: ['New York City', 'Lagos', 'Berlin', ['two', 'more', 'cities'],]};
var expr = 'cities.flatten()';
var exprObj = Jsen.parse(expr);

// get result with eval()
console.log(exprObj.eval());

// ----------------------

// Using the utility.Obj.each method now...
// parse() an object expression and log each entry to the console.
var cities = {cities: {city1: 'New York City', city2: 'Lagos', city3: 'Berlin'}, console: console};
var expr = 'cities.each((key, val) => {console.log(key, val)})';
var exprObj = Jsen.parse(expr);

// get result with eval()
console.log(exprObj.eval());
```

By default, JSEN comes with certain preconfigured utilities. The official JSEN JavaScript library uses the @onephrase/commons library as its default utilities. So all the @onephrase/commons's string me are usable in expressions. JSEN implementations in other languages would want to implement similar functions where possible, so we can use expressions that work everywhere JSEN works.

### Syntax Extension

JSEN is implemented as a parser combinator that coordinates specialized classes in precise order. Each class handles one type of expression. For example, the Math class is what captures a math expression during parsing, evaluates the math during evaluation and stringifies the math during seralization. The comparison class does the same for comparison expressions.

Now these classes can be replaced with custom classes that understand a different syntax for the same type of expression. Details are coming in the official documentation.

