# Getting Started

## Installation

### Embed as script

```markup
<script src="https://unpkg.com/@onephrase/observable/dist/main.js"></script>

<script>
// The above tag loads Observable into a global "OnePhrase" object.
const Observable = window.OnePhrase.Observable;
</script>
```

### Install via npm

```text
$ npm i -g npm
$ npm i --save @onephrase/observable
```

#### Import

Observable is written in and distributed as standard JavaScript modules, and is thus imported only with the `import` keyword.

Observable works both in browser and server environments.

```javascript
// Node-style import
import Observable from '@onephrase/observable';

// Standard JavaScript import. (Actual path depends on where you installed Observable to.)
import Observable from './node_modules/@onephrase/observable/src/index.js';
```

### Basic Usage

```javascript
// The object to observe...
var obj = {fruit: 'apple'};

// Wrap this in an Observable Instance...
let _obj = new Observable(obj);
// and mutate via the Observable instance...
// Set prop
_obj.set('fruit', 'orange');
// Delete prop
_obj.del('fruit');
// Read prop
var fruit = _obj.get('fruit');
// Test prop
if (_obj.has('fruit')) {
}

// ---------------------
// Switch to syntax sugar,
// using ES6 proxies...
// ---------------------

// Convert the Observable instance to a proxied object...
var _obj = _obj.proxy();
// now we can use normal object operators on the Observable;
// all operations will be forwarded to the original Observable instance...
// Set prop
_obj.fruit = 'orange';
// Delete prop
delete _obj.fruit;
// Read prop
var fruit = _obj.fruit;
// Test prop
if ('fruit' in _obj) {
}

// ---------------------
// The Observable's instance methods can still
// be used in a proxy mode, 
// but this time, with a prefix...
// ---------------------

// Set prop
_obj.$set('fruit', 'pear');
```

### Observing an Object

```javascript
let obj = {};
let _obj = new Observable(obj);
// Use in proxy mode
_obj = _obj.proxy();

_obj.$observe((newValues, oldValues, event) => {
    console.log(newValues);
    console.log(oldValues);
    console.log(event);
});

// Modify the wrapped object and watch your console 
_obj.fruit = 'bannana'; 

// Notice that the instance's observe() method is here prefixed with $.
// (We'll get to the implementation note in a moment.)
```

### Observing an Array

```javascript
let arr = [];
let _arr = new Observable(arr);
// Use in proxy mode
_obj = _obj.proxy();

_arr.$observe((newValues, oldValues, event) => {
    console.log(newValues);
    console.log(oldValues);
    console.log(event);
});

// Modify the wrapped array and watch your console 
_arr[0] = 'bannana'; 
_arr[1] = 'mango'; 

// Call an array method
// (Notice that these are the array's prototype methods.)
_arr.push('orange');
```

### Observing One or More Properties

```javascript
// We observe ALL propertes when we do...
_obj.$observe((newValues, oldValues, event) => {
    // newValues and oldValues will be key/value
    // objects of new values and old values respectively
});

// We can observe a list of properties...
_obj.$observe(['fruit', 'isFavourite'], (newValues, oldValues, event) => {
    // newValues and oldValues will be
    // objects of changes on any of the observed properties
});

// We can observe a single property...
_obj.$observe('fruit', (newValue, oldValue, event) => {
    // newValue and oldValue will directly represent
    // the property's new value and old value respectively
});

// ---------------------
// Everything above holds true for arrays...
// ---------------------

// To observe a specific key, for example...
_arr.$observe(2, (newValue, oldValue, event) => {});
```

### Publishing One or More Changes

```javascript
// We publish a single change when we do...
_obj.fruit = 'Orange';

// This is the same as using the observable's set() method...
_obj.$set('fruit', 'Orange');

// ---------------------
// We can publish multiple changes
// ---------------------

// The listed properties below will be published with a shared value
_obj.$set(['fruit2', 'fruit3'], 'mango');

// We can provide a key/value map...
_obj.$set({
    fruit2: 'Mango',
    fruit3: 'Orange',
});

// ---------------------
// Everything above holds true for arrays...
// ---------------------

// For a single key...
_arr[0] = 'Orange';
// Or _arr.$set(0, 'Orange');

// For multiple keys...
_arr.$set([1, 2], 'Orange');

// For a key/value map...
// (an object must be used)
_arr.$set({
    1: 'Mango',
    2: 'Orange',
});
```

### Observing Deep Changes

It is possible to observe changes on nested objects or arrays. They just need to be an observable instance themselves and their changes will bubble up to the root instance.

