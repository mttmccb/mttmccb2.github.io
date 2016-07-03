---
title: "Setter Callback in Aurelia"
excerpt: ""
tags: [aurelia, design-patterns, observer]
---

Full credit for this technique goes to [@atsu](https://github.com/atsu85) and [@axwalker](https://github.com/axwalker/aurelia-observer-patterns/wiki/Observer-patterns-in-Aurelia).

Design patterns are great but sometimes they can be a little bit abstract. Episode 42 of [coding blocks](http://www.codingblocks.net/podcast/episode-42-command-repository-and-mediator-design-patterns/) and their attempts to explain the command and mediator patterns were good examples of that.

I came across a situation where I was struggling to handle deleting an object from an array without giving the object too much knowledge of the array it's part of.

That's still too abstract, we need an example.

## Lemmings!

![Lemmings Box]({{ site.url }}{{ site.baseurl }}/images/Lemmings-BoxScan.jpg){: .align-right}

If you've never played the classic puzzle-platform game you should find it and play it. It's great fun.
The main premise of the game is that you're trying to save lemmings who will gladly walk to their death. It was of course a [myth](http://io9.gizmodo.com/lemming-suicide-is-a-myth-that-was-perpetuated-by-disne-1549040246).

You job was to save then getting them to build bridges, dig tunnels and blow up a few of them along the way to get them to the level exit. Sounds harsh but you rarely had to save them all to complete the level.

To start the game let's setup a view model for the lemmings and a lemming custom element.

### lemmings.js
```javascript
export class Lemmings {
    constructor() {
        this.lemmings = [];
    }
    
    activate() {
        setInterval(() => this.newLemming(), 1000 );
    }

    newLemming() {
        this.lemmings.push({ name: randomName()});
    }
}
```

### lemmings.html
```html
<template>
    <require from="lemming"></require>

    <div repeat.for="lemming of lemmings">
        <lemming model.bind="lemming"></lemming>
    </div>
</template>
```

### lemming.js
```javascript
import {bindable} from 'aurelia-framework';

class LemmingCustomElement {
    @bindable model;
    fallToDeath() {
        // Delete from lemmings array
    }
}
```

### lemming.html
```html
<template>
    <h2>${model.name}</h2>
</template>
```

## Starting the carnage
When the game starts a new lemming falls out the level entrance every second so we've setup a timer to add a lemming to the lemmings array when the `activate()` method is called.

![Lemmings Level]({{ site.url }}/images/lemmingslevel.jpg)

The problem here is when our lemming falls to it's death we want it delete itself from the array of lemmings. In the level shown above what you need to do it get your diggers to dig through the floor but if your lemmings fall too far they die, which is why we have the `fallToDeath` method. This is a fairly tame and small level but it's surprisingly difficult.

## Using EventAggregator
My first instinct was to use the EventAggregator, a lightweight pub/sub messaging system, to signal the event and then take the action, but this seemed a bit heavy-handed and after some discussions in the [Aurelia Gitter]() it became clear I had been overusing it in the past. It's great when the channel between the event and the listener is quite far apart and it wouldn't make sense to bind them together.

To get that setup it might look something like the updated code below.

### lemmings.html (updated)
```javascript
import { EventAggregator } from 'aurelia-event-aggregator';

@inject(EventAggregator)
export class Lemmings {
    constructor(EventAggregator) {
        this.lemmings = [];
        this.ea = EventAggregator;
        this.lemmingUndetaker = this.ea.subscribe('lemmingDied', response => this.buryLemming(lemming)); 
    }

    buryLemming(lemming) {
        const index = this.lemmings.indexOf(lemming);
        this.lemmings.splice(index, 1);
    }

    newLemming() {
        this.lemmings.push(new Lemming());
    }

    deactivate() {
        this.lemmingUndetaker.dispose();
    }
}
```

### lemming.html (updated)
```javascript
@inject(EventAggregator)
class Lemming {
    constructor(EventAggregator) {
        this.ea = EventAggregator;
    }

    fallToDeath() {
        this.ea.publish('lemmingDied', this);
    }
}
```

In this example you have got some loosely coupled objects but there's a bit of a overhead involved to do very little.

## Passing the callback function reference
Another option is passing the setter/callback function reference from parent to child component using `.call` binding. Then the reference to the callback function that is defined in parent will be stored on child component and it can be used as normal function with the parameters specified by the function declaration in parent component.

### lemmings.html (updated)
```html
<template>
    <require from="lemming"></require>

    <div repeat.for="lemming of lemmings">
        <lemming model.bind="lemming" bury-lemming-callback.call="buryLemming(lemming)"></lemming>
    </div>
</template>
```

### lemming.js (updated)
```javascript
import {bindable} from 'aurelia-framework';

class LemmingCustomElement {
    @bindable model;
    @bindable buryLemmingCallback;

    fallToDeath(lemming) {
        this.buryLemmingCallback(lemming);
    }
}
```

## Summary
I hope you can see there are interesting ways to make a child object able to interact with it's parent without too much intrusive code.