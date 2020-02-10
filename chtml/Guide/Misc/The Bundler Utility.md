# The Bundler Utility
This is a little server-side utility that automatically bundles markup-based files into a single file that can be linked-to as an external CHTML bundle.

## Basic Usage

```js
import Bundler from ‘@onephrase/chtml/src/Bundler.js’;
const bundler = new Bundler(entry);
```

Above, `entry` is the base directory for finding files. Files are scanned recursively from here into subdirectories and are bundled each with a namespace that reflects their location.

To get the bundled contents as string, call the `output()` method.

```js
var bundleContent = bundler.output();
```

To save the contents to a file, provide the destination file to the `output()` method; this can be an absolute path or a relative path – relative to the bundle `entry` given initially.

```js
bundler.output(outputFile);
```

Let’s demonstrate this with the sample file structure below.

```html
/root
      |--- project-files
      |      |--- components
      |      |	|--- component1.html
      |      |--- page1.html
      |      |--- page2.html
      |--- public-folder
```

Now we bundle the files in the `project-files` directory into `public-folder`.

```js
const bundler = new Bundler(‘/root/project-files/’);
bundler.output(‘/root/public-folder/bundle.html’);
```

This file can now be loaded as a chtml-bundle.

```html
<template is=”chtml-bundle” src=”public-folder/bundle.html”></template>
```

On checking the bundle, you will notice that the namespace of each module is prefixed with the extension name of their original file. Here’s how that could look:

```html
<div chtml-ns=”html/components/component1”></div>
<div chtml-ns=”html/page1”></div>
<div chtml-ns=”html/page2”></div>
```

## Bundling From Multiple Entries
To create multiple bundles from multiple entries, use the static `Bundler.multiple()` method. This method accept a list of entries as an object and returns an object with each bundle name mapped to their respective bundle content. 

```js
var bundles = Bundler.multiple({
	images: ‘path/to/images/’,
	template: ‘path/to/templates/’,
});
console.log(bundles.images);
```

To save all bundles to a common directory, provide an output file path as second argument to `Bundler.multiple()`.

```js
Bundler.multiple({
	images: ‘path/to/images/’,
	templates: ‘path/to/templates/’,
}, ‘/path/to/public-folder/[name].bundle.html’);
```

Notice the `[name]` placeholder in the destination filename. For each of the bundles, this placeholder is replaced with the unique bundle name. So, while all bundles are saved to `/path/to/public-folder/`, each bundle is saved with a unique filename. (If a placeholder is not in the given file path, something bad happens – each bundle is saved to the same file and previous bundles are overwritten!)

We can now load each bundle.

```html
<template is=”chtml-bundle” src=”public-folder/images.bundle.html”></template>
<template is=”chtml-bundle” src=”public-folder/templates.bundle.html”></template>
```

## Bundling Assets
While HTML modules are created by reading the file’s contents, assets, like images, are handled differently. They are still allowed to remain on the server, but this time, copied to the output directory where the regular bundles are located. An element that points to this new location is automatically generated in the bundle. This is illustrated below.

This is the normal file structure. It now contains an image.

```html
/root
      |--- project-files
      |      |--- components
      |      |	|--- component1.html
      |      |--- images
      |      |	|--- image1.png
      |      |--- page1.html
      |      |--- page2.html
      |--- public-folder
```

Let’s bundle the files in the `project-files` directory.

```js
const bundler = new Bundler(‘/root/project-files/’);
bundler.output(‘/root/public-folder/bundle.html’);
```

Now the image at `/root/project-files/images/image1.png` will now be copied to `/root/public-folder/images/image1.png` and an `<img>` element pointing to this new location is added to the bundle.

This is the new file structure:

```html
/root
      |--- project-files
      |      |--- components
      |      |	|--- component1.html
      |      |--- images
      |      |	|--- image1.png
      |      |--- page1.html
      |      |--- page2.html
      |--- public-folder
              |--- images
              |	|--- image1.png
              |--- bundle.html
```

And our `bundle.html` should look like this:

```html
<div chtml-ns=”html/components/component1”></div>
<img chtml-ns=”png/images/image1” src=”images/image1.png” />
<div chtml-ns=”html/page1”></div>
<div chtml-ns=”html/page2”></div>
```

In another way, it is possible to bundle small images (or other media) in data-URL format. The Bundler just needs to know the at what file size to use the data-URL format. Set the static `Bundler.maxDataURLsize` to a size measured in bytes.

```js
Bundler.maxDataURLsize = 1024;
```

Media files lower than `1024` in size will now be bundled in data-URL format. This can greatly reduce the number of HTTP requests the browser has to make.
