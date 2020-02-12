# Reactivity and Bindings

Applications can be built with CHTML as View layer. This is even more seamless with the concept of reactivity and bindings.

## Reactivity

Reactivity is one beautiful thing about the component object. The Chtml class is built off the Observable class from `@onephrase/observable`. This brings us all the reactivity we'll need with components and everything else that's possible with Observables.

Being an Observable instance, we can observe when nodes land for the first time on the component object and when they exit the object - either by an explicit delete operation or via a direct removal from the DOM.

```javascript
// Remember that nodes are lazy-loaded.
// So the author node won't be loaded on the component until the first time it is accessed.
// First, let's construct an observer for this event
article.$observe('author', author => {
    console.log(author.el);
});

// Let�s access the node for the first time.
// Our observer should be called.
article.author.el.html('John Doe');

// Let's explicitly delete the node from the object.
// Our observer should be called and its author parameter should be undefined.
delete article.author;
// Re-accessing the node should recreate the node from the DOM and call our observer.
article.author.el.html('New Name');

// Let's directly remove the node from the DOM.
// Our observer should be called and its author parameter should be undefined.
article.author.el.remove();
// Re-accessing the node should now return undefined.
article.author.el.html('New Name'); // Reference error
```

### Binding

It is possible to introduce arbitrary properties into the component object. A component's properties must not all be nodes; application data can be set on a component on any property. But we will be careful with our choice of property name to avoid accidentally unsetting a node.

So we could actually do the following:

```javascript
// Trying to avoid conflicting with a node
article.authorNameVal = 'John Doe';
article.author.el.html(article.authorNameVal);
```

But we will be safer to use a property name that won't interfere with node names. The `$` character should be our best choice. \(The `$` character will now be reserved from being used as a node name.\)

```javascript
// Safe from conflicts
article.$ = 'John Doe';
article.author.el.html(article.$);
```

Suppose we had more than one data value to set on the component. This would normally mean setting multiple arbitrary properties on the component; but that would be polluting the component's property namespace. A better approach would be to make the `$` property an object.

```javascript
// Set multiple values out of the component's property namespace
article.$ = {};
article.$.author = 'John Doe';
article.$.description = 'Article description';
article.author.el.html(article.$.author);
article.description.el.html(article.$.description);
```

We can leverage reactivity and make the above operations dynamic. So we observe the `$` data property.

```javascript
// Render data dynamically
article.$observe('$', $ => {
    article.author.el.html($.author);
    article.description.el.html($.description);
});

// We could as well observe the data values as path
article.$observe('$.author', author => article.author.el.html(author));
article.$observe('$.description', description => article.author.el.html(description));

// Bring the data anytime
article.$ = {author:'John Doe', description:'Article description',};

// Update the data anytime
article.$ = {author:'Mark Spencer', description:'Updated article description',};
```

At this point, we have bound operations to the `$` data property. But we still have more to explore with bindings.

#### The Chtml.bind\(\) Instance Method

This method is just another way to set the component's data property. It accepts the data component to bind and sets it to the `$` property automatically.

```javascript
article.bind({author:'John Doe', description:'Article description',});
```

To change the default data property from the `$` character to something else, the params.dataqKey is used.

```javascript
// Set this globally
Chtml.params.dataKey = 'data';
// To change this per instance
var article = new Chtml(el, {dataKey:'data'});
```

Calls to `article.bind()` will now set the given data object to the component's `data` property. So it serves to hide implementation details and constitutes a more standard way to add data to a component.

### Binding Observables

If you noticed, binding plain data objects required that we replace the entire plain object to update. This is because updating properties of the plain object `$` would not bubble up to notify the article object of a change. But this is easy to fix by making the data object an Observable.

```javascript
// Bring in the Observable class
import Observable from '@onephrase/observable';

// Set the observable base for data.
// Initial properties for the Observable instance are optional
article.$ = new Observable({author:'John Doe'});

// Update properties.
// Operations bound to '$.author' will be re-executed.
article.$.author = 'Mark Spencer';

// Set new properties.
// Operations bound to '$.description' will be executed for the first time.
article.$.description = 'Article description';

// Nest Observables as needed to make deep properties reactive.
article.$.author = new Observable({fname:'John', lname:'Doe'});
// Update and anything bound to "$", "$.author", or "$.author.fname" will be called.
article.$.author.fname = 'Mark';
```

As a general good practice, application state, and all functionality over state, are not built on CHTML components, but off the component as standalone observable components that plug-in to the CHTML component.

### Controlling State

In the examples above, we have directly mutated the state of our data object. But a proper way to do this is to encapsulate the mutation logic within the Observable object itself and expose mutations as methods. This way, we will be sure states are mutated in a standard way across the application.

Below, we create a dedicated Author data component, with state control.

