---
title: "GitRaven: My next toy project"
datePublished: Fri Aug 01 2025 16:00:31 GMT+0000 (Coordinated Universal Time)
cuid: cmdt0fodd000102l71dwc7kmc
slug: gitraven-my-next-toy-project
tags: cpp, software-development, projects, git, qt

---

Hi

It’s been a while since I built something for fun. I will be covering my journey on GitRaven. It’s a VCS GUI similar to GitHub Desktop or SourceTree without the code reviews, PR merge or other advanced capabilities.

The idea is to try to build this project in a new language with little experience and take my readers along for the ride. This will be my interesting project of 2025 and beyond.

# Background

GitRaven will serve as MY replacement for VS Code’s Version Control feature. I use it on all projects even when using a different IDE. It is easy to work with (to me at least) and helps me visualize the `git status` output.

The project aims to help me maintain my projects wrt source code management. See my goals below pushing the changes to the remote Git server. This means no PRs, MRs, ticket management, etc. It’s a simple diff viewer + editor and general Git repo management tool.

The project is being built on C++ with Qt Widgets. This will be my first huge project on C++ however I have [written C++ before](https://hashnode.com/post/cm8k2y60n000109jogb3khvk4).

# Goals

## Primary

1. Serve as my VS Code’s replacement tool for managing Git repos.
    
2. Cleaner design, simple to use. Always be “straight to the point”.
    
3. Use `libgit2` for dealing with Git stuff instead of relying on Git binary (unlike VS Code).
    
4. Use `KTextEditor` for text viewing and editing purposes. I have been wanting to try it ever since I used `GtkSourceView` on my other project.
    

## Bonus

1. Learn Git internals
    
2. Learn C++
    
3. Fewer progress bars, loading screens and “Please wait” messages when using the app.
    

# Tech Stack

## TL;DR

1. **C++**
    
2. **Qt**
    
3. KTextEditor
    
4. libgit2
    

I originally intended to use Rust just like my [other toy project](https://blog.suryatejak.in/series/mystudio-ide) and I did build a prototype with it. I went so far as fetching Git branch details from a given local directory using `libgit2` but I later realized it didn’t support `KTextEditor`.

I was leaning towards C++ by then as I have been itching to try it ever since I got a confidence boost when I created a my own KDE Dolphin plugin. You can read about it [here](https://hashnode.com/post/cm8k2y60n000109jogb3khvk4).

I did try using Qt with Rust with `cxx-qt` but I couldn’t figure out how to use a struct as a child property inside a parent struct. I opened a ticket on the Discussions section of `cxx-qt` [here](https://github.com/KDAB/cxx-qt/discussions/1313).

Meanwhile, I wanted to try C++ approach for testing purposes and so, I put my Rust+Qt prototype on hold and fired up Qt Creator to experiment with C++ with Qt. I was trying to replicate the prototype functionality in C++.

I managed to make good progress on it and go beyond the details. I was already having fun figuring out C++ and Qt nuances while working on the prototype like using a QSplitter for QTreeView and diff view, add mock data to render the tree items just like the prototype, etc.

Finally, I decided to pull the plug on Rust implementation and focus on the C++ app.

# Current Progress

Here’s how the app looks like as of writing:

![GitRaven Qt + C++ prototype](https://cdn.hashnode.com/res/hashnode/image/upload/v1753978549306/7f014cda-0b9a-40e4-8a3f-8c499b040482.png align="center")

I managed to write a custom delegate class for rendering the Git status text (*the “D” and “U” text on the right*)

You can see a basic diff view of the selected file using `KTextEditor` . It is still long ways before I can call it a “diff” without the quotes but its a start.

# AI Usage

I used ChatGPT and Gemini on this project to solve certain challenging problems after many hours of researching didn’t yield results. I have searched including but not limited to Qt forum, StackOverflow, dev blog posts, etc.

I personally am not a fan of AI tools. There is a potential use case in the future but right now it’s imitation imitating reality at the worst case and I do not like it. Hence, they are my last resort.

When I am stuck with an issue, I ask my query and look for some context. If it presents an answer which worked, I went back to compare my code with it’s proposed changes to figure that part.

It was a wild run and I am happy with my choices so far. I will list down these adventures in my future posts.

# Next Steps

## Diff Viewer & Editor

### Need to decide on approach

1. Build a diff viewer on top of `KTextEditor`
    
2. Replace `KTextEditor` with [QtWebEngine](https://doc.qt.io/qt-6/qtwebengine-index.html) + [monaco-editor](https://microsoft.github.io/monaco-editor/). Monaco provides an out-of-the-box diff viewer component which can be seen inside VS Code and it’s forks.
    
    > Want to learn more about Monaco editor? I wrote about it some time ago [here](https://blog.suryatejak.in/do-you-know-about-the-library-that-powers-vscode).
    

I am personally not a fan of (2). It embeds a web browser into the project just for diff viewer component. I would much rather stick with (1) and figure out how to build a diff viewer out of `KTextEditor`. This will be a technical challenge due to my beginner C++ skill level.

Also, I am aware of [Kompare](https://apps.kde.org/kompare/) app by KDE. It uses its own diff viewer component similar to my (1) approach. Kompare is a GPL project and I intend to license GitRaven as MIT just like all my other projects. This means, I need to *carefully* figure out how to integrate GPL code into an MIT project.

I was thinking of moving the diff viewer code into a GPL licensed `libravendiff` static library. I can dynamically link it to GitRaven’s executable.  
This way, I need to make changes to the library while respecting both licenses. Is this feasible?

Lastly, what approach would you pick?

## Git Features

1. `git fetch`
    
2. List all branches and checkout to different branch
    
3. Pull/Push changes
    
4. Generate or apply Patch files for selected files.
    
5. Stash
    
6. Rebase & Merge support
    

# Conclusion

Thank you for reading my post. I plan on updating the blog when I’ve made some progress on the project.

You can @ me on [Mastodon](https://social.linux.pizza/@shanmukhateja/) or leave a comment below if I missed something.

Bye for now :-)