# On the DLL
<p>From the webpack docs:</p>
<p style="text-align: center">The DllPlugin and DllReferencePlugin provide means to split bundles in a way that can drastically improve build time performance.</p>
---
### HYFN8 Front End App Design Goals
* Each channel's front end code should live in it's own repository.
* Teams should be able deploy code without recompiling the other teams code.
* We can't or shouldn't load multiple instances of react or other large libraries.
---
### The Problem
* How do we share code and dependencies between different react codebases all running as one big app?

Example: for this to work each bundle would need their own react at compile time. 

```html
<!-- index.html -->
<script src="shell.bundle.js" />
<script src="snap.bundle.js" />
<script src="twitter.bundle.js" />
<script src="pinterest.bundle.js" />
<script src="facebook.bundle.js" />
```
This leads to errors and huge bundles!
---
### The webpack Dll plugin can solve this problem.
---
1. Create a JS file that simply requires the packages that need to be shared.
```javascript
# sharedPackages.js
require('react')
require('react-dom')
require('redux') //... and many more!
```
---
2. Create a webpack config from where to generate the shared bundle. Notice the entry file is the file we just created.

```javascript
//webpack.dll.config.js
var path = require("path")
var webpack = require("webpack")

module.exports = {
  context: path.join(__dirname, "client/shell"),
  entry: {
    lib: ["./sharedPackages"]
  },
  output: {
    path: path.join(__dirname, "/dll"),
    filename: "[name].js",
    library: "[name]_[hash]",
    pathinfo: true
  },
  plugins: [
    new webpack.DllPlugin({
      path: path.join(__dirname, "/hyfn8-fe-apps-dll/[name]-manifest.json"),
      name: "[name]_[hash]",
      context: __dirname
    }),
  ]
};
```

---
4. Generate the Dll bundle and manifest.
running `webpack --config=webpack.dll.config.js`
yields:

```
dll/
* lib.js
* lib-manifest.json
```

---
3. Add the DllReference plugin to the webpack config of any build that you want to consume the bundle.

---