```javascript
let obj = {};
let _obj = new Observable(obj);
// Use in proxy mode
_obj = _obj.proxy();

let childObj = {};
let _childObj = new Observable(childObj);
// Use in proxy mode
_childObj = _childObj.proxy();

// So we observe deep changes with the allowBubbling option set to true
_obj.$observe((newValues, oldValues, event) => {
    console.log(newValues);
}, {allowBubbling: true});

// Set the _childObj
_obj.child = _childObj; 
// Now changes in _childObj will bubble up into _obj
// So with the assignment below, _childObj will report a change on its "brand" property
// while _obj will report a change on its "child" property.
_childObj.brand = 'Apple';

// How can a handlerFunction differentiate between direct change events and bubbling events?
// Bubbling cases are reported in a handlerFunction's event parameter.
_obj.$observe((newValues, oldValues, event) => {
    console.log(event.bubbling);
    // Should give ["child.brand"]
}, {allowBubbling: true});

// To observe a specific deep property...
_obj.$observe('child.brand', (newValue, oldValue, event) => {
    console.log(newValue);
}, {allowBubbling: true});

// To observe a mix of direct and deep properties...
_obj.$observe(['fruit', 'child.brand'], (newValues, oldValues, event) => {
    console.log(newValues);
}, {allowBubbling: true});
```

### Unpublishing Properties

For most use cases, setting a property to undefined or some other falsey value is enough to regard it as unavailable. Some other use cases might be very sensitive to object keys and a property would need to be completely unset from the object to be really unavailable. Deleting a property/key or multiple properties/keys is easy.

```javascript
// Using the delete keyword
delete _obj.fruit;

// Using the observable's del() method
_obj.$del('fruit');

// Now this accepts multiple properties/keys to delete
_obj.$del(['fruit', 'isFavourite']);

// ---------------------
// Everything above holds true for arrays...
// ---------------------

// Using the delete keyword, as one example
delete _arr[2];
// Or _arr.$del(2);

// Using the array's own mutation methods
_arr.pop();
_arr.splice(1, 1);

// ---------------------
// Property deletion triggers observers too...
// ---------------------

// We'll get notified both when the fruit's value changes
// and when it is completely deleted
_obj.$observe('fruit', (newValue, oldValue, event) => {
    // When deleted, newValue will become undefined
    // But there's a better way of detecting deletions as we'll soon see
});

delete _obj.fruit;

// Confirm that properties have been deleted.
// Get all keys...
let keys = Object.keys(_obj);
```

### Observing Property Entries and Exits

The first time a property is set, it is said to make an entry. When deleted, a property is said to have made an exit. These events aren't the same as regular value changes, so they are reported differently.

```javascript
// The first time a property is set, our handlerFunction's newValue
// parameter recieves the value while the oldValue parameter reports undefined.
// But oldValue being undefined isn't really unique to a property's entry
// as it is possible fot a property to have had undefined as its previous, real value.

// Entries are best detected in the event parameter.
_obj.$observe('fruit', (newValue, oldValue, event) => {
    if (event.entries.includes('fruit')) {
    }
});

_obj.username = 'foo';

// ---------------------

// In a similar way, when a property is deleted, our handlerFunction's oldValue
// parameter holds its last value before deletion while the newValue parameter reports undefined.
// Here again, newValue being undefined doesn't exclusive to a property's exit as properties can
// actually be assigned undefined as a value.

// Exits are also best detected in the event parameter.
_obj.$observe('fruit', (newValue, oldValue, event) => {
    if (event.exits.includes('fruit')) {
    }
});

delete _obj.username;
```

### Observing Switching Events

By default, the handlerFunction is triggered on a property's change of value. So we get notified on whatever the new value is. But we can observe a property's state change between truthy and falsey states as it switches values, using the options.pulse parameter. \(The observe\(\) method accepts an options object as an optional parameter after the functionHandler parameter.\)

```javascript
// Observe a property's state.

// With options.pulse === 1, we're notified when
// the property switches from a falsey state to a truthy state.
_obj.$observe('isFavourite', (newValue, oldValue, event) => {
}, {pulse: 1});

// This will trigger our handler as "isFavourite" becomes truthy...
_obj.isFavourite = true;
// This WON'T trigger our handler as "isFavourite" is still maintaining a truthy state...
_obj.isFavourite = 'yes';
// This WON'T trigger our handler as "isFavourite" switches to a falsey state...
_obj.isFavourite = 0;
// And this WON'T...
_obj.isFavourite = false;

// Convsersely, with options.pulse === 0, we're notified when
// the property switches from a truthy state to a falsey state.
_obj.$observe('isFavourite', (newValue, oldValue, event) => {
}, {pulse: 0});

// This WON'T trigger our handler as "isFavourite" maintains its falsey state (from an initial undefined state...)
_obj.isFavourite = false;
// This WON'T trigger our handler as "isFavourite" switches to a truthy state...
_obj.isFavourite = 1;
// This will trigger our handler as "isFavourite" switches to a falsey state...
_obj.isFavourite = 0;
// And this WON'T...
_obj.isFavourite = false;
```

### Unobserving Properties

It is easy to unobserve an object.

