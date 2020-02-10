# The Component Model
A component is simply an element that is designed or designated for a specific role or functionality. Components may be designed to work with other key elements called nodes. These work as subroles within their owning component or scope. This scope-node relationship is simply the component model.

Component models are formed at markup level, either explicitly by using a role-based markup pattern or implicitly as based on standard HTML and ARIA semantic models.

## Explicit Models
Explicit component models are formed using the `chtml-role` attribute. A rolename represents a functional scope

```html
<div chtml-role=”article”></div>
```

Certain elements can be associated with this component as nodes using the `chtml-role` attribute. This time, the rolename (or subrole) is prefixed with the scope name (or owning role).

```html
<div chtml-role=”article”>
  <div chtml-role=”article-author”></div>
  <div chtml-role=”article-content”></div>
</div>
```

And nodes could be anywhere within the component’s subtree. 

```html
<div chtml-role=”article”>
  <div>
    <div chtml-role=”article-author”></div>
    <div>
      <div chtml-role=”article-content”></div>
    </div>
  </div>
</div>
```

Now we have a clear component model that we can always bank on regardless of a component’s implementation details.

```html
article
      |--- author
      |--- content
```

### Nested Components and Scope Boundaries
Components can be nested. So a node may constitute a component of its own, and establish its own scope and have its own nodes.

Below, the article-author node is also a component.

```html
<div chtml-role=”article”>
  <div>
    <div chtml-role=”article-author user”>
      <div chtml-role=”user-avatar”></div>
      <div chtml-role=”user-name”></div>
    </div>
    <div>
      <div chtml-role=”article-content”></div>
    </div>
  </div>
</div>
```

This produces the following component model.

```html
article
      |--- author (user)
      |          |--- avatar
      |          |--- name
      |--- content
```

Where two identical scopes (components) are nested, nodes are associated with the scope that’s closest to them up the hierarchy.

```html
<div chtml-role=”article anotherrole”>
  <div>
    <div chtml-role=”article-author”></div>
    <div chtml-role=”article-content”></div>
    <div chtml-role=”article-brief article”>
      <div chtml-role=”article-date”></div>
      <div chtml-role=”article-description”></div>
      <div chtml-role=”anotherrole-othernode”></div>
    </div>
  </div>
</div>
```

The nested articles produce the following component model.

```html
Article (anotherrole)
      |--- author
      |--- content
      |--- othernode
      |--- brief (article)
              |--- date
              |--- description
```

### Special Nodes
The element on which a functional scope is established is called the root node. While a component references its nodes by their individual role, it reserves an invisible reference to this root node with the special `el` role. The `el` role is thus reserved and cannot be used as a role name.

Nested components have a parent-child relationship. Now, for every child component, there exists an invisible reference to an inherent parent node on the special underscore character `_`. The underscore character is thus reserved and cannot be used as a role name.

### Related Nodes
Sometimes a component may need to reference elements outside of its root as related nodes. References to related nodes are declared together in the `chtml-related` attribute. This attribute follows a CSS-like convention of key/value pairs, and pairs are separated by a semicolon.

```html
<div chtml-role=”collapsible” chtml-related=”nodeName1:reference1; nodeName2:reference2”></div>
```

**References Are Node-To-Node Paths to Foreign Nodes.** A related node is actually a node contained in another component outside of the component’s scope. A reference is a node-to-node path to this foreign node, written in dot (.) or bracket [] notation.

Below, an app has multiple components. Now a collapsible component is referencing a button node in `app-bar` as its control node.

```html
<body chtml-role=“app”>
  <header chtml-role=”app-bar header”>
    <div chtml-role=”header-button”></div>
  </header>
  <main chtml-role=”app-content collapsible” chtml-related=”control:_.bar.button”></main>
</body>
```

Our path begins with a back reference to the parent component – app. From here, we could easily traverse down the component tree.

Since related nodes are usually out of scope, paths may always begin with a back reference.

In another scenario, we are moving two levels up the hierarchy to reference our button. We also played with the bracket notation.

```html
<body chtml-role=“app”>
  <header chtml-role=”app-bar header”>
    <div chtml-role=”header-button”></div>
  </header>
  <main chtml-role=”app-main”>
    <div>
      <div chtml-role=”main-content collapsible” chtml-related=”control:_._.bar[‘button’]”></div>
    </div>
  </main>
</body>
```

Notice the quoted key in our brackets! Unquoted keys are actually references of their own. Later, in the section for parameters and cascading, we will see other properties of a component that we can reference this way in our paths. These properties are usually not seen in markup (just as the parent reference isn’t a role name in markup.) We’ll also see how paths can be dynamic by using logical expressions.

**References Can Be Defined in a Params Sheet.** Inline references (related nodes listed in the `chtml- related` attribute), like inline CSS, can quickly begin to look congested. CHTML lets us define them in a Data Block type of script element where these can be listed more comfortably, just as with a style sheet. This script element is called JSEN Params Sheet (jsen-p for short) and is placed at the root of the component.

