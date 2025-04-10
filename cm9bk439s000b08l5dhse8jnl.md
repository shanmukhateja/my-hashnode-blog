---
title: "How to use Ctrl+Shift+F keybinding in Neovim?"
datePublished: Thu Apr 10 2025 16:12:42 GMT+0000 (Coordinated Universal Time)
cuid: cm9bk439s000b08l5dhse8jnl
slug: how-to-use-ctrlshiftf-keybinding-in-neovim
tags: linux, neovim, alacritty, vim-keybindings

---

Hello,

This post is a reference on how to generate \`Ctrl+Shift+&lt;key&gt;\` keybindings for Neovim. As you know, Vim (apparently) doesn’t support this since some terminal emulators do not pass key combinations.

# Theory

1. What we’ll be doing instead is asking the terminal emulator (in my case Alacritty) to send a different character (`char`) when \`Ctrl+Shift+&lt;key&gt;\` is pressed.
    
2. So, when the key combination is pressed, a different character will be sent to Neovim.
    
3. At Neovim end, we will bind the required Neovim action to this character instead of `C<-S-f>` or something.
    

For my usage, I will be triggering `Snacks.picker.grep()` whenever I press `Ctrl+Shift+F` on my keyboard.  
  
I use this particular keyboard shortcut very often in VSCode and so trying to retain the pattern in `nvim` (for muscle memory’s sake) if possible :)

# Implementation

> The steps are intended for Alacritty terminal emulator. If you’re using something else, please note, the instructions might be slightly different but you’ll be achieving the same goal here.

1. Open Alacritty config at `$USER/.config/alacritty/alacritty.toml`
    
2. Add the following code snippet below:
    

```ini
# Global search in nvim
# I am using the `https://www.compart.com/en/unicode/U+058D` character as the replacement.
[keyboard]
bindings = [
  { key = "F", mods = "Control|Shift", chars = "֍" },
]
```

3. Now, let’s go to nvim config file at `$HOME/.config/nvim/lua/config/keymaps.lua`
    

```plaintext
-- Global search
map("n", "֍", function()  Snacks.picker.grep() end, {desc="Global search", remap = true })
```

4. Save all changes and test. You should see the “Grep” window open inside nvim when you press `Ctrl+Shift+F`
    

# Conclusion

This post was quickly put together for reference purposes. Sorry if it didn’t include your specific terminal emulator.

Thanks for reading!

Bye for now :)