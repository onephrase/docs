# Imports and Recomposition, Inheritance and Cascading
CHTML makes it seamless to reuse a defined component either in whole or in part. This helps us build entire applications with much fewer components.

## Imports
CHTML imports are a way to place anything anywhere in a HTML document. The idea is to use a temporary element to import the needed element or component. This temporary element is the chtml-import element.

```html
<chtml-import chtml-ns=”html/content/article/readonly”></chtml-import>
```

Imports use the chtml-ns attribute to reference their source element. The import element itself gets replaced by the imported element; so at best, an import element is only a placeholder.

Using imports, we can easily compose a larger component with smaller components. An article component, for example, can now simply import a user component as its author node.

```html
<template is=”chtml-bundle”>

  <div chtml-role=”user” chtml-ns=”html/badge/user“>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>

  <div chtml-role=”article” chtml-ns=”html/content/article/readonly“>
    <div>
      <chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user”></chtml-import>
        <div>
          <div chtml-role=”article-content”></div>
        </div>
    </div>
  </div>

</template>
```

Actually, anything can be imported – anything tag-based – as long it is placed in a CHTML bundle and has a namespace.  So although we like to refer to things as components, a more general name for them is module – CHTML module. A bundle is composed of modules.

On this general note, a bundle could look like:

```html
<template is=”chtml-bundle”>

  <!—components -->
  <div chtml-role=”user” chtml-ns=”html/badge/user“>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>
  
  <!—Media: images, videos, etc -->
  <img src=”/assets/img/brand/logo.png” chtml-ns=”img/brand/logo” />

  <!— Media with data-URLs. -->
  <!-- Good for preloading resources -->
  <img src=”data:image/png,%89PNG%0D%0A…” chtml-ns=”img/brand/logo2” />

  <!—SVG, possibly SVG icons -->
  <svg chtml-ns=”svg/brand/logo/24x24” >
    <path />
  </svg>

  <!—Stylesheets -->
  <style type=”text/css” chtml-ns=”css/badge” >
  /* rules */
  </style>

</template>
```

Now we can import them as needed:

```html
<html>

  <head>
  <!—
  If we had created the bundle as a html file, we link to it here
  <template is=”chtml-bundle” src=”/path/to/bundle.html”></template>
  -->
  </head>

  <body>
    <!—Place an image here -->
  <chtml-import chtml-ns=”img/brand/logo2”></chtml-import>
  </body>

</html>
```

### Imports Ondemand
By default, imports are resolved as they land in the DOM. This makes sense most of the times. At other times, we may want imports to resolve on-demand – just at the time they are accessed in an application. This is achieved with the ondemand Boolean attribute.

When implemented as a node in a component, for example, imports of this type are provisional and get resolved on first access to the node.

```html
<div chtml-role=”article”>
  <chtml-import ondemand chtml-role=”article-author” chtml-ns=” html/badge/user”></chtml-import>
</div>
```

### Shadow Imports
It is possible to import components directly into an element’s shadow DOM. This form of import is called shadow import. Shadow imports are qualified with the shadow Boolean attribute. An import’s shadow host is its immediate parent.

The import element itself gets replaced and is never part of the Shadow DOM nor the light DOM. Imported components are never visible anywhere in the DOM but live hidden in the host element’s shadow DOM.

```html
<div id=”host”>
  <chtml-import shadow chtml-ns=” html/badge/user”></chtml-import>
</div>
```

We could even send some CSS into the shadow DOM for the component.

```html
<div id=”host”>
  <chtml-import shadow chtml-ns=” html/badge/user”></chtml-import>
  <chtml-import shadow chtml-ns=”css/badge”></chtml-import>
</div>
```

## Recomposition
Import elements can do more than just place a component. They can be empowered to recompose the component being imported. We do this by predefining contents on the import element with a view to having these on the component they import. The import element is replaced as usual, but this time, with a richly composed component.

For example, if we wanted an incoming component to have an ID, we would set this on the import element.

```html
<chtml-import id=”some-id” chtml-ns=”html/badge/user”></chtml-import>
```

We could import the same component in another place with a different ID.

```html
<chtml-import id=”some-other-id” chtml-ns=”html/badge/user”></chtml-import>
```

Recomposition helps us fine-tune imported components to fit perfectly into every new use-case. The component below, for example, could be recomposed on every import we make of it.

```html
<template is=”chtml-bundle”>

  <div chtml-role=”user” chtml-ns=”html/badge/user“ style=”color:black”>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>

</template>
```

