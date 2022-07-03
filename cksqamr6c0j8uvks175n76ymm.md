## MyStudio IDE: I'm building an IDE with Rust+GTK!

## Background:

In my journey for learning Rust, I've started a side project called MyStudio IDE which is, as the name implies, an IDE built on Rust.

Shoutout to [Andreas Kling](https://twitter.com/awesomekling) and his Serenity OS project for inspiration. 

> This project is meant as a learning experience. If you're hoping for something like VS Code, I'm sorry to disappoint you.

Let's build a Hello world GTK program with Rust!

#### Prerequisites:

1. GTK3 [LINK](https://gtk.org)

2. Rust Lang, Cargo, etc. [LINK](https://rust-lang.org)

3. VS Code, subl, etc.

#### Terminology:

__crate__: 

An external library that contains modules and provides some functionality. 

__crates.io__: 

The Rust lang's official package registry. It's NPM for Rust.

__cargo__:

Cargo is a CLI tool to work with Rust projects. A beginner's analogy to this would be JS Angular's `ng` CLI tool.

#### Project Setup:

I create a new project by running:

`cargo new my-studio` 

You should see output similar to:

```
~/projects: cargo new my-studio

Created binary (application) `my-studio` package
```

By default, cargo provides us with a hello world. Let's run it.

```bash
~/projects: cd my-studio
~/projects/my-studio: cargo run
```
> This will take time. Please be patient...

After sometime, you'll see something like this:

```bash
Compiling my-studio v0.1.0 (/home/user/projects/my-studio)
Finished dev [unoptimized + debuginfo] target(s) in 0.80s
Running `target/debug/my-studio`
Hello, world!
```

Cool! Let's move on.

### GTK + Rust

Let's add the [gtk-rs](https://gtk-rs.org/) crate to Cargo.toml at the project root. So, Cargo.toml is like package.json for Rust.

__Cargo.toml__

```
[package]
name = "my-studio"
version = "0.1.0"
edition = "2018"


[dependencies]
gtk-rs="0.14.2"  #  <-- Include it here
```

> At the time of writing, 0.14.2 is the latest version of the crate. Please check [crates.io](https://crates.io/) for new releases.

#### Hello World

This code is borrowed from `gtk-rs`repo on GitHub. 
[Link](https://github.com/gtk-rs/gtk3-rs/blob/master/examples/basics/main.rs)

```

use gtk::prelude::*;

fn build_ui(application: &gtk::Application) {
    let window = gtk::ApplicationWindow::new(application);

    window.set_title("First GTK+ Program");
    window.set_border_width(10);
    window.set_position(gtk::WindowPosition::Center);
    window.set_default_size(350, 70);

    let button = gtk::Button::with_label("Click me!");

    window.add(&button);

    window.show_all();
}

fn main() {
    let application =
        gtk::Application::new(Some("com.github.gtk-rs.examples.basic"), Default::default());

    application.connect_activate(build_ui);

    application.run();
}

```

#### Output:

<img src="https://raw.githubusercontent.com/gtk-rs/gtk3-rs/master/examples/basics/screenshot.png" alt="hello world rust">


Hope you liked it! Feel free to @ me on [Twitter](https://twitter.com/shanmukhateja94) on your thoughts on this. Bye :)