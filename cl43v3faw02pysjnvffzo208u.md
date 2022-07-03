## MyStudio IDE: New features & future plans

Hello,

It's been a while since I found time and energy to work on my side project, [MyStudio IDE](https://github.com/shanmukhateja/mystudio-ide/), written in Rust using GTK.

After a few weeks of hiatus, I started hacking on some things on and off and I finally have something worth sharing with you all.

**Features:**

There's been quite a few changes in the project compared to previous release [v0.1.2](https://github.com/shanmukhateja/mystudio-ide/releases/tag/v0.1.2)

Notable ones include:

- Replace `SourceView` widget with `placeholder` in `main_window.ui` and create multiple instances on-the-fly using code.
<br>
There are now ready-to-use functions in-place to manage this.

- Convert `notebook.rs` to a Rust module and split it's functionality into several Rust files.
<br>
This change brings me great joy as the code is more maintainable to my eyes.

- All new PRs must be raised against `master`. I felt using `development` branch would become a burden. 

- Update application layout so `GtkNotebook` tabs aren't hidden when scrolling vertically.
<br>
This update broke the File Save feature in the project. I will talk more about this in few moments.

- Move `GtkNotebook` tab caching APIs as part of `NotebookTabCache` struct instead of individual functions inside `cache` module.
<br>
This change is a conflict for me as it manipulates a static `Vec<NotebookTabCache>` which is _technically_ not part of the cache struct.  

**Feature broken - Save Changes:**

This feature works by fetching contents of `TextBuffer` inside `SourceView`, converting them to Rust [string slice](https://doc.rust-lang.org/std/primitive.str.html) and writing it to disk.

> Note that, `tab` and `page` are interchangeable in this context. `GtkNotebook` calls them pages; I refer to them as tabs, even in code, for my understanding.

We first need to gain access to `TextBuffer` of the `SourceView` widget using `GtkNotebook` by finding tab (or page) index.  

The application has an internal cache to keep track of open tabs for quick use. It is also used to switch to existing tab when clicking on file in Workspace Explorer, if it is already open. 

Here's how this cache looks like:

![cache_struct_code.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654322841864/pGu02JoYz.png align="center")

> `pub` is an access specifier in Rust which is equivalent to `public` in other languages.

> `struct` is how we hold groups of values. [Learn more](https://doc.rust-lang.org/book/ch05-01-defining-structs.html)

__Previously on MyStudio IDE:__

We use `file_path` from cache and compare it to currently open file ([ref](https://github.com/shanmukhateja/mystudio-ide/blob/master/src/workspace.rs#L48)) to fetch `page number` (or tab index) from `GtkNotebook`. This would give us `SourceView` widget of currently open file which can be used to gain access to it's `TextBuffer`.

The layout looked as follows:

![prev_notebook_layout2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654325892521/p1fK8EXle.png align="center")

__Present:__

However, my fix for vertical scrolling issue has changed the layout slightly.  I fixed it by placing a `GtkScrolledWindow` (which is the widget used to add scrolling to a widget) as a "main child" widget to `GtkNotebook`.

This widget would have a single child, `GtkViewport` (think of it as a container of other widgets). 

Finally, I add my dynamically initalized `SourceView` widget as a child to `GtkViewport`.

The new layout  looks as follows:

![new_layout.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654326211143/k7lbnp-XQ.png align="center")

> This layout has to be replicated, in code, for every file opened inside the editor.

The problem was, once we figure out the page number, we get a `ScrolledWindow` widget however what we're looking for is its "grandchild" - `SourceView` widget. 

There was a `upcast` of `GtkWidget` to `SourceView` in the code which I didn't update when the layout fix was made and this ultimately broke the Save feature.

The solution was to access the first child of `ScrolledWindow` and the first child of `Viewport` to retrieve `SourceView` widget and return it for further processing.

You can find the Git commit for this [here](https://github.com/shanmukhateja/mystudio-ide/commit/41d7a0aa6537ecf1e1b1abc9e72ad5f9a3df9cda).

Here's my layout fix commit that caused this in the first place. [link](https://github.com/shanmukhateja/mystudio-ide/commit/d345eced123b83983d93263ecc272d550ad51913) 

> It was an easy fix however I didn't realize it until I accidentally came across it while working on the project.

**Future Plans:**

The Save feature bug made me realize I need to improve how I develop this project. Here's an list of changes I'd like to bring to the project.

- Convert the `mystudio-ide` project into a monorepo using [Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html). [PR link](https://github.com/shanmukhateja/mystudio-ide/pull/5/)

- Create a `libmystudio` Rust library inside the monorepo to hold all "non-UI functionality". [DONE]

- Update `mystudio-ide` repo to use `libmystudio` and remove all non-essential code. [DONE]

- Write unit tests for both the projects.

I'm not sure when/how/if I can find the time for these changes but I know these big steps will ensure what I've written will work as expected in the future.

Thanks for reading! Leave a comment on your thoughts and ideas for this.

You can @ me on [Twitter](https://twitter.com/shanmukhateja94)

Cheers!

**Update:** updated Future plans progress