### Attributes and Params Recomposition
Certain attributes can be predefined on an import element with a view to having them on the imported component. For some types of attribute, values are replaced on the component; for others, values are merged.

**Classes and Inline CSS Are Merged.** Classes predefined on an import element are merged with any existing class-list on the component. Also, any inline CSS styles are appended to the component’s style attribute, and CSS cascading takes effect, with styles from the import element taking priority.

```html
<div chtml-role=”article”>
  <chtml-import class=”class1 class2” style=”color:blue” chtml-ns=”components/badge/user”></chtml-import>
</div>
```

The final composition gives us:

```html
<div chtml-role=”article”>
  <div chtml-role=”user” class=”class1 class2” style=”color:black; color:blue” chtml-ns=”components/badge/user“>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>
</div>
```

**Roles Are Merged.** Any ARIA roles (in the role attribute) and CHTML roles (in the chtml-role attribute) predefined on an import element are merged with existing component roles. This adapts the imported component for additional roles to play.

Below, we’re importing the user component to be an article’s author node. Notice that this works because roles are merged. (We’ve done this a number of times before now, but the details might not have been obvious.)

```html
<div chtml-role=”article”>
  <chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user”></chtml-import>
</div>
```

The final composition gives us:

```html
<div chtml-role=”article”>
  <div chtml-role=”article-author user” chtml-ns=”html/badge/user“ style=”color:black”>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>
</div>
```

**Cascaded Params Are Merged.** Both inline params (the chtml-related and chtml-directives attributes) and the Cascaded Params Sheet (JSEN-P) that may be predefined on an import element are composed into the imported component. So while a component would normally be defined with parameters for general use-case, we can compose new parameters into each import we make of it.

For inline parameters, parameters on the import element are appended to the component’s existing parameters or created on the component for the first time. When appended, cascading takes effect naturally and newer parameters get to override existing identical parameters, as governed by JSEN-P cascading rules.

This is what happens below.

```html
<template is=”chtml-bundle”>

  <div chtml-role=”user” chtml-ns=”html/badge/user“ chtml-directives=”name.el.append:data.first_name”>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>

</template>
```

```html
<div chtml-role=”article”>
  <chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user” chtml-related=”extra_node:_.some_node” chtml-directives=”name.el.append:data.first_name.toUpperCase() !important”></chtml-import>
</div>
```

The final composition would give us:

```html
<div chtml-role=”article”>
  <div chtml-role=”user” chtml-ns=”html/badge/user“ chtml-related=”extra_node:_.some_node” chtml-directives=”name.el.append:data.first_name; name.el.append:data.first_name.toUpperCase() !important”>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>
</div>
```

For **sheet-based parameters**, an import element’s JSEN-P is either created on the component for the first time or its parameters are appended to the component’s existing JSEN-P. When appended, cascading takes effect naturally and newer parameters get to override existing identical parameters, as governed by JSEN-P cascading rules.

This is what happens below.

```html
<template is=”chtml-bundle”>

  <div chtml-role=”user” chtml-ns=”html/badge/user”>
    <script type=”text/jsen-p”>
      @directives {
          name.el.append:data.first_name;
      }
    </script>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>

</template>
```

```html
<div chtml-role=”article”>
  <chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user”>
    <script type=”text/jsen-p”>
      @related {
          extra_node:_.some_node;
      }
      @directives {
          name.el.append:data.first_name.toUpperCase() !important;
      }
    </script>
  </chtml-import>
</div>
```

The final composition would give us:

```html
<div chtml-role=”article”>
  <div chtml-role=”user” chtml-ns=”html/badge/user“>
    <script type=”text/jsen-p”>
      @related {
          extra_node:_.some_node;
      }
      @directives {
          name.el.append:data.first_name;
          name.el.append:data.first_name.toUpperCase() !important;
      }
    </script>
    <div chtml-role=”user-avatar”></div>
    <div chtml-role=”user-name”></div>
  </div>
</div>
```

**Other Attributes Are Set If Not Already Exists.** Other attributes found on an import element (except the chtml-ns attribute) are re-created on the imported component if not found on the element.

#### Managing Recomposition
As a general rule, everything an import element has gets stripped and composed into the component it imports. But this recomposition can be controlled. There are two ways.

**Using the Norecompose Attribute.** To object to recomposition on an element, the norecompose attribute can be used. If without a value, this attribute disables recomposition completely. Setting its value to * achieves the same thing. A list of attribute names may, however, be set to specify the attributes that should be ignored during composition.

