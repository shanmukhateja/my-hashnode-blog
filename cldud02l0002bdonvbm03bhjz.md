# Do you know about the library that powers VSCode?

Hello,

If you use VSCode as *religiously* as I do, you'd agree how amazing a tool it is.

In this blog post, let's learn what makes the code editing aspects of VSCode to be this awesome.

## Monaco Editor:

[Monaco Editor](https://microsoft.github.io/monaco-editor/) is a JavaScript library that powers VSCode.

It comes with lots of features built-in such as:

1. Diff Editor
    
2. Syntax highlighting
    
3. Breadcrumbs navigation
    
4. Command Palette
    
5. Go To \* functionality (ex: Ctrl+Click)
    
    [and many more!!](https://code.visualstudio.com/docs/editor/editingevolved)
    

Acc. to Monaco's FAQ, it is generated from VS Code's source code with wrappers for certain functionalities that are unavailable when running inside a browser.

## Try it online

Open the Monaco Editor Playground and use the dropdown to load various examples provided by default and get to know Monaco a little more.

Link: [Playground](https://microsoft.github.io/monaco-editor/playground.html)

## Integrating into your project

Let's see how to set up Monaco in a vanilla HTML5 project.

> I highly recommend using any existing integrations for your UI framework for production environments.
> 
> Click [here](https://www.npmjs.com/search?q=monaco%20editor) to look for a suitable package for your project at NPM.

Let's create a project directory named `playground` and install `monaco-editor`.

```sh
# init NPM project
mkdir playground
cd playground
npm init -y

# install monaco-editor
npm install monaco-editor@latest
```

Initialize Monaco editor with a default `index.html` file inside the project.

```xml
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monaco Editor Demo</title>

    <link
			data-name="vs/editor/editor.main"
			rel="stylesheet"
			href="node_modules/monaco-editor/min/vs/editor/editor.main.css"
		/>
    <link href="./style.css" rel="stylesheet">
</head>

<body>
    <div id="container"></div>
</body>

<!-- A hack to let monaco "require" necessary packages -->
<script>
    var require = { paths: { vs: 'node_modules/monaco-editor/min/vs' } };
</script>

<script src="node_modules/monaco-editor/min/vs/loader.js"></script>
<script src="node_modules/monaco-editor/min/vs/editor/editor.main.nls.js"></script>
<script src="node_modules/monaco-editor/min/vs/editor/editor.main.js"></script>


<!-- A custom script to init monaco editor -->
<script src="./scripts.js"></script>

</html>
```

```css
/* style.css */
body, #container {
    height: 100vh; /* mandatory */
}
```

```javascript
// scripts.js

// Initialize monaco editor on init
document.addEventListener('DOMContentLoaded', function () {
    const container = document.getElementById('container');
    
    // Create new instance of editor
    const editor = monaco.editor.create(container, {
        // text to show inside editor
        value: "function hello() {\n\talert('Hello world!');\n}",
        // for better syntax highlight and auto-complete
        language: 'javascript',
    });
});
```

You should see something like this:

![Monaco editor in action](https://cdn.hashnode.com/res/hashnode/image/upload/v1675749084128/4b74a5fd-8901-494a-bd77-bc27d12e9863.png align="center")

If it didn't work, try running the code from a `http://` or `https://` context by using an HTTP server.  
You can often get away with `npx http-server` inside the project directory and then goto `http://localhost:8000` in your web browser.

#### Command Palette

We can launch Command Palette by either right-clicking anywhere inside Monaco editor and selecting it or by pressing `F1` on your keyboard.

![Monaco editor with Command Pallete open](https://cdn.hashnode.com/res/hashnode/image/upload/v1675749304065/154fa25c-7353-4863-b251-57afacd9395f.png align="center")

There's a lot more that can be done with Monaco which is difficult to put in this post. Feel free to explore more using the Playground.

## Conclusion

I hope you learned something new today. Consider giving this post a ❤ if you liked it.

You can also follow me on [Mastodon](http://social.linux.pizza/@shanmukhateja) so you don't miss any of my new posts.

Bye for now :-)