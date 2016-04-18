---
title: "[Crosspost] Jumping into C# 6"
modified:
excerpt: "A few of my favourite parts of C# 6."
tags: ["c#6"]
---

> Originally published on [mttmccb.net](http://mttmccb.net/blog/2016/jumping-into-c-6)

I watched a few videos on [Pluralsight][0], I found [Exporing C# 6 with Jon Skeet][1] a very good way to get an insights into the new features. On top of that [Play by Play: C# Q&A with Scott Allen and Jon Skeet][2] was very good to provide some more complex examples.

## Taking a leap

I decided to try using it on the project I'm working on to see what the experience was like. Because I wanted to use it in production too I installed [.net 4.6.1][3] on the server, it needed a reboot but luckily I managed to fit that in with other updates I needed to do.

I later found out you [probably don't need to get the latest version][4].

## Top Features

It would be pretty easy to pick string interpolation as my favourite when you can do

[0]: http://pluralsight.com
[1]: http://www.pluralsight.com/courses/csharp-with-jon-skeet
[2]: http://www.pluralsight.com/courses/play-by-play-csharp-q-and-a-with-scott-allen-and-jon-skeet
[3]: https://www.microsoft.com/en-gb/download/details.aspx?id=48130
[4]: http://stackoverflow.com/questions/28921701/does-c-sharp-6-0-work-for-net-4-0

```c#
$"{name} was born on {dob:dd mmm yyyy}"
```

instead of

```c#
String.Format("{0} was born on {1:dd mmm yyyy}", name, dob)
```

but I don't think that's it.

I'm really liking `using static` so I can remove a lot of cruft which are essentially references to the same static class over and over. You can use it with library class like `System.Math`, your own static classes and even Enums.

String interpolation reminds me a lot of the new features in JavaScript ES6 but following on from that is expression body members. It's a rubbish name and I don't fully understand what it means expect it means that this

```c#
public decimal MoneyEarned { get { return MonthlyWage * 12 - Tax; } }
```

becomes

```c#
public decimal MoneyEarned => MonthlyWage * 12 - Tax;
```

While not on the same scale as string interpolation (although I have found situations where I am combining both) it certain makes it a lot simpler to understand.

The final one is `nameof`, I didn't fully understand this at first and managed to bodge a bit refactoring while trying to use it but it will essentially get rid of strings which refer to field name or variable names.

I'm using it when I need to get a property name which maps to stored procedure parameter for example instead of 

```c#
p.AddParameter("@Name", person.Name)
```

I can get rid of the string and do

```c#
p.AddParameter(@"@{nameof(Name)}", person.Name)
```

It's a pretty verbose example but what this mean is it I refactor the Name property then I don't have to update the variable, in the first one my renaming of Name would not rename the variable (in some case this might be fine).

## Go for it..

It was really easy to get started with C# 6 and use it in production, if you're able to then just start using it. You might find Dustin Campbell's [C# Essentials extension][5] useful as it will highlight some changes to use the new syntax in Visual Studio. It can also help you do the refactorings with a few clicks.

I'm sure there will be more to share but after looking into it for a few days those are the immediate highlights.

[5]: https://visualstudiogallery.msdn.microsoft.com/a4445ad0-f97c-41f9-a148-eae225dcc8a5