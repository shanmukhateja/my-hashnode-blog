## MyStudio IDE: Windows + UTF-16 encoding - Software

Hello,

This post is about an interesting bug I found in MyStudio IDE which I felt needed to be shared here.

## Background

I was using [quick-bt](https://github.com/shanmukhateja/quick-bt/) as the test project (workspace) when I noticed the IDE failed to open [requirements.txt](https://github.com/shanmukhateja/quick-bt/blob/master/requirements.txt) to read/edit.

As a part of testing [Windows](https://surya-dev-journey.hashnode.dev/mystudio-ide-cross-compile-rustgtk-for-windows) builds of [MyStudio IDE](https://github.com/shanmukhateja/mystudio-ide/releases/tag/v0.1.3), I ran into an interesting issue with one of my test projects.


## The Problem:

I saw "File Not Supported" (declared [here](https://github.com/shanmukhateja/mystudio-ide/blob/79f3ff630265acf2525c4280b57f89ea7aa59d72/src/ui/w_explorer/tree_view.rs#L168)) error when opening the `requirements.txt` file on the repository. 

This error was meant for binary files like _images, PDFs, binaries or executable files, etc._ so it didn't make sense why would this file trigger it.

![ss_mys_ff_error.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658063244727/8Jq-FbO7X.png align="center")

I switched to VSCode and opened the same file. It was working fine; I could see the content and can make changes with ease.

Take a moment and check the error declaration link above. See if you can find why this was happening.

...Done? Okay, good! 

I began searching for clues as to what is different with this file and a few moments later, as I glanced at the bottom-right of the window, I realized what's wrong.


![ss_mys_highlight_encoding.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658063827451/d8x65wNLD.png align="left")

The file was in UTF-16 encoding! This tracks because I worked on this on Windows environment and as you might know, Windows uses UTF16 as the default encoding. 

> Did you know Windows Kernel still has a few pieces with UTF-16 encoding? This is their commitment to ensure backwards compatibility with legacy software in Windows.

`LE` stands for  **L**ittle **E**ndian and it determines the byte-ordering used in a file. Here's a Wikipedia link that explains it [better](https://en.wikipedia.org/wiki/Endianness).

In simple terms, it determines the order or sequence with which set of bytes are stored in a file. In this case, the `UTF-16 LE` means its a text which is UTF-16 encoded whose byte ordering is in Little Endian format.

Still with me? Don't worry. All this tech jargon was meant to help you understand what `UTF-16 LE` meant.

### Rust + UTF-8

Rust uses UTF-8 internally and so it's standard library assumes UTF-8 when accepting input or processing IO operations. 

There are few places like [String](https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf16) where exceptions are made however file IO doesn't work with UTF-16 out-of-the-box.

For instance, we don't have to concern ourselves with all of this in say, Java. `ByteArrayOutputStream` would take care of storing given input file as a form of bytes without making it. We can also go with Apache Commons JAR for abstracting this even further.

## So, what now?

### GUI
Once I realized what went wrong, I wanted to add a GUI for showing currently open file's encoding. This write-up focuses on the software implementation. I'll make a new post about the GUI adventure in the near future. 

### Software

I went on [crates.io](https://crates.io/) which is like a repository of third-party packages for Rust. In Rust_-land_, what's called a `package` in say, JavaScript, is labelled `crate`. I don't know why that is though :\

Anyway, the goal was to locate a crate that can work with different file encodings which potentially also handles endianness. As a text editor, a user can open a text file of any encoding and be able to work on it without issue just like I did with VSCode when testing this bug.

So, I found this crate called `encoding_rs` which supports a wide variety of encoding standards. The exact list can be found [here](https://crates.io/crates/encoding_rs). I was very excited to work on this but then came more problems.

### Working without encoding_rs_io

I kind-of brought this on myself by choosing not to use their helper crate, [encoding-rs-io](https://crates.io/crates/encoding_rs_io) which integrates with `std::io` to read/write non-UTF-8 files.

I tried to approach this in different ways like trying to obtain `u8` vector slices (byte arrays) off UTF-16 content and saving them to disk all the while preserving UTF-16 LE. So standard IO function expects `u8` vector slices.

I could've gone down the rabbit hole by left-shift any bytes out of range of `u8` and maybe try to save it but I never got that far.

Looking back, I should've gone with `encoding_rs_io` crate and implement it in `libmystudio` but we developers can be stubborn at times!

Anyway, this went on for few more days where I continued to fail this task and a few more days later, I couldn't find time anymore as I had to prioritize the day-job. I finally decided it was time to use a work-around and move this to _TODO_ state.

## Solution

The catch with Rust's File IO APIs is that they default to UTF-8 encoding. This means, when MyStudio tries to save a UTF-16 encoded file, it will instead be overwritten as a UTF-8 encoded file or [panic](https://doc.rust-lang.org/std/macro.panic.html).

Hence, I chose to **not allow** saving a file if it's not using UTF-8 encoding. This way, I can prevent potential data loss or file content mismatch. This is temporary and once I find the time, I plan on fixing it.

Check my Git commit [here](https://github.com/shanmukhateja/mystudio-ide/commit/f07e8a9b939c7497d5394a91cf538ba0aa74d4a1).

The first thing to do is update `read_file_contents` in `libmystudio` to detect UTF-8, UTF-16 encoding and return its contents as `String`. 

If we encounter an "unsupported file type" like binary files or text files with UTF-32 encoding, we return `None` which is a special type in Rust. 
This is facilitated by [Option](https://doc.rust-lang.org/std/option/) keyword and you can think of it as the `Optional` class in Java_-land_.

Lastly, when saving file changes, we use similar logic to determine if file encoding is UTF-8 and if not, we return a `Write support for xxx encoding is unavailable.` error which is visible in the status bar.

![ss_mys_stop_save.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658120378286/BFPdq34uO.png align="center")

## Conclusion

I hope you've learned something interesting here. Give a Like to this post (on the right) and don't forget to add a comment on your thoughts about this.

You can also @ me on [Twitter](https://twitter.com/shanmukhateja94).

Bye for now :-)