```html
<template is=”chtml-bundle”>

  <div chtml-role=”user” chtml-ns=”html/badge/user” chtml-directives=”el.prepend:data.names” style=”color:blue” norecompose=”chtml-directives @JSEN-P style”></div>
</template>
```

To ignore sheet-based parameters recomposition, the @JSEN-P keyword could be added to the norecompose list.

The norecomposition directive may also be set generally for all modules in a bundle.

```html
<template is=”chtml-bundle” norecompose=”chtml-directives @JSEN-P style”>

  <div chtml-role=”user” chtml-ns=”component/badge/user” chtml-directives=”el.prepend:data.names” style=”color:blue”></div>

</template>
```

With a bundle-level norecompose directive in place, modules will now be imported without having the listed attributes recomposed. A module-level norecompose directive could still be set to list additional attributes specific to the module.

**Using a Callback.** As we will see in the section for CHTML API, the entire recomposition process can be intercepted, or hooked-into, using a callback. This offers the ability to handle the composition of other types of attributes that would have been simply transferred by default.

### Node-To-Node Recomposition
Nodes in a component can be replaced on each import we make of it. We do this by creating the replacement nodes on the import element ahead of the incoming component. On arrival, the imported component gets its matched nodes replaced by these replacement nodes.

This is what happens below. We import the user component into the article component, but with a replacement node (user-name) that features special styling.

```html
<div chtml-role=”article”>
  <div>
	<chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user”>
	  <div chtml-role=”user-name” style=”text-transform:uppercase; font-weight:bold”></div>
	</chtml-import>
	<div>
	  <div chtml-role=”article-description”></div>
	</div>
  </div>
</div>
```

The final composition would give us:

```html
<div chtml-role=”article”>
  <div>
	<div chtml-role=”article-author user” chtml-ns=”html/badge/user“ style=”color:black”>
	  <div chtml-role=”user-avatar”></div>
	  <div chtml-role=”user-name” style=”text-transform:uppercase; font-weight:bold”></div>
	</div>
	<div>
	  <div chtml-role=”article-description”></div>
	</div>
  </div>
</div>
```

Attributes and params recomposition also happens on each node replacement. Before they are finally replaced, attributes and parameters from component nodes are composed into replacement nodes. So replacement nodes inherit properties of the component’s original nodes.

Now a few notes apply:
•	Replacement nodes must be at the root of the import element if they must be discovered.
•	Replacement nodes must define a role that corresponds to an original node in source component. But imports could also have root-level elements without a role. These get copied to the root of the imported component.

#### Recursive Recomposition
When we import a component and replace its node, we perform one level of import. But we can use replacement nodes to perform another level of import – a replacement node also being an import on its own.

For example, as we import an article component into a page, we could predefine a replacement node for its author node, but this time, the replacement node will be an import on its own, referencing a special type of user component.

This is what happens below. In the second level of import, we’re importing a new type of user component as the article’s author node. This user component splits a user’s name as first name, and last name (fname and lname).

```html
<html>
  <head>…</head>
  <body>
    <chtml-import chtml-role=”body-main” chtml-ns=”html/content/article/readonly”>
      <chtml-import chtml-role=”article-author” chtml-ns=”html/badge/user2”></chtml-import>
    </chtml-import>
  </body>
</html>
The final composition would give us:
<html>
  <head>…</head>
  <body>
    <div chtml-role=”body-main article” chtml-ns=”html/content/article/readonly”>
      <div chtml-role=”article-author user” chtml-ns=”html/badge/user2“>
        <div chtml-role=”user-avatar”></div>
        <div chtml-role=”user-fname”></div>
        <div chtml-role=”user-lname”></div>
      </div>
    </div>
  </body>
</html>
```

We could go even deeper to a third level, and a fourth, and as far as composition requires!

## Inheritance
In CHTML, any piece of code is considered a piece of knowledge. Inheritance provides extra opportunity to avoid costly duplication of knowledge. It’s a type of code reuse that is inherent – by virtue of where components live; it requires no extra steps to work.

### Namespace-Based Inheritance
The namespace-based component layout in CHTML is no mere naming convention or code organization. It brings with itself the wonderful concept of inheritance. 

In CHTML, components in a namespace are believed to be built off their supernamespace. So, the component at html/content/article/readonly/dark-mode is believed to be built off the component at html/content/article/readonly, which in turn, is believed to be built off the root component at html/content/article. Now instead of having to repeat common semantics down the namespace, CHTML makes components inherit them!