```javascript
class Author extends Observable {

    /**
      * Here we initialize the instance with author names
      */
    constructor() {
        super();
        this.authors = [{fname:'John', lname:'Doe'}, {fname:'Mark', lname:'Spencer'}];
    }

    /**
      * This method publishes a new author name on each call
      */
    next() {
        var nextAuthor = this.authors.shift();
        if (nextAuthor) {
            // The set method must be used to set state
            this.set('fname', nextAuthor.fname);
            this.set('lname', nextAuthor.lname);
            // Or in one set() operation
            //this.set(nextAuthor);
        }
    }
}
```

Next, the binding that updates the DOM. And the binding that updates the data component.

```javascript
// We let state-change in the data component trigger update on the DOM.
// We�ll be showing the full name
article.$observe(['$.author.fname', '$.author.lname'], authorNameArray => article.author.el.html(authorNameArray.join(' ')));

// We let event in the DOM trigger update on the data component.
// We�ll be seeing a new author on double-clicking the article element
article.el.on('dblclick', () => article.$.author.$next());
```

Now, we plug in the Author data component and make our first click.

```javascript
// Add an Author instance to the article component
article.$.author = new Author();
```

At this point, a control-flow pattern has emerged; let's call this Actions and Reactions! Here, state components are controlled via actions \(method calls\), DOM components are controlled via reactions \(state bindings\).

In the Actions / Reactions control-flow pattern, one party \(the State component\) is designed to be totally agnostic of the other party \(DOM components or other observers\). So whether or not DOM Components have been bound, the State component remains functional and independent. In other words, it does not need to know who is triggering its methods \(actions\) and who is listening to states \(observers, bindings\). Meanwhile, any part of the application can act on, and react to, a State component.

### Synchronizing Actions with Reactions

Although State components by design need not know about observers or bindings, they might still sometimes need a feedback from the observers bound to a state. This feedback comes very useful when the State component's next action needs to synchronize with these observers. Consider the case below.

In the `next()` method of our Author component above, we simply published authors in succession on the fname and lname states. But we could make things more interesting by announcing the beginning and end of this publishing event. We will capture the feedback from observers of this announcement to determine how we go about publishing the new author details. Let's call this project Author Display.

```javascript
/**
  * This method publishes a new author name on each call.
  *
  * It announces the start and end of each author-change with the 'headsup' state.
  */
  next() {
      var nextAuthor = this.authors.shift();
      if (nextAuthor) {
          // Announce the intention to publish new author details
          var announcementFeedback = this.set('headsup', 'publishing');
          // Did any observer ask to prevent this action?
          if (announcementFeedback.defaultPrevented) {
              return;
          }
          // Did any observer return a promise?
          // That would mean asking this publishing event to hold for a time.
          var returnedPromise = announcementFeedback.promises;
          if (returnedPromise) {
              returnedPromise.then(() => {
                  this.set('fname', nextAuthor.fname);
                  this.set('lname', nextAuthor.lname);
                  this.set('headsup', 'published');
                });
          } else {
              this.set('fname', nextAuthor.fname);
              this.set('lname', nextAuthor.lname);
              this.set('headsup', 'published');
          }
    }
}
```

As seen, the `next()` method has chosen to honor observer feedbacks. Now we can create bindings that really do return a Promise. In the binding below, we implement a fade-out /fade-in animation as a new author gets published. We do this by returning a Promise on hearing the `publishing` announcement, while playing the fade-out animation. The Promise is resolved at the end of the animation and the `next()` method notices this and publishes the details. Finally, the fade-in animation plays on hearing that the details have been `published`.

```javascript
// Bind the fading-out and fading-in to the 'headsup' state
article.$observe('$.headsup', state => {
    if (state === 'publishing') {
        return new Promise((resolve, reject) => {
            var animation = article.author.el.animate([{opacity:1, opacity:0}], {duration:600});
            animation.onfinish = resolve;
        });
    } else if (state === 'published') {
        article.author.el.animate([{opacity:0, opacity:1}], {duration:600});
    }
});

// As before, the binding that updates the DOM
article.$observe(['$.author.fname', '$.author.lname'], authorNameArray => article.author.el.html(authorNameArray.join(' ')));

// As before, the binding that updates state
article.el.on('dblclick', () => article.$.author.$next());

// Plug in the Author, and let double-clicking begin
article.$.author = new Author();
```

### Optimizing DOM updates

As seen, CHTML does not intercept operations that update the DOM. But it makes room for implementing DOM updates that are performant. A common technique is to batch DOM-mutation operations and execute them differently from DOM-read operations while keeping everything in sync with the browser's "animation frame" \(the `window.requestAnimationFrame()` function\). That way, we would be avoid unnecessary DOM thrashing.

This is an optional optimization strategy and is covered outside the scope of CHTML. But below would be a contrived implementation of the `el.html()` method we have been using.

```javascript
html(content) {
    // We wrap the actual operation
    // in a callback from an imaginary batch() function.
        batch(() => {
            this.innerHtml = content;
            // Or if this were a custom jQuery method, we would say
            //this.get(0).innerHtml = content;
    });
}
```

