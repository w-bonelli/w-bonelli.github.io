---
layout: post
title: spirography
date: 2019-12-4
---

I stumbled across a cool problem presented [here](http://codyraskin.com/research/?p=158) by Cody Raskin. His post is short, so I'll reproduce it in full:

> "Suppose you had a circle made of a hundred or so equidistant points on its circumference. If you drew straight lines connecting distant points on this circle (chords) in some systematic way, you’d produce a kind of spirograph. I don’t know what these plots are actually called, and “spirograph” is actually just the brand-name of a popular toy, but in any case, most people have probably drawn something like this at some point in their lives.

> The most common form of these that I’ve encountered are ones where the pattern for connecting the points is to simply connect every point *p<sub>i</sub>* to *p<sub>i + k</sub>* where *k* is some fixed offset. What if instead we connected *p<sub>i</sub>* to *p<sub>ik</sub>* where *k* is now a fixed multiplier?

> I used the Desmos graphing calculator to play around with different values of *k* in real time. I did not expect the “lobe” count to scale as *∝ k−1*.

> I wonder if someone could come up with some hand-wavy argument as to why this behaves this way? In any case, you can try it out at Desmos [here](https://www.desmos.com/calculator/yjayzmpgzr)."

These things are kind of beautiful. I don't recall ever drawing one when I was a kid. To make up for it, I've tried to reason through the question with the help of [D3.js](https://d3js.org/). (Let me know if I got anything wrong, please!)

If you haven't clicked on the [Desmos link](https://www.desmos.com/calculator/yjayzmpgzr) yet, I encourage you to fiddle with it and see if you can get a sense for how it works.

---

Let's start at the beginning, which also happens to be the end. Where's that? The rightmost point on the graph. We can refer to it as *p<sub>0</sub>* (that is, *p<sub>i</sub>* where *i* = 0). We increment *i* (the "index" of the current point) as we progress counter-clockwise around the perimeter of the circle and calculate *j* (the "index" of the connected-to point) as a function of *i* and *k*. The spirograph is [*modular*](https://en.wikipedia.org/wiki/Modular_arithmetic), like a clock:  *i* and *j* "wrap around" after some maximum value, back to 0. In the graph Cody has given us that value is 500, so we say we're using arithmetic modulus 501, or *n* = 501. This means 0 and 501 are *congruent*, or interchangeable, in this system. So are 1 and 502, 2 and 503, and so on.

This is called a *congruence relation*, and we can say it exists when ([per Wikipedia](https://en.wikipedia.org/wiki/Modular_arithmetic#Definition_of_congruence_relation)):

> a and b have the same remainder when divided by n.

It's as if we've cut a slice (of length *n*) out of the number line, curled it into a loop, and glued the ends together. If you take *n* steps from *a* and call the place you arrive *b*, you'll discover *b* is actually *a*.

Before considering the multiplier case, let's think about the offset case: each point *p<sub>i</sub>* connects to point *p<sub>j = i + k</sub>*. This pattern produces some recognizable forms: the pentagram (*n* = 5, *k* = 2), for example. (Grab a piece of paper and try it out!)

The operative difference between the offset case and the multiplier case is the relationship between two values: *i*, and the minimum distance between *p<sub>i</sub>* and *p<sub>j</sub>* (as measured in either clockwise or counter-clockwise "hops" between adjacent points on the perimeter of the graph). Let's call this distance *d*. In the offset case we know by definition that *d* = *k*. If we plotted  *d* as a function of *i* or *j*, we'd get a horizontal line at *k*.

Now consider the multiplier case, wherein *p<sub>i</sub>* connects to *p<sub>j = ik</sub>*. Thus *d* is no longer constantly *k*, but *j*. There's a catch, though: we've got to observe the rules of modular arithmetic. Points on our graph are only numbered from 0 to *n* - 1, after all. So what happens if (in standard arithmetic) *j* &ge; *n*? We substitute for it the *congruent* integer *c* such that 0 &le; *c* &lt; *n*. How do we find it? Easy: just take *j* mod *n*.

Wait a second. What happens when we take *p<sub>i</sub>* and try to connect it to point *p<sub>j</sub>*, but find that *j* = *i*? We can try to draw a line from a point to the very same point, but if we're sufficiently precise, we won't get very far! These strange, self-referential spots deserve a deeper look.

When does *j* = *ik* = *i*? When *i* = 0, to begin with. Hence the very first step in constructing any spirograph according to this sort of pattern: just connect the initial point to itself. When else? When *j* = *ik* = *n*, since *p<sub>n</sub>* *is* *p<sub>0</sub>* (this is equivalent to the example above, where we started from *a*, took *n* steps, and ended up back at *a*).

Points which connect to themselves (save for the initial point) indicate that a *cycle* has been completed: that is, *p<sub>j</sub>* has "caught up to" *p<sub>i</sub>*. One will always find *k* - 1 cycles because *p<sub>j</sub>* ultimately "travels" *k* times farther than *p<sub>i</sub>*, and must "lap" it *k* - 1 times. If we were to plot *d* for the multiplier case, we would get something like a sine wave with domain [0, *n* - 1] and range [0, *n* / 2]. Intuitively, lobe count scales at *k* - 1 because each darkly-shaded cycle is a kind of "shadow" directly opposite a lightly shaded "lobe". "Shading" is lighter within each lobe and darker without because as you move away from a lobe center and nearer to a midpoint between lobes, each point's outgoing connection line departs further from the tangent and approaches orthogonality: that is, the line bisects the circle. At bisection, *p<sub>i</sub>* is maximally distant from *p<sub>j</sub>*: *d* = *n* / 2 (remainder 1 when *n* is odd). Outgoing connections swing back toward the tangent as you move toward the center of the next lobe. Dark shading occurs in small "shadows" near bisection initial points (opposite "lobes") and forms lobe boundaries where chords "stack" tightly around the tangent line. Here's what that looks like for  *k* = 2 and *k* = 4 (*p<sub>i</sub>* is red and *p<sub>j</sub>* is green):

<p class="codepen" data-height="265" data-theme-id="dark" data-default-tab="js,result" data-user="w-bonelli" data-slug-hash="NWPPgrm" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Spiro_k2">
  <span>See the Pen <a href="https://codepen.io/w-bonelli/pen/NWPPgrm">
  Spiro_k2</a> by w-bonelli (<a href="https://codepen.io/w-bonelli">@w-bonelli</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

<p class="codepen" data-height="265" data-theme-id="dark" data-default-tab="js,result" data-user="w-bonelli" data-slug-hash="eYmmRZp" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Spiro_k4">
  <span>See the Pen <a href="https://codepen.io/w-bonelli/pen/eYmmRZp">
  Spiro_k4</a> by w-bonelli (<a href="https://codepen.io/w-bonelli">@w-bonelli</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

---

**TL;DR:** "Lobes" correspond to cycles in the spirograph. Each cycle is preceded by a bisection. Each bisection entails a small, darkly-shaded "shadow" near its initial point, and a larger, lightly-shaded "lobe" directly opposite (for small values of *k*, that is: patterns tend to recede into noise then re-emerge in more complex formulations as *k* increases).

Now my lobes are tired. That's it for this post. Thanks again to Cody for providing the basis for it.
