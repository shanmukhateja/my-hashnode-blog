## MyStudio IDE: How Goto Line feature works

Hello,

I'd recently implemented "Goto Line" feature on my side project, MyStudio IDE and here's how I did it.

## Background:

As a part of building essential features for an IDE, "Goto Line" seemed like an easy milestone. 

There may be other ways to accomplish this but here's how I did it.

## UI Goals:

1. Show a UI widget, "Line X, Column Y" to the bottom-right on the application's status bar.

2. Clicking on it should spawn a dialog with an input field to enter two types of data:
  - a line number
  - a key:pair of line number and column. Ex: 4:2 (line number 4, column number 2)  

3. User can initiate the request by either pressing Enter key or clicking on OK button of the dialog.

4.  This indicator should be visible ONLY when we have an open file. 

5. It should also respond to keyboard shortcut `Ctrl+G` when request is valid.

## UI Design:

### Indicator UI:

![goto_line_wireframe2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654333252333/2ZqyS7nj9.png align="center")

### Dialog:

![goto_line_dialog_wireframe2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654333688885/_YkrZcFj-.png align="center")

## Implementation:

I looked at [gedit](https://wiki.gnome.org/Apps/Gedit) and [KWrite](https://apps.kde.org/kwrite/)  and decided to use `GtkButton` as the line indicator widget instead of `GtkLabel`. 

We start by initializing status bar UI and storing references to button and input field of dialog from `main_window.ui` (Glade resource file) from `Builder` object.

### Feature 1: Changes to cursor position should update line indicator

`TextBuffer` of `GtkSourceView` instance (text editor widget) has a `connect_cursor_position_notify` callback, also called a 'signal' in GTK terminlogy.

This callback, as the name suggests, is triggered each time cursor position changes and it returns a reference to `TextBuffer` instance. 

I wrote [fetch_line_number_by_buffer](https://github.com/shanmukhateja/mystudio-ide/blob/1dabe849c89a575582d3528d76662a0d187afdc6/libmystudio/src/notebook/editor.rs#L19) function to fetch the current line number and column number as a [Tuple](https://doc.rust-lang.org/book/ch03-02-data-types.html?highlight=tuple#the-tuple-type).

With this info, we call the [update]() function that does some sanity checks before updating the UI widget. 
> TextBuffer returns a zero-based number for the line numbers which is why we need to adjust the values as required before updating the indicator. 

### Feature 2: Jump to given line number & update indicator


Just like before, we need access to `TextBuffer` to change the cursor's position for a given text editor instance.

I wrote the [jump_to_line_with_editor](https://github.com/shanmukhateja/mystudio-ide/blob/1dabe849c89a575582d3528d76662a0d187afdc6/libmystudio/src/notebook/editor.rs#L4) function that takes 3 parameters - line number, column number and a`GtkSourceView` (text editor) instance.

Inside this function,  we get access to [TextIter](https://docs.gtk.org/gtk3/struct.TreeIter.html) object of the `TextBuffer` for a given line number AFTER decrement it by 1.

The `line - 1` is necessary because TextBuffer assumes given line number will start from zero.

Later, we set the column number (after decrement by 1, ofcourse!) to this `TextIter` object and use `TextBuffer`'s `place_cursor` function to update the UI.

> We don't have to validate the user input here as this is handled by text editor instance. For example, if user provides a line number greater than total lines of an open file, it will simply go to the last line.

## End Result:
![ss_line_1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655490507949/r1VUTjIAU.png align="center")

![ss_goto_dialog.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655490631187/v07HVvytL.png align="center")

![ss_line_42.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655490571874/3W9mf_vdo.png align="center")

Thanks for reading! Leave a comment down below on your thoughts or any ideas for this.

Don't forget to give it a like to this post seen to your right :)

You can @ me on [Twitter](https://twitter.com/shanmukhateja94)