# The Component API

The CHTML API is the official API for CHTML. It provides a complete suite of functions for working with the various aspects of CHTML. This API can be implemented in any language and platform. But the specification and examples in this guide are of JavaScript implementation.

## The Component Instance

The Chtml instance is used for translating a component’s Conceptual Model to an Object Model for programmatic use. In its basic form, a Chtml instance lets us access a CHTML component model as properties and objects.

```javascript
import Chtml from ‘@onephrase/chtml’;
```

If Chtml has been loaded via a script tag, it will be available in the global “OnePhrase” object.

```javascript
const Chtml = OnePhrase.Chtml;
```

A component instance is created via the Chtml constructor. The constructor takes a DOM element as it’s the component’s root element, and optionally accepts a params object that provides additional parameters for the instance.

Syntax:

```javascript
const component = new Chtml(el[, params]);
```

This creates an object that maps its properties to the underlying component’s nodes.

Examples:

```javascript
// Lets create a component on the DOM documentElement itself
// and access its nodes. 
const doc = new Chtml(document.documentElement);

let head = doc.get(‘head’);
let body = doc.get(‘body’);

// But we can better access nodes as properties using a proxied version of the component instance.
// (Details shortly.)
const _doc = doc.proxy();
let head = _doc.head;
let body = _doc.body;

// Here’s how we could write to the body node
body.innerText = ‘Hello beautiful world!’;
```

Notes:

* Proxied instances obtained from the proxy\(\) method give us the benefit of accessing nodes as properties while actually forwarding each access to the instance’s get\(\) method. In this mode however, instance methods would need to be prefixed with the $ character to prevent the proxy from forwarding the method name as node name.

```javascript
let body = _doc.body;
let body = _doc.$get(‘body’);
```

You can always tell whether or not an instance was proxied. Chtml proxies are created using the Js utilites from @onephrase/commons which allows us to work with the proxied object in other ways. If you have Commons installed...

```javascript
import {Js} from ‘@onephrase/commons’;

// Test if an instance is a proxy
if (Js.isProxy(_doc)) {
    // true
}

// Get the original instance object
let doc = Js.getProxyTarget(_doc);
```

* By default, an instance’s underlying root element is the native DOM element. But we could decide to have these DOM elements wrapped in a DOM abstraction object like jQuery. This is done via params.nodeCallback parameter. The params.nodeCallback should be a function that receives these DOM elements, as they are created, and returns their abstraction. 

```javascript
const doc = new Chtml(document, {nodeCallback: el => $(el));
```

To do this globally for all instances created from the Chtml class, the global Chtml.params object is used.

```javascript
Chtml.params.nodeCallback = el => $(el);
```

With jQuery objects now returned, we can now work with DOM elements with a more interesting syntax.

```javascript
let _doc = doc.proxy();
_doc.body.html(‘Hello from the other side!’);
```

And we can go on to extend jQuery with some custom methods.

Most examples in this documentation will use the jQuery DOM manipulation API. Note that this is just for demonstration purposes as CHTML does not ship with jQuery nor does it even require it.

* Nodes are lazy-loaded. So the DOM is accessed once for each node. The node is stored for subsequent access.

### The Chtml.from\(\) Method

In addition to creating instances from the Chtml constructor, the Chtml.from\(\) static method may be used. This method accepts the same augments as with the constructor, but also agrees to accept the root element as a CSS selector or even a HTML markup.

Syntax:

```javascript
const component = Chtml.from(input[, params]);
```

Examples:

```javascript
// Create an instance from a DOM object as usual
const doc = Chtml.from(document.documentElement);

// Create an instance from a selector
const body = Chtml.from(‘body’);
const component = Chtml.from(‘#some-element’);

// Create an instance from markup;
// the from() method will first automatically resolve this to an element.
let markup = ‘<div chtml-role=”comp”><span chtml-role=”node”></span></div>’;
const component = Chtml.from(markup);
```

## The Component Tree and Drilldown

With a mental model of a component’s nodes and its nested sub components, we can easily traverse the component tree. Here is how we could access an article’s author component from the article component itself.

```markup
<div chtml-role=”article” id=”article”>
  <div>
    <div chtml-role=”article-author user”>
      <div chtml-role=”user-avatar”></div>
      <div chtml-role=”user-name”></div>
    </div>
    <div>
      <div chtml-role=”article-description”></div>
    </div>
  </div>
</div>
```

```javascript
const article = Chtml.from(‘#article’).proxy();
let articleAuthor = Chtml.from(article.author).proxy();
let authorName = articleAuthor.name;
```

Well this could be much simpler. Chtml implements a params.drilldown property that provides a way to seamlessly access deep nodes in a component tree. With this feature on, all nodes are returned as a component instance instead of the default node type.

```javascript
const article = Chtml.from(‘#article’, {drilldown: true}).proxy();
let authorName = article.author.name;

The `authorName` variable is now rather holding a Chtml instance, not a DOM element. This makes it seamless to continue the drilldown. Using a `.el` at any point drops the chaining and returns the node’s underlying element.

```js
const article = Chtml.from(‘#article’, {drilldown: true}).proxy();
let author = article.author.el;
let authorNameElement = articleAuthor.name.el;

// With an object like jQuery as endpoint…
Chtml.from(‘#article’, {drilldown: true, nodeCallback: el => $(el)}).proxy();
article.author.el.html(‘John Doe’);
```

Notes:

* Drilldown always returns a Chtml instance for every node, whether or not the node has been defined as a component in HTML. This makes everything more predictable when accessing a component tree. The special el key is used to access the node’s underlying element.
* A component’s params object is transferred from component to sub components as they are created. So we don’t have to worry about passing params to deep nodes.
* Drilldown wisely employs the Chtml.from\(\) method to resolve a node.