```html
<div chtml-role=”main-content collapsible”>

  <script type=”text/jsen-p”>
  @related {
      control: _._.bar[‘button’];
  }
  </script>

</div>
```

Where references are declared both inline and in JSEN-P, the combined parameters are used in a cascaded manner, usually with inline parameters taking precedence on duplicate parameters. The cascaded nature of parameters is covered in the section for cascading.

## Implicit Models
Certain elements in HTML have been naturally designed to work together in a predefined manner. Usually, one element establishes the functional role or scope, while the others serve as rubroles or nodes. These natural relationships or implicit models are automatically recognized in CHTML and need not be explicitly designed.

Not commonly known, every element has a design note or specification that states the element’s **category** and dictates what and what can serve as its content – better known as the element’s **content model**. For example, the `<html>` element must only have the `<head>` and `<body>` tags as its direct children. Okay, that’s common knowledge, but really, that’s the specs dictating.

Content models are automatic component models in CHTML; here an element’s category, if defined, forms the scope, and its permissible elements, serve as nodes. Below are examples of these implicit models. The same rule holds for other models as defined in their standards specs.

The `<html>` element’s content model permits two elements that must also be direct children.

```html
<html>
  <head></head>
  <body></body>
</html>
```

This gives us the component model:

```html
html
      |--- head
      |--- body
```

The `<head>` element’s content model permits only elements that have been categorized a Metadata elements.

Below, the following Metadata elements can only be used once in the `<head>` element.

```html
<head>
  <title></title>
  <base />
</head>
```

This gives us the following component model

```html
head
      |--- title
      |--- base
```

Now certain elements are permitted to appear any number of times within their permissible scope. The `<head>` element, again, permits multiple `<meta>` elements.

```html
<head>
  <title></title>
  <meta name=”” content=”” />
  <meta name=”” content=”” />
  <meta name=”” content=”” />
  <base />
</head>
```

This gives us the following component model. (Notice that the meta node becomes a list.)

```html
head
      |--- title
      |--- meta (3)
      |--- base
```

### Nested Scopes Create a Boundary
Where two identical scopes are nested, nodes are associated with the scope that’s closest to them up the hierarchy. The `<body>` element and the `<blockquote>` element are a good example of nested identical scopes.

The `<body>` element is categorized as Sectioning Root which permits elements categorized as Flow Content. The `<blockquote>` element is one of those Flow Content elements. (Most elements are categorized as Flow Content – `h1`, `div`, `p`, etc.) Now the `<blockquote>` element is also a Sectioning Root that could have its own Flow Content. So below, every element is a node to the body’s Sectioning Root scope except those within blockquote’s Sectioning Root scope.

```html
<body>
  <div>
    <h1></h1>
    <div>
      <p></p>
    </div>
    <blockquote>
      <h1></h1>
      <div></div>
    </blockquote>
    <div></div>
  </div>
</body>
```

This gives us the following component model.

```html
body
      |--- div (3)
      |           [0]
      |               : (this model is omitted for simplicity)
      |           [1]
      |               |--- p (1)
      |           [2]
      |--- h1 (1)
      |--- blockquote (1)
      |           [0]
      |                |--- h1 (1)
      |                |--- div (1)
      |--- p (1)
```

Another form of the Sectioning Root category is Sectioning Content. Elements in this category are `<article>`, `<aside>`, `<nav>`, and `<section>`. Notice how the `<header>` element, being a Flow Content element, is semantically associated below. Also note that there cannot be more than one within its permissible scope.

```html
<body>
  <div>
    <header></header>
    <article>
      <header></header>
      <div></div>
    </article>
  </div>
</body>
```

This gives us the following component model.

```html
body
      |--- div (1)
      |           [0]
      |               : (this model is omitted for simplicity)
      |--- header
      |--- article (1)
                  [0]
                        |--- header
                        |--- div (1)
```

### ARIA Roles Are Automatically Recognized
The implicit and explicit roles defined in the ARIA specification are automatically recognized in CHTML. Elements that have an explicit or implicit ARIA role can be accessed by both their rolename and their tagname.

```html
<body>
  <div>
    <div role=”header”></div>
    <article>
      <div role=”header”></div>
      <div></div>
    </article>
  </div>
</body>
```

This gives us the following component model.

```html
body
      |--- div (2)
      |           [0]
      |               : (this model is omitted for simplicity)
      |           [1]
      |--- header
      |--- article (1)
                  [0]
                        |--- div (2)
                        |--- header
```

The `<section>` element has the implicit ARIA role region. So both the names section and region will point to the `<section>` element.

```html
<body>
  <section>
  </section>
  <section>
  </section>
</body>
```

This gives us the following component model.

```html
body
      |--- section/region (2)
```

The `<ul>` element has the implicit ARIA role list. The `<li>` element has the implicit ARIA role listitem. Here’s how it looks.

```html
<body>
  <ul>
    <li></li>
    <li></li>
  </ul>
</body>
```

This gives us the following component model. Also notice that the `<li>` element do not reflect in the `<body>` as their permissible scope ends with the `<ul>`.

```html
body
      |--- ul/list (1)
        	[0]
		      |--- li/listitem (2)
```
