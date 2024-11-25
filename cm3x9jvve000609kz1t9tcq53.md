---
title: "GhostLLM: Add LLM magic directly into GTK3 GtkEntry"
datePublished: Mon Nov 25 2024 16:49:45 GMT+0000 (Coordinated Universal Time)
cuid: cm3x9jvve000609kz1t9tcq53
slug: ghostllm-add-llm-magic-directly-into-gtk3-gtkentry
tags: linux, gtk, llm, ollama

---

Hello,

This is part one of my series of fun project that felt challenging for me. Read on to know more :)

# Background

I’ve been using [Ollama](https://www.ollama.com/) for running LLM models locally on my Linux machine for more than a year. It allows us to use many LLM models from the CLI.

During Apple’s Keynote this year, I was fascinated with “AI sentence rewriting” feature available starting from macOS Sequoia.

[This](https://www.google.com/search?sca_esv=c744cb070de47b7e&q=macos+sequoia&spell=1&sa=X&ved=2ahUKEwi854XF3PeJAxXdd2wGHSHJEBYQkeECKAB6BAgQEAE) led me down a rabbit hole for ways to extend existing input fields (at least ones that use GTK3) with LLM model for quickly rewriting sentences.

It is possible that things might not go my way however I picked up this project for 2 things:

1. Better understanding of GTK text entry widgets
    
2. A silly project to keep my nerdy brain occupied :)
    

I have decided to name this project “GhostLLM”. It is based on the word [ghostwriter](https://www.google.com/search?q=ghostwriter+meaning) with “LLM” joined to the hip. I think it fits simply because the LLM models will be the ghostwriters every time I ask it to rewrite a sentence.

# Theory

The idea is to fork GTK library and add a dedicated “GhostLLM” submenu for right-click context menu when text is selected. When user selects “Rewrite Text” option inside this menu, we will call DBus interface which will provide the alternative sentences.

This means I need to setup DBus address that can process the request. It would be written in Python (or Rust) and will be responsible to communicate with Ollama for models.

> I will be using GTK3 **version 3.24.43 for this project**. It’s what Manjaro Linux (my daily driver) shipped as of writing.

### GtkEntry instead of GTKTextView

As of writing, I could not find software that still used GTK3 + GTKTextView on Manjaro’s repos. Honestly, I didn't search thoroughly so I will revisit this in the future.

For the time-being, I chose [GtkEntry](https://docs.gtk.org/gtk3/class.Entry.html) widget. It is used as the address bar on [Thunar](https://docs.xfce.org/xfce/thunar/start) (which I have installed on my machine alongside Dolphin). It will make it easier to compile and test my changes.

# Code

I have written the necessary boilerplate code to get things moving. The callback for “Rewrite Text” option simply prints sample text to the console.

```diff
diff --git a/gtk/gtkentry.c b/gtk/gtkentry.c
index a71578218c..3bafbee079 100644
--- a/gtk/gtkentry.c
+++ b/gtk/gtkentry.c
@@ -5921,6 +5921,12 @@ gtk_entry_backspace (GtkEntry *entry)
   gtk_entry_pend_cursor_blink (entry);
 }
 
+static void
+ghostllm_rewrite_text_cb (GtkMenuItem *menuitem, gpointer user_data)
+{
+  g_print("hello from rewrite text callback!!");
+}
+
 static void
 gtk_entry_copy_clipboard (GtkEntry *entry)
 {
@@ -9601,6 +9607,26 @@ popup_targets_received (GtkClipboard     *clipboard,
                             mode == DISPLAY_NORMAL &&
                             info_entry_priv->current_pos != info_entry_priv->selection_bound);
 
+      // GhostLLM stuff
+      
+      // The root GtkMenuItem that shows "GhostLLM" option on the context menu
+      GtkWidget *ghostllm_root_menuitem = gtk_menu_item_new_with_mnemonic (_("GhostLLM"));
+      gtk_widget_set_sensitive(ghostllm_root_menuitem, info_entry_priv->editable && info_entry_priv->current_pos != info_entry_priv->selection_bound);
+
+      // The GtkMenu widget which holds the LLM menu options.
+      GtkWidget *ghostllm_submenu = gtk_menu_new ();
+      
+      menuitem = gtk_menu_item_new_with_mnemonic (_("_Rewrite Text"));
+
+      g_signal_connect_swapped (menuitem, "activate", G_CALLBACK (ghostllm_rewrite_text_cb), entry);
+      gtk_widget_show (menuitem);
+
+      // Append "Rewrite Text" option into LLM submenu
+      gtk_menu_shell_append (GTK_MENU_SHELL (ghostllm_submenu), menuitem);
+
+      // Set submenu options for GhostLLM root menu item
+      gtk_menu_item_set_submenu ( GTK_MENU_ITEM (ghostllm_root_menuitem), ghostllm_submenu);
+
       append_action_signal (entry, menu, _("_Paste"), "paste-clipboard",
                             info_entry_priv->editable && clipboard_contains_text);
 
@@ -9615,6 +9641,11 @@ popup_targets_received (GtkClipboard     *clipboard,
       gtk_widget_show (menuitem);
       gtk_menu_shell_append (GTK_MENU_SHELL (menu), menuitem);
 
+      gtk_widget_show(menuitem);
+      gtk_widget_show(ghostllm_root_menuitem);
+      // Append the GhostLLM root GtkMenuItem into GtkEntry's context menu
+      gtk_menu_shell_append (GTK_MENU_SHELL (menu), ghostllm_root_menuitem);
+
       menuitem = gtk_menu_item_new_with_mnemonic (_("Select _All"));
       gtk_widget_set_sensitive (menuitem, gtk_entry_buffer_get_length (info_entry_priv->buffer) > 0);
       g_signal_connect_swapped (menuitem, "activate",
```

## LD\_PRELOAD

After successfully compiling GTK library with the above changes, I found the modified GTK3 library at `./_build/gtk/libgtk-3.so`.

You can find building instructions on `INSTALL.md` file after checking out the `3.24.43` tag.

I tried `sudo meson install -C_build` to install this version globally on the system but it didn’t work. Hence, I went with \`LD\_PRELOAD\` approach to test the changes.

```bash
$ LD_PRELOAD=/home/suryateja/Projects/gtk3/_build/gtk/libgtk-3.so thunar
```

# What’s Next?

In the next part, I will write GhostLLM daemon which will create DBus address that will talk to LLM models via [Ollama endpoints](https://github.com/ollama/ollama/blob/main/docs/api.md).

I have yet to decide whether to go with Python or Rust for this part. Please add a comment to let me know your thoughts.

# Conclusion

Thank you for reading. Give this post a Like to show your appreciation for this post.

You can follow me on [Mastodon](https://social.linux.pizza/@shanmukhateja/) for more updates.

Bye for now :-)