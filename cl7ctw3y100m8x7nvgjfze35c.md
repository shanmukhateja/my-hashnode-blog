## Lightweight alternative to Postman? Just use VSCode!

Hello,

I was looking for alternatives for Postman which didn't require installing yet another Electron app and found a handy tool that can become part of VSCode.

## Background

After using Postman for testing REST APIs over the years, I felt it became heavy with all new features it had received over the years. My usage of Postman remained exclusively to test REST APIs and as such I didn't need fancier cloud stuff or any other features it provides really. 

I remember when Google Chrome [deprecated](https://arstechnica.com/gadgets/2017/12/google-shuts-down-the-apps-section-of-the-chrome-web-store/) Chrome Apps in favor of PWA or Progressive Web Apps however many apps had to resort to Electron since PWA just wasn't enough.

This led me to a web search for "alternatives to Postman" on a weekend and I was astounded to learn about a VSCode extension that fits my (simpler) needs perfectly.

## Meet the contender

I found [Thunder Client](https://marketplace.visualstudio.com/items?itemName=rangav.vscode-thunder-client) extension for Visual Studio Code to be an excellent replacement for a simple REST API testing tool.

It's pretty lightweight, works just like Postman, supports import/export and didn't require installing as an another application.

## Installation

1. Open VSCode and open Extensions section.

2. Search for "thunder client" and install the extension as seen below.

![ss_tc2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661658933223/0_QpPOQdD.png align="center")

3. After installation, you should see a new tab with lightening bolt icon on the left. Click on it or alternatively press `Ctrl + Shift + R` on the keyboard.

![ss_tc3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661659043124/irIqGsgoY.png align="center")

## Let's make a GET request

Click on `New Request` on the top to get started with a request.

You should see a GET request set to /welcome endpoint on Thunder Client's website.

> This is seen every time you click on `New Request` button.

To initiate a request, click on "Send" button. You should be greeted with this response.

![ss_tc4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1661659302179/NIrxkB2X0.png align="center")

You can customize this request by updating `Auth`, check out `Headers` and `Body` sections.

Here's a Getting Started video from the official Thunder Client YouTube channel:

<iframe width="560" height="315" src="https://www.youtube.com/embed/NKZ0ahNbmak" title="YouTube video player" frameborder="0" style="width:100%" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> 

<sup>Credits: https://github.com/rangav/thunder-client-support</sup>

## Conclusion

I love the fact this tool is a VSCode extension and doesn't constitute another app to be installed on my machine.

Feel free to comment your thoughts below. You can also @ me on [Twitter](https://twitter.com/shanmukhateja94).

Thanks for reading! Have a great day!