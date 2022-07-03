## Say hello to patch-package: An efficient approach to patching packages in node_modules

**Background:**

When working on NodeJS project, I came across a bug with a third-party dependency. A quick fix was applied by modifying the source files directly in my `node_modules` directory which got me thinking for efficient ways to accomplish this. 

**Problem:**

When making changes to feature in NodeJS project, I discovered a bug with path finding on production builds. The culprit was an npm library which has been marked as deprecated by upstream and since we're already running on latest version, there's no newer versions available. Looking for an alternative library and re-writing parts of code would be very time consuming so I resorted to making changes to code directly.

Looking around the source code in VS Code got me to work a quick fix in no-time. Now the challenge was to make this code git-efficient. I need to devise a way to distribute my changes (to library) to all developers either working on this project or simply using it.

**Solution:**

After googling for a while and few trial and errors, I found, what I believe to be, an elegant solution. It's called  [patch-package](https://npmjs.com/package/patch-package) and is described by the package author as:
 
> patch-package lets app authors instantly make and keep fixes to npm dependencies. It's a vital band-aid for those of us living on the bleeding edge.

In simpler terms, it looks for changes to a given package inside your `node_modules` with an upstream copy which it downloads to a temporary directory and writes these changes to a file inside `patches` directory in your project.

Then you add these changes to your git history with `git add` and your developers can `git pull` them just like any other commit. *pretty neat, right?*

**Get Started:**

- Install `patch-package` as a dev dependency:


```
npm i -D patch-package
``` 


- Add the following line to `scripts` section of your `package.json`:

```
postinstall: "patch-package"
``` 

This will run `patch-package` after every successful `npm install` to apply your changes to libs.
  
- Make your changes to the library inside `node_modules` and save the file(s).

- Run the command below:

```
patch-package <lib_name_here>
```

> The library name should match the name in your `package.json` so I highly recommend copying it.

- You should now see `patches` directory with your custom changes created inside your project directory.  Add all change to Git, commit and push it to your Git backend.

- Once the changes are pushed, ask your developers to run `git pull` followed by `npm install` to apply changes on their machines.

**Bonus Round:**

This package can also be used for sub-dependencies of your project's dependencies like `your_project/node_modules/dependency/node_modules/sub_dependency/`. 

You can read more about the package by clicking [here](https://npmjs.com/package/patch-package).

**Conclusion:**

I hope you learned something new today which can help you in your projects. Feel free to comment below on your thoughts on this.  You can also @ me on [Twitter](https://twitter.com/shanmukhateja94).