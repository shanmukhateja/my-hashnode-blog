## Here's how Angular 10 broke my app in older browsers

After upgrading my Angular 9 web app to v10, I finally got to meet the infamous "Angular white screen of death". This new version broke the app completely. Nothing works. Read on to learn how I figured out the solution.

**Background:**

I keep a stock `ng new` Angular demo project for "SCIENCE!!". It gets introduced to every new feature I'd like to try on my projects - personal or work related. So, when Angular 10 rolled out, I upgraded it to Angular 10 with:

`ng update @angular/core @angular/cli`

It took a while but eventually the upgrade was successful. My initial tests on recent versions of Firefox were alright, nothing to see here. However, things got interesting when I fired up my "testing" browser, it's Firefox 45 kept around for testing projects used for production deployments.

So, the Angular 10 prod build failed - all I see is a white screen. I checked the Console for errors but nothing. It was completely empty, not even Angular's default message "*Angular is running in development mode...*". 

It turns out, Angular 10 and future versions transpile their code to `ES2015` which is not supported in my older browser. Hence, the index.html was being parsed however the JS engine was unable to execute the JS files bound by `<script>` tags.

**Solution:**

Go to `<project_dir>/tsconfig.json` and change `target` value from `es2015` to `ES5`.

Compile the code using:

`ng build --prod`

**Conclusion:**

I found this solution searching through older GitHub issues in `angular-cli` repo. This would've been avoided had I read the Angular blog about v10:


> The Angular Package Format no longer includes ESM5 or FESM5 bundles, saving you 119MB of download and install time when running yarn or npm install for Angular packages and libraries. These formats are no longer needed as any downleveling to support ES5 is done at the end of the build process.

This info can also be found if you check the deprecations section in Angular Guide  [here](https://angular.io/guide/deprecations#esm5-and-fesm5-code-formats-in-angular-npm-packages) 

Always remember to check for issues when upgrading to major release!