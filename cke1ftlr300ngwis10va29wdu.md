## Here's how I refactored Navbar menu toggle logic in ngx-bulma.

## Background:

I stumbled upon [ngx-bulma](https://github.com/ngx-builders/ngx-bulma/) project on GitHub. While browsing Issues page, I found this [one](https://github.com/ngx-builders/ngx-bulma/issues/437) which piqued my interest. The issue talks about refactoring their Navbar component implementation such that end users wouldn't need to have write boilerplate code for toggling `navbar-menu`. 

I felt this was a low hanging fruit and I should pick it up. Take a look at my [commit](https://github.com/ngx-builders/ngx-bulma/commit/e7f9a5c9ca93bfde20a25115600389f989298714) for better understanding. 

My journey began by forking the GitHub repo and cloning it to my home directory.

`git clone https://github.com/shanmukhateja/ngx-bulma/`

After running `npm install` to let dependencies install, I opened the project directory in VS Code. 

I feel this part isn't explained well in `CONTRIBUTING.md` but to work on this project you need two terminal instances:

1. The `npm run build:lib:watch` command watches and compiles any changes made to lib.
2. The `npm run start` command starts `bulma-app` project which serves as documentation and also as testing ground for our changes in library. 

### Navbar component:

This component is located  at `projects/ngx-bulma/projects/ngx-bulma/src/lib/navbar/navbar.component.ts`. This component is a "holder" component as in the entire navbar resides as child nodes to it.

The issue takes place with `ng-content` tag inside the component which holds the `navbar-menu` and `navbar-burger` div elements which we need. 

So, we want to get a hold of the child elements of `ng-content`. I had to research a bit but in the end I learned that it is possible with [ContentChild](https://angular.io/api/core/ContentChild). 

From Angular docs:

> Use to get the first element or the directive matching the selector from the content DOM.

In simpler terms, it fetches the first child element that matches the given selector input (string). So, in our case, we can fetch child navbar items inside `ng-content` using `ContentChild`.

`@ContentChild(BulmaNavbarBrandComponent, { read: ElementRef }) cc: ElementRef; <-- #1`

The above code returns an `ElementRef` to `bu-nav-brand` component which contains the required `navbar-burger` as well as the `navbar-menu` items which we're interested in.

## Theory:

To make this work, we will need to get access to child nodes of `BulmaNavbarComponent` that have `navbar-burger` class to them. Next, we need access to `nav-menu` items themselves to show/hide when `navbar-burger` is clicked. These nodes are available within the same component so we inject `ElementRef` in the constructor and `querySelector` them within itself ( i.e. `BulmaNavbarComponent`). 

We can now either use `addEventListener` or go fancier with `fromEvent` from `rxjs` to implement the actual toggle logic. (I chose the fancier route!).

**Note:** I could have called `document.querySelector` and implemented the click functionality to all nodes in the entire page however that would mean my component would be manipulating nodes which are not it's children and it felt very wrong.

> We should call `removeEventListener` or call `unsubscribe()` from the nodes in `ngOnDestroy` when done to avoid memory leaks.

## Implementation:

Now that we have a working theory, I went ahead with writing the code in `projects/ngx-bulma/src/lib/navbar/navbar.component.ts` by creating two subscriptions which are initialized at `ngAfterViewInit` to handle toggle event and unsubscribed at `ngOnDestroy` when component is destroyed.

Additionally, I added a flag `useBuiltInToggler` for `BulmaNavbarComponent` to provide users to implement toggle events by themselves if they desire.

## Conclusion:

It took me a month to get the functionality working. I was battling my imposter syndrome during first 2 weeks, assuring myself the code I wrote is up to standards and  would be completely alright for me to send the PR. I finally pushed the code to my fork and asked Santhosh for a quick review for confirmation.

I want to say although it wasn't my first open source contribution, this PR always felt like one! It taught me great deal about `read` option in `ViewChild` and `ContentChild` APIs as well as satisfaction that my code will help my fellow developers. 
In this context, I highly encourage contributing to an open source project, it'll expand your understandings of the framework that you use. Cheers!