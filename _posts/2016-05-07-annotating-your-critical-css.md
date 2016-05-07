---
title: "Annotating your critical css"
excerpt: "A good start for widgets in bootstrap"
tags: [performance]
---

I've been looking into ways to improve the speed of sites and using the [Critical CSS](https://css-tricks.com/annotating-critical-css/) method looks to be a good way to do it. There have been plenty times when I've look at the percentage of used styles from the stylesheet (chrome will give you this information) and it's usually really low. So trying to load on the essential styles inline first makes sense.

Like a lot of optimisation techniques I'd probably leave them until last because there will be an overhead.