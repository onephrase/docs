# List Composition
It is intuitive to make a list in CHTML! Lists can be composed either programmatically or declaratively.

## Using the Chtml.populate() Instance Method
This method is used to populate a list component programmatically. It takes a data component to populate and a namespace for importing children into the list component. Now for each item in the data component, a child element is imported and appended to the list component.

Here are the list and list-item elements.

```html
<html>
  <head>
  <template is=”chtml-bundle”>

      <ul chtml-ns=”html/list “></ul>
      <li chtml-ns=”html/listitem“></li>

  </template>
  </head>

  <body>

    <!—import the list here -->
    <chtml-import id=”list-component” chtml-ns=”html/list”></chtml-import>

  </body>
</html>  
```

The import element above should be automatically resolved. So the body should have a `<ul>` element by now. Now, we’ll populate this element with data. An import will be made for each data item.

```js
Chtml.from(‘#list-component’).populate([‘item-1’, ‘item-2’], ‘html/listitem’);
```

This should produce the following:

```html
<ul id=”list-component” chtml-ns=”html/list”>
  <li chtml-ns=”html/listitem”></li>
  <li chtml-ns=”html/listitem”></li>
</ul>
```

An object would as well produce the same result:

```js
Chtml.from(‘#list-component’).populate({key1:‘item-1’, key2:‘item-2’}, ‘html/listitem’);
```

To manually set data on list items as they are loaded, provide a third argument as a callback function. This function will receive the just-imported list item and the data item.

```js
Chtml.from(‘#list-component’).populate([‘item-1’, ‘item-2’], ‘html/listitem’, (listItem, dataItem, itemKey) => {
	listItem.innerHtml = dataItem;
});
```

Now we should have something like the following:

```html
<ul id=”list-component” chtml-ns=”html/list”>
  <li chtml-ns=”html/listitem”>item-1</li>
  <li chtml-ns=”html/listitem”>item-2</li>
</ul>
```

To achieve this declaratively, we could simply declare the bindings on the list-item elements themselves or on the import element that imports them.

```html
<template is=”chtml-bundle”>

  <li chtml-ns=”html/listitem“ chtml-directives=”el.html:$”></li>

</template>
```

Now, as a list item lands in the list component, its directives are automatically executed. It would like the implementation below:

```js
Chtml.from(‘#list-component’).populate([‘item-1’, ‘item-2’], ‘html/listitem’, (listItem, dataItem) => {
	Chtml.from(listItem).bind(dataItem);
});
```

## Using an Extended Namespace
The list above can be all declaratively-rendered! CHTML supports using a two-part namespace to import a list component. The first part of the namespace serves as the actual import namespace, while the second is stripped off for subsequently importing children. Two forward slashes separate these namespaces.

```html
<body>

  <!—import the list here -->
  <chtml-import id=”list-component” chtml-ns=”html/list//html/listitem”></chtml-import>

</body>
```

Now we can simply set the data to the list component and have things automatically populated; every component loaded with a two-part namespace is automatically handled this way.

```js
var list = Chtml.from(‘#list-component’).proxy();
list.$bind([‘item-1’, ‘item-2’]);
// Or put transparently…
//list.$ = [‘item-1’, ‘item-2’];
```

## Updating a List
List population is implement in CHTML as a binding contract between the DOM and data components. So more list items are imported as needed to match the number of data items received.

Below, we render two extra items to our list. Since two list-item elements already exists in the list component, they will simply be re-rendered. Only two new imports will now be made.

```js
list.$ = [‘item-1’, ‘item-2’, ‘item-3’, ‘item-4’];
```

But what happens if we reduced the list data to a single item after having rendered many? The number of list-item elements will also be reduced! And setting an empty array will as well empty the list component. Remember, this is a binding contract!

```js
list.$ = [‘item-1’,];
```

Now, our component should be:

```html
<ul id=”list-component” chtml-ns=”html/list”>
  <li chtml-ns=”html/listitem”>item-1</li>
</ul>
```

Really, instead of replacing the entire list and making the component render from the beginning, we can selectively add items. This time, the data component will need to be an Observable from the start.

```js
var listData = new Observable([‘item-1’, ‘item-2’,]);
list.$ = listData;

listData[2] = ‘item-3’;

// We can directly call the observed array’s prototype methods…
// Add a fourth item
listData.push(‘item-4’);
// Empty the array
listData.splice(0);
// Add a new first item
listData.push(‘new item-1’);
```

And list rendering is index-aware! Items can be added to the data component on any key at any time, yet imported list-item elements will be correctly placed on the right index.

```js
listData[10] = ‘item-10’;
listData[8] = ‘item-8’;
listData.unshift(‘newest item-1’);
```

The last item “newest item-1“ should still come first, and “item-8” should still be placed before “item-10”:

```html
<ul id=”list-component” chtml-ns=”html/list”>
  <li chtml-ns=”html/listitem”>newest item-1</li>
  <li chtml-ns=”html/listitem”>new item-1</li>
  <li chtml-ns=”html/listitem”>item-8</li>
  <li chtml-ns=”html/listitem”>item-10</li>
</ul>
```

## Implementing Item Namespaces
When populating a list, items are retrieved, as seen, using the sub-namespace given. But individual items are retrieved with a namespace that is uniquely derived for the item. CHTML appends the item’s key to the base sub-namespace to import an item. So in the population we made above, the first item was imported using the namespace `html/listitem/0`, and the second item was imported using the namespace `html/listitem/1`, and so on.

The same pattern holds true for object-type list data. So for an object like `{key1:‘item-1’, key2:‘item-2’}`, the first item will be imported as `html/listitem/key1`, and the second, `html/listitem/key2`.

These derived namespaces can optionally be implemented where items really need to be unique. The base namespace, `html/listitem` in our case, remains the source of every item imported. 
