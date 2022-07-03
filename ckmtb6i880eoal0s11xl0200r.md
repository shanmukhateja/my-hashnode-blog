## Setting incorrect peerDependencies can ruin your day as a library maintainer!

Let me tell you how setting incorrect peerDependencies inside package.json can ruin your day! *TL;DR at the bottom!*

## Background:

As a maintainer of angular-datatables, I've been trying to fix a bug with `ng add` schematics wherein the package would install incorrect, incompatible library versions on my Angular projects. 

I've tackled issues with ng add several times in the past:

1. [due to schematics-utitlities and rollup](https://github.com/l-lin/angular-datatables/issues/1441)
2. [The peerDependencies Menace](https://github.com/l-lin/angular-datatables/issues/1498)
3. [Revenge of the peerDependencies](https://github.com/l-lin/angular-datatables/commit/becb1f5a6996ec073b1c48ffadf1e0f2015af940)

## Theory:

When you install make a library, you can specify certain requirements as to how a specific version of your package is "the right choice" in the current working directory.  This can often be specified with `peerDependencies` inside `package.json`which means two things:

1. This library **requires** this package to work.
2. If this package isn't found and/or not compatible with node_modules, warn the user about it.
3. If this package does exist, do nothing.

Say for example, you would like to install angular-datatables  inside your Angular 10 project. So when you run `ng add angular-datatables`, it would fetch metadata of angular-datatables in npmjs.com and among other things, verify compatibility of your Angular project with each version of our library using `peerDependencies`.
So if the project had invalid or "too strict" peerDependencies, the match would fail and the search for the chosen one continues. 

This happened to me on "The peerDependencies Menace" due to "too strict" peerDependency on `core-js@^3.6.5` in all our major releases then.  Here's the specific [commit](https://github.com/l-lin/angular-datatables/commit/6b17ffb810263f029cbfa7017d4f0d4d5fc42ec4).
In hindsight, I should've removed "angular/platform-browser" and "angular/platform-browser-dynamic" but oh well!

## Implementation:

The commit worked and everything was good...for a while. But then it came back. One day, I was testing something on newly generated Angular 10.2.x project and tried installing the plugin with `ng add angular-datatables` but it seemed to have gone for v11.x instead of v10.x 

After some trial and error with `verdaccio` and `npm publish`, I was able to figure out, it was `angular/platform-browser` and `angular/platform-browser-dynamic` inside package.json of v10.1.1 (latest for 10.x then) which were removed from later releases.

The long-term solution which I've come up is to bind latest releases of angular-datatables library to their respective Angular major version. For example, in the case of v10.1.1, it's peerDependency now includes `angular/core@^10` which translates to "make sure this version is only installed on Angular >=10.x and <11.x projects only. 

**TL;DR:** Make sure your `peerDependencies` match your library's version. 