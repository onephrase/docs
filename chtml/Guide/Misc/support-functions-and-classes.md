# The Composition API
Chtml offers certain methods for working with modules and bundles. We will demonstrate these methods against the simple template below.

```html
<template is=”chtml-bundle”>

  <div chtml-role=”user” chtml-ns=”html/badge/user“>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>

  <div chtml-role=”article” chtml-ns=”html/content/article“>
    <div>
      <chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user”></chtml-import>
        <div>
          <div chtml-role=”article-content”></div>
        </div>
    </div>
  </div>

  <chtml-import chtml-ns=”html/content/article/readonly“></chtml-import>

</template>
```

## The Chtml.import() Static Method
This method simply retrieves a module by namespace from across loaded bundles.

```js
var articleEl = Chtml.import(‘html/content/article’);
```

The result is the HTML element when found, or undefined, when not found. At this point, all applicable recomposition would have been made.

Below, importing the `html/content/article/readonly` module will apply inheritance-based composition from its `html/content/article` super namespace. And the element returned remains the `<chtml-import>` element.

```js
var readonlyArticleEl = Chtml.import(‘html/content/article/readonly’);
```

## The Chtml.from() Static Method
This method also works like the Chtml.import() method, but this time, used to retrieve elements from the within the document body, not bundles. Below, we retrieve a namespaced element from within the document body.

```html
<body>
  <chtml-import ondemand chtml-ns=”html/content/article/editable“></chtml-import>
</body>
```

```js
var editableArticleEl = Chtml.from(‘html/content/article/editable’);
```

The result is an HTML element that’s fully resolved to the html/content/article module. At this point, the temporary <chtml-import> element will be automatically replaced by the imported article module.

## The Chtml.ready() Static Method
This method is used to keep code execution in sync with the document’s “ready” state. It accepts a callback to be run when the DOM announces readiness – an event that indicates that the document tree has been initialized and safe to access.

```js
Chtml.ready(() => {
	// Put code here
});
```

This method also extends to wait for CHTML bundles to load. This is useful when running code that relies on external CHTML bundles. As it is, bundles have to be loaded in order to access their modules. In fact, a warning is issued on attempting to import modules while bundles are still loading.

```js
Chtml.ready(() => {
	var remoteModule = Chtml.import(‘html/content/article/readonly’);
});
```

To prevent waiting for bundles, pass false as the second argument to this method.

```js
Chtml.ready(callback, /*waitForBundles*/false);
```