**Components Are Implicitly Composed.** Semantics from the component at a supernamespace are, by default, composed into components at each level of a namespace path.

By default, only attributes and params composition is performed, which means that nodes are not inherited; components retain their unique structural semantics.

Here is how this could look.

```html
<template is=”chtml-bundle”>

  <!—Standard, readonly article -->
  <div chtml-role=”article” chtml-ns=”html/content/article/readonly” chtml-directives=”title.el.append:data.title; content.el.append:data.content”>
    <div chtml-role=”article-title”></div>
    <div chtml-role=”article-content”></div>
  </div>

  <!—Dark-mode, readonly article -->
  <div chtml-ns=”html/content/article/readonly/dark-mode”>
    <div style=”color:white; background-color:black”>
      <div chtml-role=”article-title”></div>
      <div chtml-role=”article-content”></div>
    </div>
  </div>

</template>
```

Notice that in our dark-mode edition of an article component, it was not necessary to redefine the role article. The bindings were also inherited.

Inheritance takes effect at the time a component is being used.

Taking things a little further, a node-to-node recomposition is also possible, which means that structural semantics can be inherited. The chtml-import element comes into play here, this time as a component definition on its own. Here is how this looks.

```html
<template is=”chtml-bundle”>

  <!—Dark-mode, readonly article, with a different type of content node  -->
  <chtml-import chtml-ns=”html/content/article/readonly/dark-mode/framed”>
    <div chtml-role=”article-content” style=”border-color:white“></div>
  </chtml-import>

</template>
```

Now so much has been saved!

**Namespaces Can Be Used Ahead of Implementation.** By virtue of inheritance, we would be safe to find a component deep in a namespace that is yet to be implemented, as long as fallbacks exist up the namespace hierarchy.

Right below, we’re importing an animated type of article that we’re yet to create. Maybe soon; maybe someday! But we can, at the moment, make do with the closest implement that lives up the namespace hierarchy.

```html
<!—file: index.html -->
<html>
  <head>…</head>
  <body>
    <chtml-import chtml-role=”body-main” chtml-ns=”html/content/article/readonly/dark-mode/animated”></chtml-import>
  </body>
</html>
```

Now this really allows us to progressively build features into an app while using a real layout plan from the start.
 
### Cross-Bundle Inheritance
When multiple bundles are defined on a document, they are all used in a cascaded manner; in the same way multiple CSS stylesheets work. This helps us reuse code across bundles from multiple sources.

Now this is how it works: when a namespace is accessed for import, all bundles are queried for the requested component and matches are gathered and composed. Components from latter bundles thus inherit components from earlier bundles. Obviously, the order of these bundles matter.

In the example below, let us assume that the first bundle was linked from a third-party. The second bundle is a local bundle that inherits a component from the first bundle and applies dark-mode styling.

```html
<template is=”chtml-bundle”>

  <!—Standard, readonly article -->
  <div chtml-role=”article” chtml-ns=”html/content/article/readonly” chtml-directives=”title.el.append:data.title; content.el.append:data.content”>
    <div chtml-role=”article-title”></div>
    <div chtml-role=”article-content”></div>
  </div>
  
  <img src=”/assets/img/ui/banner.png” chtml-ns=”ui/banner” />

</template>
<template is=”chtml-bundle”>

  <!—Dark-mode, readonly article -->
  <chtml-import chtml-ns=”html/content/article/readonly/dark-mode” style=”color:white; background-color:black”></chtml-import>
  
  <img src=”/assets/img/brand/logo.png” chtml-ns=”img/brand/logo” />

</template>
```

The two bundled images above do not live in the same namespace. So they are imported independently of each other.

But how does this work with the namespace-based inheritance? The answer lies in a wonderful type of algorithm.

### The Inheritance Matrix
With namespace-based inheritance working from left to right and cross-bundle inheritance working top-down, a matrix is formed. On this matrix, inheritance begins from the left and moves level-by-level towards the right, but with top-down inheritance being applied at each level. This becomes clear in an example.

The two bundles below each have a pair of components. Each pair constitutes namespace-based, left-right inheritance. The two bundles themselves constitute cross-bundle, top-down inheritance. Now we have four components and three have a CSS color each. Let’s see what happens as we try to import them.

```html
<template is=”chtml-bundle”>

  <div style=”color:blue” chtml-ns=”html/base1/base2”></div>
  <div style=”color:red” chtml-ns=”html/base1/base2/red”></div>

</template>
<template is=”chtml-bundle”>

  <div style=”color:yellow” chtml-ns=”html/base1/base2”></div>
  <div chtml-ns=”html/base1/base2/red”></div>

</template>
```

