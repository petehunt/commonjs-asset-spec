# CommonJS components specification

## Motivation

Reusing code is great. `npm` is a great way to share reusable modules for the server-side. However, it's difficult to use for client-side apps for a few reasons:

1. `npm` is not designed for expressing static assets that have to map to public URLs. Components may be packaged and hosted a multitude of ways so a hardcoded URL convention is not an option.
2. CSS is not modular, so stylesheets in multiple packages may have conflicting rules.

## Initial assumptions

This spec assumes that your components will be delivered as CommonJS modules. It also assumes that your project will have a globally unique name -- generally this will be the `name` field in `package.json`, but `package.json` and `npm` are not strictly required by this specification (though they are encouraged).

## 1. Delivering static assets

In this spec we introduce the idea that static assets are part of the `CommonJS` dependency graph. We assert that your JavaScript packaging system (i.e. `browserify`, `RequireJS` or `node-haste`) should be able to optimize your static assets with the knowledge of this graph.

### Reference implementation

A reference implementation based on `browserify` is available; see the [the staticify repo](http://github.com/petehunt/staticify).

### `require()`ing images

You can `require()` images the same way you `require()` JavaScript modules except you must include the file extension (i.e. `var MyImage = require('./MyImage.jpg');`). When you `require()` an image an absolute protocol-relative URI to the image will be returned.

`staticify` returns a data-URI for simplicity (and performs quite well), but a more advanced packager may sprite the image and keep a sprite map for lookups.

Support for `.png` and `.jpg` is required (all of these plus `.gif` are included in staticify).

### `require()`ing stylesheets

You can `require()` CSS the same way you `require()` JavaScript modules except you must include the file extension (i.e. `require('./MyStylesheet.css')`). After this statement has executed any DOM nodes with `class` attributes should get styled by the stylesheet that was required.

`staticify` appends a `<style>` element with inline CSS to the `<head>` of the document. A more advanced packager could link to a concatenated `.css` file or do style calc in JavaScript based on the `class` attribute and assign an inline style to each DOM node instead.

Support for `.css` is required, but this could be extended to support `.less` and `.scss` as well (all of these are included in `staticify`).

## 2. Modular CSS

CSS does not support any form of modularity, so we introduce a simple set of conventions for all CSS that a component may load onto the page.

- **Only class selectors may be used.** No tags, IDs or anything else. Tag selectors can easily conflict and IDs are not reusable.
- **All class names must be prefixed by the package's globally unique name and a dash.** If you're using `package.json` this unique name will come from its `name` field. This is to ensure that class names used in one component won't leak into another one.

## Future work

This proposal focuses on being easy-to-use and open-ended. There are a few future areas of improvement:

- **Private CSS classes.** The idea that some CSS classes may be overridden and others may not.
- **Opinionated rules on CSS specificity.** This will encourage reliable rule overrides if the package load order changes.
- **CSS refcounting.** Components can enter and leave the DOM. When all components that reference a given CSS file leave the DOM, a CSS reaper should be able to prune unused CSS rules.