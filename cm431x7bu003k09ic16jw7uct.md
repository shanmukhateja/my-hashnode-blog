---
title: "GhostLLM : RewriteText now works on GtkEntry!"
datePublished: Fri Nov 29 2024 18:02:46 GMT+0000 (Coordinated Universal Time)
cuid: cm431x7bu003k09ic16jw7uct
slug: ghostllm-rewritetext-now-works-on-gtkentry
tags: artificial-intelligence, linux, development, opensource-inactive, gtk, llm

---

Hello,

This is part three of my GhostLLM project. This project is about integrating “Rewrite Text” functionality natively into GTK3 widgets. Check the [part one](https://hashnode.com/post/cm3x9jvve000609kz1t9tcq53) and [part two](https://blog.suryatejak.in/ghostllm-creating-the-dbus-server-with-ollama-integration) in case you missed it.

# Theory

We will be working with `gtkentry.c` for this part. The goal is to implement `ghostllm_rewrite_text_cb` callback function to replace the user selected text.

The idea is to declare new files - `ghostllm.c` & the header file `ghostllm.h` to deal with DBus related communications. We also update `meson.build` file to include them in the build system.

# Implementation

The code lives on GitHub repo [here](https://github.com/shanmukhateja/ghostllm-gtk3). Please refer the repo link for latest changes :)

Here’s the relevant code:

**ghostllm.h:**

```c
// This is the only line in this file!
char* ghostllm_rewrite_text(char* input_str);
```

**ghostllm.c**

```c
#include "config.h"

#include <gtk/gtk.h>

#include <gio/gio.h>

char *ghostllm_rewrite_text(char *input_str) {
  // Connect to DBus Session Bus
  GDBusConnection *conn = g_bus_get_sync(G_BUS_TYPE_SESSION, NULL, NULL);

  // Define the remote service and object path
  char *service_name = "in.suryatejak.ghostllm";
  char *object_path = "/in/suryatejak/ghostllm";

  // This is required for communicating with DBus.
  GDBusMessage *call_message = NULL;
  GDBusMessage *reply_message = NULL;
  GError **error = NULL;

  // Define the DBus message to be sent.
  call_message = g_dbus_message_new_method_call(service_name, object_path,
                                                service_name, "RewriteText");

  // Set arguments
  g_dbus_message_set_body(call_message, g_variant_new("(s)", input_str));

  // Call the RewriteText method on the remote service
  reply_message = g_dbus_connection_send_message_with_reply_sync(
      conn, call_message, G_DBUS_SEND_MESSAGE_FLAGS_NONE, -1, NULL, NULL,
      error);

  // Retrieve the response data from DBus
  GDBusMessageType reply_type = g_dbus_message_get_message_type(reply_message);

  if (reply_type == G_DBUS_MESSAGE_TYPE_ERROR) {
    // TODO: show error to user
    g_dbus_message_to_gerror(reply_message, error);
    return NULL;
  }

  if (reply_type == G_DBUS_MESSAGE_TYPE_METHOD_RETURN) {
    // We got something!
    GVariant *response = g_dbus_message_get_body(reply_message);

    /**
     * We receive a tuple which contains required string.
     * Now, we fetch the first item and extract the string.
     */
    gsize index = {0};
    response = g_variant_get_child_value(response, index);

    if (response == NULL) {
      return NULL;
    }

    char *msg = g_variant_get_string(response, NULL);

    return msg;
  }

  return NULL;
}
```

As seen above, we call our `RewriteText` method with the user’s selected text. It returns a tuple (DBus type: `(s)`) from which we extract rewritten string.

**gtkentry.c:**

```c
static void
// If you've seen this function in part one, you'll
// notice the change in function arguments.
ghostllm_rewrite_text_cb (GtkEntry *entry)
{

  // Get selection bounds for string
  GtkEditable *editable = GTK_EDITABLE (entry);
  gint start, end;
  gtk_editable_get_selection_bounds (editable, &start, &end);

  // Get selected text from selection bounds
  gchar *input_str = _gtk_entry_get_display_text (entry, start, end);

  // Rewrite the input string
  char* response = ghostllm_rewrite_text(input_str);

  if (response == NULL) {
    g_printerr("Unable to generate GhostLLM response.");
    return;
  }

  // HACK: Delete user selected text using "Cut" operation
  gtk_entry_cut_clipboard(entry);

  // Insert rewritten text
  gtk_entry_insert_text(editable, response, strlen(response), &start);

  g_free (response);

}
```

In this step, we simply generate new text from our server, then copy the user’s selected input text to the clipboard and lastly insert this new text at the cursor position.

It might take a few seconds depending on your machine but in the end, it works. I noticed this still works even when the window is inactive.

# Demo

**User Input:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732902483309/b7d840d5-315e-4655-81d9-f87e0b452c1b.png align="center")

**Ollama response:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732902517162/f1fcd19f-5cf7-4270-aa5b-cfcfb6599999.png align="center")

As you can see, the output has changed. Now, this is just to demonstrate something like this is doable even when it isn’t practical.

> I was worried about the additional work involved in replacing the user selected text with the rewritten text but, as it turns out, I can cheat a little here using clipboard. :)

# Next Steps

The idea is to implement this feature to `GtkTextView` widget which was the original goal of this project. I will keep the `GtkEntry` changes as-is for my amusement.

# Conclusion

Thanks for your time!

This was a very fun project however I think the next part would mark the end for this. As always, the code will live on my GitHub.

I hope you liked this post. Please give it a Like to show your appreciation. If you feel there’s something I missed or can do better, let me know in the comments.

Bye for now :-)