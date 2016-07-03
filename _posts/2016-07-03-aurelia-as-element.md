---
title: "Aurelia: As Element"
excerpt: ""
tags: [aurelia]
---

If you've done anything with tables then there's a good change you've come across the problem where it doesn't render quite right. So you try `@containerless` which works fine except when you want to use custom element. [Jason Sobell](http://www.sobell.net/dynamically-adding-row-templates-to-a-table-in-aurelia/) has found a nice solution, which is also part of aurelia.

By using the attribute `as-element` you can get your `compose` element to act like a different element.

I can see how this might be useful if you have something like below

```html
<custom-article as-element="article"></custom-article>
```

In this case you might want your article to be contained in an `article` tag, this gives you some flexibility as you may decide that you want to use an `aside` instead. You don't have to create a separate custom element you can just use `as-element`.