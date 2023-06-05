---
title: "auto-dark-theme: Let's build a CLI tool for convenience"
datePublished: Mon Jun 05 2023 05:56:41 GMT+0000 (Coordinated Universal Time)
cuid: cliifvw1l000309mohf7rg49y
slug: auto-dark-theme-lets-build-a-cli-tool-for-convenience
tags: kde, linux, python, developer, command-line

---

Hello,

In my previous posts [here](https://hashnode.com/post/clh4dx437000p09l10u1v1hm2) and [here](https://hashnode.com/post/clhmve1ly000109jt5leo1oso), I talked about creating a Python app that switches my KDE Plasma's theme to light or dark according to the system time. This app uses a config file that lives in `$HOME/.config/auto-dark-theme` as per [XDG user base specification](https://wiki.archlinux.org/title/XDG_Base_Directory).

## Background

I wanted a way to "force" light or dark theme using my app for testing purposes without having to tweak the config file (and restart the systemd service).

This CLI will help me *temporarily* switch the theme irrespective of system time until the next trigger.

## Theory

The idea is the CLI should allow users to forcefully switch from a light or dark theme with a simpler syntax.

Next, it should allow the user to view their current config preferably in a table on their terminal.

We should provide a helper shell script to call `cli.py` without having to use `python -m auto-dark-theme.cli` every time.

### Valid commands

1. It should support a `--theme` option with `-t` as its shorthand that allows either `light` or `dark` as the supported parameters.
    
2. It should support a `--list-config` option that parse the config file and displays it in a table.
    

## Implementation

> You can find the latest code for the CLI on the GitHub repo [here](https://github.com/shanmukhateja/auto-dark-theme/blob/main/auto-dark-theme/cli.py).

I create a new file `cli.py` inside `auto-dark-theme` as shown below:

```bash
App_Root
├── auto-dark-theme
│   ├── cli.py          # Hello, there.
│   ├── config.py
│   ├── dbus.py
│   ├── __init__.py
│   ├── __main__.py
│   ├── __pycache__
│   ├── spawn.py
│   └── switcher.py
├── config
│   ├── autodarktheme   # Helper shell script.
│   ├── auto-dark-theme.service
│   └── config.sample.ini
├── LICENSE
├── pyvenv.cfg
├── README.md
├── requirements.txt
└── setup.cfg
```

I will be using the excellent [argparse](https://docs.python.org/3/library/argparse.html) library for this as I have prior experience with it.

```python
# cli.py

import argparse
# Create an instance of ArgumentParser
parser = argparse.ArgumentParser(
    prog='auto-dark-theme', 
    description='Switch your DE theme between light/dark themes acc. to specified time for KDE Plasma'
)
```

`ArgumentParser` takes care of managing the CLI interface and setting up some defaults like printing output similar to other CLIs, printing the usage section if user inputs were incorrect and type-casting user inputs (for `int`, `float`, etc.) among others.

```python
# cli.py

# Add arguments supported by CLI
# -t || --theme
parser.add_argument(
    '-t', '--theme', choices=['light', 'dark'], required=False, help='Forcefully switch between light/dark theme.')
# List config
parser.add_argument(
    '-l', '--list-config', action='store_true', help='List current config.')
```

We use [add\_argument](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.add_argument) here to add arguments supported by our app. In this case, I need to support 2 arguments:

1. `-t` or `--theme` for theme switching
    
2. `-l` or `--list-config` for listing current config
    

```python
# cli.py

from tabulate import tabulate
from .switcher import ThemeSwitcher
from .config import AppConfig

# Parse & process
result = parser.parse_args()

# Trigger theme change if provided
if result.theme == 'light':
    ThemeSwitcher().switch_to_light()
if result.theme == 'dark':
    ThemeSwitcher().switch_to_dark()

if (result.list_config):
    app_config = AppConfig().list_config()
    print(tabulate(app_config, headers=['Option', 'Value'], tablefmt="outline"))
```

In the above lines lies the core logic for driving the argument parsing and performing tasks according to user input.

[`parser.parse_args()`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.parse_args) will run the parser on user input and place the resultant values inside the `result` object.

If `-t` or `--theme` are used, the first two `if` 's will be triggered according to user input.

If `-l` or `--list-config` is used, we create an instance of `AppConfig` which returns a list of items to be shown to the user. I am using [tabulate](https://pypi.org/project/tabulate/) Python library for generating pretty tables for this. It's so nice :^)

Lastly, let's create a helper script for running auto-dark-theme CLI without calling the Python file directly.

```bash
#!/bin/sh

# I know, I know. This is hardcoded and will be fixed 
# once I figure out setup.py for distribution.
/home/suryateja/Projects/auto-dark-theme/bin/python -m auto-dark-theme.cli $1
```

## Result

Here lies the screenshot of the usage section for the newly created CLI:

![Terminal output for auto-dark-theme CLI tool usage](https://cdn.hashnode.com/res/hashnode/image/upload/v1685943212848/62252632-5472-4a8a-b8af-62ec26f6b462.png align="center")

And the output for the list config argument is shown below:

![Terminal output for auto-dark-theme CLI showing the current config](https://cdn.hashnode.com/res/hashnode/image/upload/v1685943618154/b691fea0-00b6-499e-ac2d-35d560117f1a.png align="center")

## Next steps

Create a `setup.py` file for two reasons:

1. Installing the app should be a simple `pip install --user auto-dark-theme` away!
    
2. Get rid of all hardcoded paths that link to my local machine.
    

## Conclusion

I hope you've learned something new in this post. Please feel free to add your thoughts below. I would love to know them!

If you've liked this post, show your support by using the emojis on the right. It pleases the algorithm :^)

You can also @ me on Mastodon [here](http://social.linux.pizza/@shanmukhateja).

Bye for now.