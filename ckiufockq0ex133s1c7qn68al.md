## We've just added Bulma's HTML table to ngx-bulma lib

**Background:**

I have contributed code to [ngx-bulma](https://github.com/ngx-builders/ngx-bulma/) project once again and this time we're adding native Angular support for Bulma's HTML table element. This post documents my journey.

The GitHub issue can be found [here](https://github.com/ngx-builders/ngx-bulma/issues/43) which was created a year ago and my PR for this can be found [here](https://github.com/ngx-builders/ngx-bulma/pull/488).

> We are currently exploring ways for providing documentation on advanced HTML table use cases like searching, filtering, pagination via a third-party plugin to the project.

The documentation for styling tables in Bulma is pretty straightforward. We add "table" class to `<table>` element and additional properties can be appended to it as CSS classes.

**Theory:**

The idea is to create a `BulmaTableModule` and use `BulmaTableDirective` as the Angular directive to drive its necessary features. In HTML context we use it via `buTable`. This directive uses `Input()` variables to enable/disable CSS attributes. 
For example, `is-bordered` CSS class can be enabled to table when `bordered` flag is enabled to table like:

```
<table class="table" buTable bordered="true">
...
</table>
```

## Implementation:

I started by duplicating an existing library module as `projects/ngx-bulma/table/table.module.ts`, renaming it to `BulmaTableModule` and updating references accordingly to match project structure.

> When you don't know how project directory structure works, just play it safe and go along with existing elements, you'll understand how it'll all click into place given some time. 

The final directory structure looked like [this](https://github.com/ngx-builders/ngx-bulma/tree/bdc47466a481ac6981dc73377c1fc1af34bce0c9/projects/ngx-bulma/table). We export `BulmaTableModule`, `BulmaTableDirective` classes in `projects/ngx-bulma/table/public-api.ts` to make them publicly available from `ngx-bulma`.

Also, the documentation for this is rendered from Markdown (.md) files at `projects/bulma-app/src/assets/docs`.

Finally, we update `CONTRIBUTING.md` and append `table` as allowed Git commit message components when committing code to project.

## Conclusion

It is always educational contributing to open-source projects. I learned a lot working on this feature and I'm grateful to [Santosh](https://twitter.com/SantoshYadavDev) for his guidance.

Feel free to @ me on [Twitter](https://twitter.com/shanmukhateja94) on your thoughts on this. Bye :)