```javascript
// Unobserve everything.
// This frees the instance of all observers
_obj.$unobserve();

// Unobserve a property.
// This works if the property was observed alone
_obj.$unobserve('fruit');

// Unobserve multiple properties.
// This works if the properties were observed together
// But the order doesn't matter now...
_obj.$unobserve(['isFavourite', 'fruit']);

// ---------------------
// We can unobserve an object with more specificity.
// This saves us of unntentionally unbserving similar properties
// observed by other parts of our code.
// ---------------------

// Remove all observers made with the following handlerFunction instance.
// (The handlerFunction here must be the exact original instance.)
_obj.$unobserve(null, handlerFunction);

// Remove all observers that are using the following options.
// (The options object here must not necessarily be the exact original instance;
// and the order of properties doesn't matter.)
_obj.$unobserve(null, null, {allowBubbling: true});

// Remove the observer matching the combination
// of the following properties and handlerFunction instance.
_obj.$unobserve(['isFavourite', 'fruit'], handlerFunction);

// Remove the observer matching the combination
// of the following properties, the handlerFunction instance, and the options
_obj.$unobserve(['isFavourite', 'fruit'], handlerFunction, {allowBubbling: true});

// ---------------------
// Unobserve by tag.
// ---------------------

// To make it easier to remove observers in group, the observe() method allows us to
// tag observers. This tag can now be used to remove all associated observers.
// (The observe() method accepts the optional tag parameter after the options parameter.)

// Tag multiple observers with a name
_obj.$observe(props, handlerFunction1, options, 'tagname');
_obj.$observe(props, handlerFunction2, options, 'tagname');

// Remove all observers bound with the tag
_obj.$unobserve(null, null, null, 'tagname');

// ---------------------
// Using the observer.remove() method.
// ---------------------

// It happens that the observe() method returns an observer object that represents the intended observation.
// This object features a remove() method that performs a self-removal on the Observable instance.

// Save the observer object...
let observer = _obj.$observe(props, handlerFunction);
// Execute a self-removal...
observer.remove();
```

### Observers Disposition

Sometimes, a notifying code wants to know what observers have to say to inform its next action. And sometimes, certain handlers are mutually exclusive to other handlers, and it becomes necessary to stop an event from reaching other handlers. An observer's disposition can be sent back in a number of ways.

```javascript
// ---------------------
// Using the event.preventDefault() method.
// ---------------------

// Similar to the DOM's Event.preventDefault() method,
// this tells the notifying code not to continue with its default action.
_obj.$observe('fruit', (newValue, oldValue, event) => {
    // Do something...
    event.preventDefault();
});

// ---------------------
// Using the event.stopPropagation() method.
// ---------------------

// Similar to the DOM's Event.stopPropagation() method,
// this stops the event from reaching other handlers.
_obj.$observe('fruit', (newValue, oldValue, event) => {
    // Do something...
    event.stopPropagation();
});

// ---------------------
// Returning false.
// ---------------------

// Similar to returning false on DOM events,
// this BOTH stops propagation and prevents default
_obj.$observe('fruit', (newValue, oldValue, event) => {
    // Do something...
    return false;
});

// ---------------------
// Returning a Promise.
// ---------------------

// To sync the notifying code with an observer's timeline,
// a promise can be returned. The notifying code can make a decision on
// the promise's resolution or rejection
_obj.$observe('fruit', (newValue, oldValue, event) => {
    // even.stopPropagation() may be called as normal
    // But calling even.preventDefault() will invalidate the returned promise
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve();
        }, 1000);
    });
});

// Using the event.promise() method
_obj.$observe('fruit', (newValue, oldValue, event) => {
    event.promise(new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve();
        }, 1000);
    }));
});
```

#### Using observersDisposition

When multiple handlers return a promise, the combined promises are returned using Promise.all\(\). When a handler prevents default with a return-false or by calling event.preventDefault\(\) all promises are ignored and false is returned to the notifying code.

```javascript
// The notifying code recieves observers' disposition
var event = _obj.$set('fruit', value);
if (event.defaultPrevented) {
    // event.preventDefault() has been called
    // Or false was returned somewhere
} else if (event.propagationStopped) {
    // event.stopPropagation() has been called
    // Or false was returned somewhere
} else {
    var promises = event.promises;
    if (promises) {
        promises.then(() => {
            console.log('All done!');
        });
    }
}
```

### Proxy-related Utilities

Observable uses the @onephrase/commons as its utility library. It comes with functions for working with proxies.

Observable's proxies are created with Commons's Js.proxy\(\) method. Proxies created by this function can be later inspected with the other proxy-related utilities.

Here are examples:

```javascript
import {Js} from '@onephrase/commons';

// To know if an observable instance is a proxy...
// It would throw an error to say...
if (_obj instanceof Proxy) {
}

// Commons' Js.isProxy() function does the job
// as proxied observable instances are made from Js.proxy().
if (Js.isProxy(_obj)) {
}

// To obtain the real observable instance from its Proxy shell...
let realObservable = Js.getProxyTarget(_obj);

// To always use the real observable instance...
let realObservable = Js.isProxy(_obj) ? Js.getProxyTarget(_obj) : _obj;
```

