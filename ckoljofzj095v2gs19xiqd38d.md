## How to obfuscate production builds of your Angular app

Let's look at how to obfuscate production builds of your Angular web app.

**EDIT:** reword article title to avoid confusion :)

**EDIT2:** fixed webpack config.

> You will need an Angular project v10 or later to continue.

We will use [javascript-obfuscator](https://github.com/javascript-obfuscator/javascript-obfuscator) for this tutorial.

**Installation:**

-  Install custom Angular builder which supports Webpack config and obfuscator packages.

```
npm i -D @angular-builders/custom-webpack javascript-obfuscator webpack-obfuscator
```

- Now, update let's update `angular.json` to use the above installed Angular builder.

**BEFORE:**

```json
"architect": {
  "build": {
     "builder": "@angular-devkit/build-angular:browser",
        "options": {
           ...
        }
     }
}
```

**AFTER:**

```json
"architect": {
  "build": {
        "builder": "@angular-builders/custom-webpack:browser",
        "options": {
           ...,
            "customWebpackConfig": {
              "path": "./webpack.config.js",
              "mergeRules": {
                "externals": "replace"
              }
        }
     }
}
```

- Later, create a `webpack.config.js` at the root of the project directory (beside angular.json).

```javascript
// webpack.config.js
const JavaScriptObfuscator = require('webpack-obfuscator');
module.exports = (config, options) => {
  if (config.mode === 'production') {
    config.plugins.push(new JavaScriptObfuscator({
      rotateStringArray: true, // please customizable with options
    }, ['exclude_bundle.js']);
  }
}
```
More options can be found [here](https://github.com/javascript-obfuscator/javascript-obfuscator#javascript-obfuscator-options).

- Finally, run a production build with `npm run build` and compare the output JS files with older ones.

Hope you learned something valuable here. Feel free to  @ me on Twitter [here](https://twitter.com/shanmukhateja94)