# CommonJS assets specification

## Motivation

Reusing code is great. CommonJS (and in particular `npm`) is a great way to share reusable modules for the server-side. However, it's difficult to use for client-side apps for a few reasons:

1. CommonJS is not designed for expressing static assets that have to map to public URLs. Specificially, today it is impossible to include a static asset like an image in your CommonJS package and figure out what its public-facing URL will be.
2. CSS is not modular, so stylesheets in multiple packages may have conflicting rules.

Existing client-side package tools do not sufficiently address these two points. We are proposing a solution to these problems that builds on top of CommonJS, so that users of this spec can continue to use their existing tools.

## Initial assumptions

This spec assumes that your package will contain modules that conform to the CommonJS spec. It also assumes that your package will have a unique name; generally this will be the `name` field in `package.json`, but `package.json` and `npm` are not strictly required by this specification (though they are encouraged).

## 1. Delivering static assets

In this spec we introduce the idea that static assets are part of the `CommonJS` dependency graph. We assert that your JavaScript packaging system (i.e. `browserify`, `RequireJS` or `node-haste`) should be able to optimize your static assets with the knowledge of this graph.

### Reference implementation

A reference implementation based on `browserify` is available; see the [the staticify repo](http://github.com/petehunt/staticify).

### `require()`ing images

You can `require()` images the same way you `require()` JavaScript modules. It will use the native `require.resolve()` to locate the file. On many popular platforms (including Node and browserify) you must include the file extension (i.e. `var MyImage = require('./MyImage.jpg');`). When you `require()` an image it will return an object that uniquely represents the image and can be used to render it in your environment. In most environments this will be a protocol-relative absolute URL to the image file.

`staticify` returns a data-URI for simplicity (and performs quite well), but a more advanced packager may sprite the image and return an object containing the sprite image URL as well as the coordinates within the sprite.

### `require()`ing stylesheets

You can `require()` CSS the same way you `require()` JavaScript modules as discussed in the previous section. After this module has been evaluated, any DOM nodes with `class` attributes should get styled by the stylesheet that was required.

`staticify` appends a `<style>` element with inline CSS to the `<head>` of the document. A more advanced packager could link to a concatenated `.css` file or do style calc in JavaScript based on the `class` attribute and assign an inline style to each DOM node instead.

## 2. Modular CSS

CSS does not support any form of modularity, so we introduce a simple set of conventions for all CSS that a package may load onto the page.

- **Only class selectors may be used.** No tags, IDs or anything else. Tag selectors can easily conflict and IDs are not reusable.
- **All class names must be prefixed by the package's unique name and a dash.** If you're using `package.json` this unique name will come from its `name` field. This is to ensure that class names used in one package won't leak into another one.

## Future work

This proposal focuses on being easy-to-use and open-ended. There are a few future areas of improvement:

- **Private CSS classes.** The idea that some CSS classes may be overridden and others may not.
- **Opinionated rules on CSS specificity.** This will encourage reliable rule overrides if load order changes.
- **CSS refcounting.** UI elements can enter and leave the DOM. When all DOM nodes that reference a given CSS file leave the DOM, a CSS reaper should be able to prune unused CSS rules.
