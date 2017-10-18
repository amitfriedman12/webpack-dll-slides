# On the DLL
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
<script src="https://somewhereonamazon/snap.bundle.js" />
<script src="https://somewhereonamazon/twitter.bundle.js" />
<script src="https://somewhereonamazon/pinterest.bundle.js" />
<script src="https://somewhereonamazon/facebook.bundle.js" />
<script src="shell.bundle.js" />
```
This leads to errors and huge bundles!
---
### The webpack Dll plugin can solve this problem.
---
### Create a JS file that simply requires the packages that need to be shared.
```javascript
# sharedPackages.js
require('react')
require('react-dom')
require('redux') //... and many more!
```
---
### Create a webpack config for the dll bundle.

```javascript
//webpack.dll.config.js
// ...
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
}
```

---
### Generate the Dll bundle and manifest.
running 
```
webpack --config=webpack.dll.config.js
```
yields 2 files:

```
dll/
* lib.js
* lib-manifest.json
```

---
### Add the Dll reference plugin to the other builds.
```
new webpack.DllReferencePlugin({
  'dll/lib-manifest.json',
  context: __dirname
})
```
Now when we bundle our individual apps they will use the manifest to delegate specified dependencies to the lib.js bundle.
---
### Now the entry file
```html
#index.html
<script>
  lazyLoader("lib.js", () => {
    lazyLoader("https://somewhereonamazon/snapchat.bundle.js")
    lazyLoader("https://somewhereonamazon/pinterest.bundle.js")
    lazyLoader("https://somewhereonamazon/twitter.bundle.js")
    lazyLoader("https://somewhereonamazon/facebook.bundle.js")
    lazyLoader("shell.bundle.js")
  })
</script>
```

---
### But... 
<p>every app needs to be compiled against the same manifest file. How do we distribute them and keep them in sync?</p>
---
### With the help of git submodules
```
git submodule update --init
```
---
### Sweet! now we can load dependencies only once and share them across apps.
<p>We also get faster build times because we don't have to rebuild our dependencies everytime we update our application code.</p>

