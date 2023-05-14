---
title: "auto-dark-theme: Python app"
datePublished: Sun May 14 2023 03:42:04 GMT+0000 (Coordinated Universal Time)
cuid: clhmve1ly000109jt5leo1oso
slug: auto-dark-theme-python-app
tags: kde, linux, systemd, dark-mode, dbus

---

Hello there.

In my previous [post](https://hashnode.com/post/clh4dx437000p09l10u1v1hm2), I talked about how my shell script-based implementation couldn't detect certain system events like screen lock/unlock or system sleep.

In this post, I'll show how the Python implementation fixed these issues.

## Background

### D-Bus

In Linux, inter-process communication happens via a messaging middleware known as [D-Bus](https://en.wikipedia.org/wiki/D-Bus). It is the de-facto way for multiple processes to communicate amongst themselves to inform events or listen to others' events. Any process can listen to and trigger events using the publicly available entities exposed on D-Bus by other processes.

In D-Bus, a process can declare properties (to know the current state of certain features), methods (functions used to perform actions or trigger events) and signals (event listeners) in the context of D-Bus via a **bus**.

There are 2 types of buses in D-Bus to differentiate user-level and OS-level events:

1. System bus -&gt; Operating system level events remain here
    
2. Session bus -&gt; (currently logged-in) user-level events are stored here
    

A connection to such a bus has a unique identifing string which is a *name.* It is a dot-separated string that follows a *Java*\-like package naming convention. For example, `org.freedesktop.Notifications` on the session bus is used to show notifications to the user.

In our case, we are interested in `org.freedesktop.ScreenSaver`. As the name implies, it contains the required signal to listen to screen lock/unlock events.

### Freedesktop?

If you've noticed, the bus names I've listed start with `org. freedesktop` This is not a coincidence.

In simple terms, Freedesktop is an organization where open-source graphical and desktop software gets discussed and standardized. This way, applications can safely rely on these standardized APIs and implement features around them.

## Theory

In our case, we will connect to two D-Bus interfaces:

1. **Screen lock/unlock** - We will connect to `ActiveChanged` signal on `org.freedesktop.ScreenSaver`. It is triggered on every screen lock and unlock.
    
2. **System sleep/resume** \- We will connect to `PrepareForSleep` signal on `org.freedesktop.login1`. It is triggered whenever a machine goes to sleep or resumes from sleep.
    

Also, We will use `systemd` to start, restart or stop the application on major Linux distributions.

## Implementation

> This section will be based on [dbus.py](https://github.com/shanmukhateja/auto-dark-theme/blob/main/auto-dark-theme/dbus.py) from the GitHub repository.

During the initialization of `DbusListener` , we will specify the bus name and the interface path along with the required signal name to listen to the required system events.

We also manually trigger the theme-switching logic at the end of the initialization cycle. I felt this would be expected by the end-user to see the theme change when the app is run manually.

The error handling for this step is incomplete as of writing. The problem is our application is triggered very early in the lifecycle of the operating system startup. This means our D-Bus interfaces are not available to us just yet.

```python
## dbus.py

import dbus

# Main loop
from dbus.mainloop.glib import DBusGMainLoop
from gi.repository import GLib  # type: ignore

from .switcher import ThemeSwitcher

# Create mainloop
DBusGMainLoop(set_as_default=True)
loop = GLib.MainLoop()
GLib.MainLoop()


class DbusListener:

    def handle_lock_unlock(listener, *args):  # type: ignore
        if args[0] == dbus.Boolean(False):
            ThemeSwitcher().run()

    def __init__(self):
        print("Initializing...")
        self.session_bus = dbus.SessionBus()
        self.system_bus = dbus.SystemBus()

        self.iface_screen_lock = self.session_bus.get_object(
            'org.freedesktop.ScreenSaver', '/org/freedesktop/ScreenSaver')
        self.iface_screen_lock.connect_to_signal(
            'ActiveChanged', handler_function=self.handle_lock_unlock)

        self.iface_suspend = self.system_bus.get_object(
            'org.freedesktop.login1', '/org/freedesktop/login1')
        self.iface_suspend.connect_to_signal(
            'PrepareForSleep', handler_function=self.handle_lock_unlock)

        try:
            # Apply theme on init
            self.handle_lock_unlock(dbus.Boolean(False))
            # Start event loop
            loop.run()
        except:
            print("Shutting down..")
            self.session_bus.close()
            self.system_bus.close()
            loop.quit()
```

We let the app exit on an error to allow `systemd` to track it and restart (after a 5 seconds delay) as specified in our service file [here](https://github.com/shanmukhateja/auto-dark-theme/blob/main/config/auto-dark-theme.service).  
In my testing, this delay has given us enough time for the D-Bus interfaces to be up.

## What's Next?

I need to test this app and fix bugs if any. Later, the first stable release will be made which will be a big milestone for the project.

## Conclusion

Thanks for sticking around! I hope you have learned something new from this post.

I would appreciate it if you could use the emojis on the right side to show how much you liked this post.

Connect with me on [Mastodon](https://social.linux.pizza/@shanmukhateja).

Bye for now :)