If we tried importing html/base1/base2, all bundles will first be asked for html/base1 to see if there is anything for the following level to inherit. The first bundled is asked first, then the second. Since no component can be found, we come empty to html/base1/base2. Again the first bundle is asked first, and we find a blue component, then the second bundle, and we find a yellow component. They are composed to produce a yellow component, which is what we would expect.

```html
<div style=”color:blue; color:yellow” chtml-ns=”html/base1/base2”></div>
```

If we tried importing html/base1/base2/red, inheritance produces the same result above by the time we get to components/base1/base2. So we get into html/base1/base2/red with two colors as inheritance. Here again, the first bundle is asked first, and we find a red component, then the second bundle, and we find the final component but without a color. They are all composed to produce a red component, which is what we would expect.

```html
<div style=”color:blue; color:yellow; color:red” chtml-ns=”html/base1/base2/red”></div>
```

## Cascading
The level of reusability and composition in CHTML requires being able to manage (accept or reject) duplicate parameters. This is especially important as multiple bundles from different vendors may now be connected to the same document. Happily, the concept of cascading enables us to do so.

Cascading applies to inline parameters in the chtml-related and chtml-directives attributes and those in the Cascaded Params Sheet (JSEN-P). The way these parameters are processed in CHTML is governed by the following rules; they’re for the most part, like CSS cascading rules.

### JSEN-P Cascading Rules
**Unique and Duplicate Declarations.** Declarations of the same key and value are duplicates. Each declaration overrides a previous declaration.

The following directives, for example, are duplicates. The last one is what takes effect.

```html
<body chtml-role=”app” chtml-directives=”el.append:‘Thanks for visiting!’; el.append:(‘Thanks for visiting!’)”></body>
```

Declarations of the same key but with variations in value, are considered unique and are all honoured. This is what happens below.

```html
<body chtml-role=”app” chtml-directives=”el.append:‘Thanks ’; el.append:‘for visiting!’”></body>
```

Note that declarations for a component’s related nodes are later streamlined to one per node name. So, below, even though the declarations are considered unique according to JSEN-P cascading, only the last one is used. 

```html
<main chtml-role=”article” chtml-related=”comments:_.comments; comments:_._.comments”></main>
```

**The !important keyword.** Just as with CSS, the !important keyword can be used at the end of a declaration. It marks the declaration as of high priority and asks to replace other declarations of the same key, regardless of their value, no matter whether they live before or after it.

The first directive below takes precedence.

```html
<body chtml-role=”app” chtml-directives=”el.append:‘Thanks for visiting!’ !important; el.append:‘Hello World!’”></body>
```

Now where two declarations of the same key both have the !important keyword, they are treated as duplicates; the latter overrides the former. This is what happens below.

```html
<body chtml-role=”app” chtml-directives=”el.append:(‘Thanks ‘, ‘for visiting!’) !important; el.append:‘Hello World!’ !important”></body>
```

**The !fallback keyword.** For further control, the !fallback keyword can also be used at the end of a declaration. It marks the declaration as of lower priority and asks to be used ONLY where no declarations of the same key can be found anywhere before or after it. 

```html
<body chtml-role=”app” chtml-directives=”el.append:‘Thanks for visiting!’; el.append:‘Hello World!’ !fallback”></body>
```

This is especially useful where newer declarations do not want to take advantage of their position and want to relinquish their rights first to any previous declarations.

Now where two declarations of the same key both have the !fallback keyword, they a treated as duplicates; the latter overrides the former. This is what happens below.

```html
<body chtml-role=”app” chtml-directives=”el.append:(‘Thanks ‘, ‘for visiting!’) !fallback; el.append:‘Hello World!’ !fallback”></body>
```

#### Inline and Sheet-Based Declarations
Where declarations are made both inline and in a JSEN-P on the same component, the combined declarations are used, but inline declarations are considered newer for the JSEN-P Cascading Rules.

Below, inline wins for related nodes, JSEN-P wins for bindings.

```html
<main chtml-role=”article” chtml-related=”comments:_.comments;” chtml-directives=”el.append:(‘Thanks ‘, ‘for visiting!’)”>

  <script type=”text/jsen-p”>
	@related {
		comments:_._.comments;
	}
	@directives {
		el.append:‘Hello World!’ !important;
	}
  </script>

</main>
``` 
