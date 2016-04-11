---
layout: post
title: "Concatenating Lists in C#"
excerpt: "A quick example of how to use Concat."
tags: [c#]
share: true
comments: true
---

I came across a time today when I had to concatenate a number of lists of the same type. To do this you can use the `Concat` extension method from the `System.Linq` namespace.

As with most linq queries you can chain `Concat` together multiple times however it returns an `IEnumerable` type and it can be converted back into a List with the `ToList` method.

```c#
var Story = new List<string>();
var Chapter1 = new List<string>();
var Chapter2 = new List<string>();
var Chapter3 = new List<string>();

Story = Chapter1
  .Concat(Chapter2)
  .Concat(Chapter3)
  .ToList();
```

I'd still rather be able do something like this

```c#
var Story = new List<string>();
var Chapter1 = new List<string>();
var Chapter2 = new List<string>();
var Chapter3 = new List<string>();

Story = Concat(Chapter1, Chapter2, Chapter3);
```

But I'll stick with the first version as it's much simpler.