# MyStudio IDE: January 2023 Updates

Hello

It's been a while since I've blogged about the IDE. A lot of exciting changes have been introduced to the codebase since my [last post](https://surya-dev-journey.hashnode.dev/mystudio-ide-lets-talk-about-utf-16) in July!

## Highlights🎉

1. UTF-16 support
    
2. Lazy-load directories in Workspace Explorer.
    
3. Show hidden files (dotfiles) in Workspace Explorer.
    
4. Find in Files feature
    
5. Config file support
    
6. Added .dockerignore to improve build times during development
    
7. Find in Files - fixed the catch and minor improvements.
    
8. Refactor IDE internals to migrate to `struct` wherever possible.
    

Let's see in detail what each of these highlights have to offer.

### UTF-16 support

I have blogged about this a few months back [here](https://surya-dev-journey.hashnode.dev/mystudio-ide-windows-utf-16-encoding-software) and again [here](https://surya-dev-journey.hashnode.dev/mystudio-ide-lets-talk-about-utf-16). Please take a look at these posts for the full journey.

### Lazy-load directories in Workspace Explorer

During the initial stages of the IDE, I went with a "recursively read all directories" approach when loading a Workspace (project directory) into the IDE. This was to ensure I don't succumb to scope creep and as a "FIXME" for the future.

The problem was I was unable to open any project which had a huge number of sub-directories. This meant I cannot open JavaScript projects (looking at you - *node\_modules)* or even the MyStudio IDE project (hello there - *target)*.

A few months pass and I finally found the inspiration to fix this. I fixed this by setting the `max_depth(1)` inside `read_dir_recursive` function and listening on `row-expanded` [event](https://docs.gtk.org/gtk3/signal.TreeView.row-expanded.html) of `GtkTreeView`.

A caveat with `GtkTreeView` was that the icon to expand/collapse a row isn't shown unless the node has at least one child. This meant adding a "dummy" row and deleting it before we add new entries (like when a row is expanded).

You can find the Git commit [here](https://github.com/shanmukhateja/mystudio-ide/commit/aefd1189917a53a7d76ea3af2bd3b629d760248a).

### Show hidden files (dotfiles) in Workspace Explorer

This was a simple fix to `read_dir_recursive` function which just set `skip_hidden` parameter to `false`.

### Find in Files

This feature is one of my favorite additions to the IDE. I use it often in other code editors when navigating a project.

I created a `Gtk::Dialog` inside the `main_window.glade` file and added relevant code to show rows of a `Label` & `SourceView` with all the *bells and whistles from the* main editor seen beside Workspace Explorer.

The `Label` would contain the full path of the search result followed by the line and column number and the `SourceView` would contain the code along with the cursor set to the line.

I am having trouble scrolling the viewport to the active line number inside `SourceView` which will be fixed in a future release.

After shipping this feature in v0.1.6 release, I realized I had forgotten to implement opening search results inside the main editor💀.

This has been fixed very recently - I have added a section for this so continue reading!

### Config File Support

This feature was introduced to allow customizing the IDE without having to rebuild as I'd like to.

> Currently, we allow window width and height settings by the config file. More features will be added as time goes on.
> 
> This file will be generated at first run (if not exist) and can be found in `%APPDATA%\Roaming\mystudio-ide` on [#Windows](https://social.linux.pizza/tags/Windows) and `$HOME/.config/mystudio-ide` on [#Linux](https://social.linux.pizza/tags/Linux).
> 
> \--
> 
> Sourced from my Mastodon post

You can find my Git commit [here](https://github.com/shanmukhateja/mystudio-ide/pull/8).

### Add .dockerignore

This change was needed to skip sending the `target` and `release` directories to the Docker "context" on every build. In Rust projects, the source code needs to be compiled before it can be executed. These compiled artifacts are stored in `target` directory.

The `release` directory stores the production build of the project which is used to publish a new release on GitHub.

This was an oversight on my part and I should've fixed it sooner.

### Find in Files (again)

As mentioned above, I completely missed the fact that my Find in Files implementation does not open the search result in the main editor section.

To fix this, I had to first refactor the communication pipeline using struct which `init` s' the pipes during init and saves a reference to `tx` or sender via a [Thread Local Storage](https://doc.rust-lang.org/std/macro.thread_local.html).

Now, I can access the `Sender` by simply calling `Comms::sender()` inside `ListBoxRow` `button-press` [event](https://docs.gtk.org/gtk3/signal.Widget.button-press-event.html).

### Refactoring IDE internals

I've been migrating public functions inside each file with a `struct` for cleaner code.

In my opinion, this makes the code more Rust-like and follows the best practices of the Rust language.

## What's Next?

Here are a few things I'd like to see in the future in no specific order:

1. Language Server Protocol
    
2. Messages panel for logging IDE internal messages and errors
    
3. Integrated terminal
    
4. Ability to create new files/folders within a workspace.
    
5. Build & Run a program
    

## Conclusion

If you've made it this far, well I thank you for your interest. This project is my way of learning Rust by building complex applications.

Please consider following me on [Mastodon](https://social.linux.pizza/@shanmukhateja) so you don't miss my new articles.

Bye for now :-)