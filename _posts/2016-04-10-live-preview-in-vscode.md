---
title: "Live Preview in VS Code"
excerpt: "A lightweight way to enable live preview in VS Code."
tags: [vscode]
---

I occassionally have to work on static HTML sites and for these I've been using VSCode rather than Visual Studio as it's so much faster to make a few small changes. I've done this with browser sync while doing Aurelia development but this is overkill for this situation. Until now I didn't have an easy way to see the changes I'm making without the usual 'F5 - change something - F5" loop.

[StackOverflow to the rescue](http://stackoverflow.com/questions/30039512/how-to-view-my-html-code-in-browser-with-visual-studio-code/34800879#34800879) you can do it like this...

As I'm on a Mac at home I tend to use Chrome most of the time. To set it up use **ctrl+shift+p** to find **Configure Task Runner**. change the parameters in **tasks.json**:

```json
{
    "version": "0.1.0",
    "command": "Chrome",
    "osx": {
        "command": "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
    },
    "args": [
        "${file}"
    ]
}
```

If you have chrome already open, it will launch your html file in a new tab.

You can also setup IE on windows like this

```json
{
    "version": "0.1.0",
    "command": "explorer",
    "windows": {
        "command": "explorer.exe"
    },
    "args": [
        "${file}"
    ]
}
```