---
title: "How to install open-webui the easy way on Linux"
datePublished: Sun Feb 16 2025 06:10:47 GMT+0000 (Coordinated Universal Time)
cuid: cm7789via000b09js8wofd74q
slug: how-to-install-open-webui-the-easy-way-on-linux
tags: python, llm, webui, ollama

---

Hello,

I will show you how to quickly and easily setup [open-webui](https://github.com/open-webui/open-webui) in your Linux distro.

This will allow using Ollama models (which you have already installed locally) from a web user interface (similar to ChatGPT).

# Prerequisites

Python 3.11 is recommended for the project. You can try this with global Python installation but make sure the version numbers match.

> If you’d like to install Python 3.11 manually, please look into [**pyvenv**](https://github.com/pyenv/pyenv)**.**

# Installation

1. I use a “Projects“ folder to store all active projects.
    

```bash
$ cd /home/$USER/Projects
```

2. Now, we’ll create a virtual environment to isolate open-webui project. (**Recommended for Arch and Arch based distros)**
    

> Note for Arch & Arch-like distro users: As of writing, Arch and Arch-like distros are shipping Python 3.13 which open-webui doesn’t support yet. (tested on v0.5.10 [GitHub](https://github.com/open-webui/open-webui/blob/v0.5.10/pyproject.toml#L115))

```bash

# Make a virtualenv for this project called "webui".
$ python3 -m venv webui

# In case of Pyenv
$ pyenv exec python -m venv webui 
```

3. Install the `open-webui` package.
    

```bash

# The below commands work in both global as well as pyenv installation of Python

$ cd /home/$USER/Projects/webui
$ source bin/activate
$ pip install open-webui
```

This command will now install open-webui to your project This will take some time depending on your internet connection.

4. Run open-webui
    

```bash
$ open-webui serve
```

This should initialize the software and start listening for connections at `http://localhost:8080`.

5. Open Firefox/Chrome and go to `http://localhost:8080`. Provide a name, email and password to login to your instance and that’s it!!
    
    ![Open WebUI Get Started page](https://cdn.hashnode.com/res/hashnode/image/upload/v1739684781931/c8575d32-f6bd-48b5-9ba0-094389f1a288.png align="center")
    
    # Update open-webui
    

1. To update open-webui to latest version, look for a notification from open-webui screen informing us there is a new version as shown below:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739685374138/1a457991-d59f-4b62-8c91-5b689dfc9093.png align="center")
    
2. Now, stop all processes of `open-webui` locally. **This step is important.** In most cases, you simply need to close the terminal which is running `open-webui serve`.
    
3. Run the commands below:
    

```bash
$ cd /home/$USER/Projects/webui
$ source bin/activate
# Replace <version> with the version number from notification.
# In my case, it is `0.5.12`
$ pip install open-webui==<version>
# Assuming no errors, let's run the serve command now.
$ open-webui serve
```

4. If you do not see any errors, try visiting `http://localhost:8080`. In most cases, this will work out-of-the-box.
    

# Conclusion

This blog post summarizes the steps involved in getting `open-webui` working on my machine. The instructions are generic *enough* to be used for other Linux distros.

Please leave a comment if I missed something.

You can also follow me on [Mastodon](https://social.linux.pizza/@shanmukhateja/) for more updates.

Bye for now :)