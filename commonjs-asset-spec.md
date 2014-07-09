# CommonJS assets specification

This specification addresses how CommonJS modules can express dependencies on non-JavaScript assets. It's basically a fork of the CommonJS modules spec v1.1

## Contract

### Module Context

* In a module, there is a free variable "requireStatic", that is a Function.
  * The "requireStatic" function accepts a *static asset identifier* that maps to a static asset.
  * "requireStatic" returns a JavaScript object that represents the static asset (called a *static asset descriptor*).
  * If there is a dependency cycle, the foreign module may not have finished executing at the time it is required by one of its transitive dependencies; in this case, the object returned by "requireStatic" must still be a valid static asset descriptor, but may not have all data available yet.
  * If the requested module cannot be returned, "requireStatic" must throw an error.

### Static Asset Identifiers

* Static asset identifiers are resolved with the exact same semantics as the CommonJS `require()` function.

### Static Asset Descriptors

The type of static asset descriptor returned depends on the version of the spec and the file extension of the asset after resolving the static asset identifier.

* `.png`, `.jpg`
  * Returns an absolute URL to the image, preferably protocol relative.
  * **Future work**
    * `.bounds.{left|right|top|bottom}` to support spriting
    * `.retina.{src|bounds}` to support `@2x` assets
* `.css`
  * Returns null, but has the side effect of inserting the stylesheet into DOM (on the client) or the HTTP response (on the server).
  * Can use `requireStatic()` inside of `url()` within the stylesheet to refer to a static asset identifier rather than a URL.
  * **Future work**
    * `retain()`/`release()` methods to "garbage collect" stylesheet
    * `.classNames` to support mangling/minifying/modular CSS class names

## Reference implementation

This is currently in production on `Instagram.com` via a set of [webpack](http://webpack.github.io/) plugins and configs. These will be open-sourced.
