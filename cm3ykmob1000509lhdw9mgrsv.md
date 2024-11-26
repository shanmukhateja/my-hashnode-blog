---
title: "GhostLLM: Creating the DBus server with Ollama integration"
datePublished: Tue Nov 26 2024 14:47:37 GMT+0000 (Coordinated Universal Time)
cuid: cm3ykmob1000509lhdw9mgrsv
slug: ghostllm-creating-the-dbus-server-with-ollama-integration
tags: linux, python, llm, dbus, ollama

---

Hello,

This is part two of my GhostLLM project. This project is about integrating “Rewrite Text” functionality natively into GTK3 widgets. Check the [part one](https://hashnode.com/post/cm3x9jvve000609kz1t9tcq53) post if you missed it.

In this part, I will going over the DBus server code (written in Python) which will be exposing DBus interface on the Session Bus with a method “RewriteText” which accepts user input (String) and returns the rewritten sentence as the output.

# Implementation

The code for this project lives on GitHub [here](https://github.com/shanmukhateja/ghostllm-dbus/).

The main part here is to setup DBus interface. I used `dbus-fast` to simplify this.

**server.py:**

```python
# File: server.py

# DBus interface is declared here
# The `_` prefix is to indicate this is an internal class.
class _GhostLLMDBusInterface(ServiceInterface):

    def __init__(self):
        super().__init__(DBUS_ADDRESS_NAME)
        self.ollama = OllamaController()
    
    @method()
    def RewriteText(self, input_text: 's') -> 's':
        return self.ollama.chat(input_text)

# Wrapper class that simplifies executing this code from `__main__.py`
class GhostLLMDBus():

    async def init(self):
        # Connect to Session Bus
        bus = await MessageBus(bus_type=BusType.SESSION).connect()
        # Implement the DBus interface
        interface = _GhostLLMDBusInterface()
        bus.export(DBUS_INTERFACE_NAME, interface)
        await bus.request_name(DBUS_ADDRESS_NAME)

        print(f"DBUS Service initialized at {DBUS_INTERFACE_NAME}")

        await bus.wait_for_disconnect()

    # We call this in `__main__.py` that will kick things off!
    def listen(self):
        asyncio.run(self.init())
```

**ollama.py:**

```python
# File: ollama.py
from ollama import generate
from ollama import GenerateResponse

from .constants import OLLAMA_MODEL_NAME, OLLAMA_SYSTEM_PROMPT

class OllamaController():

    def chat(self, input_str: str) -> str:
        response: GenerateResponse = generate(
            model=OLLAMA_MODEL_NAME, # qwen2.5:3b
            system=OLLAMA_SYSTEM_PROMPT,
            stream=False,
            prompt= f'rewrite "{input_str}"'
        )

        return response.response
```

This code is pretty straight-forward. I used the usage code from [Ollama Python library](https://github.com/ollama/ollama-python). I use `/generate` endpoint to customize the prompt to fit the needs. The constants here could be replaced with a config file for easier access.

> You should always use ‘system’ role to set your instructions. This way, your users wouldn’t be able to change or override your instructions. That’s called a prompt injection attack. Learn more about it on [Wikipedia](https://en.wikipedia.org/wiki/Prompt_injection).

**\_\_main\_\_.py:**

```python
import sys

from .server import GhostLLMDBus

if __name__ == '__main__':
    try:
        gh = GhostLLMDBus()
        gh.listen()
    except KeyboardInterrupt:
        sys.exit(0)
```

I tried this code with LLAMA3.2 model but it didn’t respect the system prompt. It always tried to process the user prompt rather than simply rewriting the text. Hence, I went with **qwen2.5:3b** model with the following prompt:

```markdown
// SYSTEM PROMPT:
Your work is to rewrite sentences to make them more professional. Reply with 1 rewritten sentence. do not say anything else.

// USER PROMPT:
rewrite "xxx"
```

You run the project using:

```bash
$ cd <project-root> # like cd ~/Projects/ghostllm-dbus
# You need to run `pip install -r requirements.txt` first!!
$ python -m ghostllm-dbus
```

# Screenshot

I used [QTDBusViewer](https://doc.qt.io/qt-6/qdbusviewer.html) software to test the server. You can use any other software :)

**Input prompt:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732632025718/93141a8a-7459-462b-b65e-91d25c5ee16a.png align="center")

**Response:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732632041205/7a731da0-d1c9-4433-8cf6-021f72ddcda9.png align="center")

# Next Steps

The next part is going to be difficult. I need to figure out C code that will interact with DBus instance and present it to the user.

I will also need to figure out a way to replace the selected text in buffer with GhostLLM’s response.

# Conclusion

Thanks for reading! I would appreciate if you could leave a Like to this post.

Please leave a comment if you think I missed something :)

Bye for now :-)