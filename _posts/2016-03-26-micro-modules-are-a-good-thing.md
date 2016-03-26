---
layout: post
title: Micromodules are a good thing
date: 2016-03-26T10:35:43+0530
---
A few days ago, [Azer Koçulu unpublished all his modules from npm](https://medium.com/@azerbike/i-ve-just-liberated-my-modules-9045c06be67c#.x4pw3natp), including `left-pad` that was depended on by React and Babel, after npm transferred the package name "Kik" to Kik Interactive. That broke several builds around the world which led to several opinionated articles about Javascript development.

Many people seem to discourage depending on a package for such a trivial function. They encourage you to write it on your own. However, people are fine with depending on larger packages like Underscore.js etc.

Now, my whole life, I've been taught to reuse code. But now, people seem to say, reuse larger amounts of code. If it's trivial, just copy it. The rationale is good though: by adding a dependency, you, by definition are depending on something (or someone) else and you don't want to do that for such a small thing. The cost of including the dependency is greater than the cost of writing it on your own. Now, instead of `left-pad`, if it were a bigger package, like Underscore.js
itself, people wouldn't have asked you to write it on your own. They would have tried to fix the core problem of the package itself being taken down.

The reason people write very small modules like `left-pad` is because modules are cheap. But this incident told us they are not as cheap as everyone thought. The solution isn't to stop writing small modules. The solution would be to make modules cheaper again.
