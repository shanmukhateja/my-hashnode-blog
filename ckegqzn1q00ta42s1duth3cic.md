## I wrote a quick fix patch to handle display brightness issues with AMD Ryzen for my laptop

## Background

I recently purchased an ASUS TUF FX505DT "gaming" laptop and installed Manjaro Linux as dual-boot to pre-installed Windows 10. My laptop comes with an AMD Ryzen 3550H with Vega 8 Graphics which is "bleeding-edge" hardware for Linux and bugs were to be expected.

One of such bugs that annoyed me was that my display's brightness settings were being reset to maximum on every reboot. This means, at each startup,  I will have to manually dial back the brightness to my comfortable value which would reset again on the next shutdown/reboot. This HAD to be fixed.

### Systemd 101: 

In modern Linux distributions, we have an init manager called `systemd` that takes care of initializing system and user processes so users can get on with watching cat videos on Internet. 

According to Wikipedia:

> systemd is a software suite that provides an array of system components for Linux operating systems.

And

> an init system used to bootstrap user space and manage user processes. It also provides replacements for various daemons and utilities, including device management, login management, network connection management, and event logging.
 
In simpler terms, it is an "init system" that loads system processes. In Windows land, think of systemd as "a software for loading system stuff like user login screen, Windows desktop that you see after logging in, checking network connectivity, various tray icons and backend daemons, etc. as well as launching all user software in Startup section like CCleaner or Java updater tray icon or Realtek."

## Problem:

The problem arises with a recent [patch](https://github.com/torvalds/linux/commit/262485a50fd4532a8d71165190adc7a0a19bcc9e) to Linux kernel by AMD.
The patch aims to provide precise brightness information to userspace applications (like systemd for instance) but had introduced a bug where it reports current brightness value of display as a 16-bit integer instead of an 8-bit integer.

In Linux, your display brightness is determined by 3 values stored as **8-bit integers**  - `min_brightness`, `brightness` (i.e. current brightness) and `max_brightness`.  According to the spec, `brightness` value should always be higher or equal to `min_brightness` but lower than `max_brightness`.

The recent patch I mentioned earlier introduced a bug where `brightness` started being reported as a 16-bit integer without updating `min_brightness` and `max_brightness` cousins. 

For example, if `min_brightness` = 0 and `max_brightness` = 255, then `brightness` value after the said patch started being reported as `10280` which is 16-bit for `40`.

`systemd-backlight` has been written to set `brightness` to `max_brightness` when invalid `brightness` value is provided. Do you understand why backlight was reset to maximim now?

You can find more information about my fix for this in my [patch](https://github.com/shanmukhateja/systemd/commit/b6596592e12af1f64f604c408410f01048edb5c3).

## Theory:

I need to `Math.floor` but in C the value of `brightness` by checking if its exceeding `max_brightness` and dial it back such that `min_brightness >= brightness < max_brightness` when necessary.

## Code:

I cloned the latest release of `systemd` which was v246, as of writing, [here](https://github.com/systemd/systemd/releases/tag/246) and opened the directory in VS Code. 

Then, I navigated to `src/backlight/backlight.c`, learned they were using pointers to handle code, got nervous, took a deep breath and kept on pushing myself to try and understand the code.

Once that hurdle was crossed, I found `clamp_brightness` function which ensures `brightness` value is in range of `min_brightness >= brightness < max_brightness`. Here, I added the required [code](https://github.com/shanmukhateja/systemd/commit/b6596592e12af1f64f604c408410f01048edb5c3#diff-770bd1b9a890d1dc32cfc71b656073beR289) to check if `brightness` was exceeding `max_brightness` (due to it being mistaken for 8-bit integer) 

I also found `run` function [here](https://github.com/shanmukhateja/systemd/commit/b6596592e12af1f64f604c408410f01048edb5c3#diff-770bd1b9a890d1dc32cfc71b656073beR289) to handle restoring `brightness` value if necessary.

### Usage Issues:

The one last thing that gave me trouble was the fact that `systemd` codebase was written to be compiled with `-shared` such that compiled executable for my patched `systemd-backlight` needed  `.so` files which are DLLs for Linux to be placed in same directory.

My workaround for this was to symlink my patched binary located in my $HOME dir to /usr/lib/systemd/system-backlight` and reboot for changes to take effect. IT WORKED 🎉


## Conclusion:

I learned so much about systemd and the way it handles system components. I also had to learn C especially performing data type conversions when pointers are involved. 

*Bonus:* Did you know `systemd-backlight` was also used to save/restore your keyboard's backlight settings as well?