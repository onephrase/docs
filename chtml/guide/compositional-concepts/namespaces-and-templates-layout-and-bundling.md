# Namespaces and Templates, Layout and Bundling

In CHTML, components intended for later use are defined together in a `<template>` element. They may, however, be defined individually in separate files on the server with a view to bundling them into a `<template>` element � whatever makes them live comfortably and easy to maintain.

## Namespaces and Templates

Components defined together in a `<template>` element are each assigned a unique namespace. Namespaces organize them in virtual categories and help us reference them and reuse them.

Namespaces are like file paths. They are assigned on the chtml-ns attribute.

```markup
<template>
  <div chtml-role=�article� chtml-ns=�html/content/article�></div>
</template>
```

Here we�ve placed article under a category named content. There could be other types in this category. And we can have subcategories � to as much as organization requires.

```markup
<template>

  <!-- Displays article -->
  <div chtml-role=�article� chtml-ns=�html/content/article/readonly�></div>

  <!-- Allows editing -->
  <div chtml-role=�article� chtml-ns=�html/content/article/editable�></div>

</template>
```

Note that the way namespaces are used in CHTML demands that a namespace be of, at least, two parts. So a single word like `content` isn�t a valid namespace; something like `html/content` is.

Templates can be defined either as elements in the `<head>` section of a document or as external HTML files that can be linked to the document. For CHTML to automatically pick them up, the `chtml-bundle` type of `<template>` must be used.

```markup
<html>
  <head>

    <template is=�chtml-bundle�>
      <div chtml-ns=�html/content/article�></div>
    </template>

  </head>
  <body></body>
</html>
```

To link a bundle from the server, the `src` attribute is used. This attribute isn�t defined as part of the `<template>` element and is entirely a CHTML�s way of loading template content as the standard `<link type=�import�>` element no longer exists.

```markup
<!�file: bundle.html -->
<div chtml-ns=�html/content/article/readonly�></div>
<div chtml-ns=�html/content/article/editable�></div>

<!�app -->
<html>
  <head>
    <!�The src attribute demands that the <template> element be empty -->
    <template is=�chtml-bundle� src=�/bundle.html�></template>
  </head>
  <body></body>
</html>
```

Now multiple bundles can be used � whether defined internally or externally. They will all work together in a cascaded manner, as covered in a subsequent section for bundle cascading. This opens the door to building rich applications with bundles from multiple sources.

## Layout and Bundling

The folder-based layout approach takes things from virtual to actual namespacing. Here, components are defined in stand-alone HTML files placed in a hierarchy of folders.

Here is the equivalent layout of the namespaced article components above:

```markup
content
        |-- article
                |-- readonly.html
                |-- editable.html
```

Now, a component�s namespace naturally becomes the combination of its path and filename; and the `chtml-ns` attribute isn�t required anymore.

In a straightforward process called bundling, it is easy to put all our components back to a `<template>` file. This time, namespaces are automatically derived and assigned to each component. Check out the little server-side bundler that ships with CHTML.

