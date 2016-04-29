---
title: "Giving up on typescript, for now"
excerpt: "A bit of a rant on importing modules in TypeScript"
tags: [typescript, webpack]
---

I've been trying to use TypeScript and Webpack in my latest [Aurelia](http://aurelia.io) side project but it has become a nightmare getting 3rd party libraries working with it. Typically it results in a `module not found` error which probably means I need a typings file so I 

```shell
typings search WidgetUtopia
```
Luckily it's there's one already done so I 

```shell
typings install WidgetUtopia --save
```
And that error is gone. It's worth noting if there wasn't one I'd have to create one myself to make the errors go away.

Finally I can import the ShinyThing

```javascript
import {ShinyThing} from 'WidgetUtopia';
```
But because there wasn't a default export defined I need to use

```javascript
import * as ShinyThing from 'WidgetUtopia';
```

and that was fine.

## The more unhappy path
But suppose you get the typings and it still doesn't find the module? Well that's where I got stuck. I know the module works in an ES2016 project but I've used it before with no problems. In this case I had to compromise on plugins I wanted to use because I just couldn't get them working.

It was especially frustrating when the compiler errors went away and I got runtime errors about things being undefined and not found instead.

## It's not worth the pain
I went through this for 3 or 4 different plugins and had to compromise on some because I just couldn't get them working. I could be missing something fundamental about how to do this but it was just painful. So I'm abandoning TypeScript for now, the benefits you get from it aren't worth the pain you need to go through.

It's entirely possible some of these issues might relate to WebPack which hasn't been without it's problems either. I can understand now why the Aurelia team are [developing their own CLI](http://blog.durandal.io/2016/04/21/aurelia-cli-debug-tools-and-microsoft-collaboration/).