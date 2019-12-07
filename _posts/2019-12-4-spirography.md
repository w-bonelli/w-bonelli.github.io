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

The operative difference between the offset case and the multiplier case is the relationship between two values: *i*, and the minimum distance between *p<sub>i</sub>* and *p<sub>j</sub>* (as measured in either clockwise or counter-clockwise "hops" between adjacent points on the perimeter of the graph). Let's call this distance *d*. In the offset case *d* is *k* up to *i* = *n* / 2, then *n* - *k* up to *i* = *n*. If we plotted *d* as a function of *i* or *j*, we'd get a horizontal line.

Now consider the multiplier case, wherein *p<sub>i</sub>* connects to *p<sub>j = ik</sub>*. Points on our graph are only numbered from 0 to *n* - 1, remember. So what happens if (in standard arithmetic) *j* &ge; *n*? We substitute for *j* the *congruent* integer *c* such that 0 &le; *c* &lt; *n*. How do we find it? Easy: just take *j* mod *n*.

What happens when we take *p<sub>i</sub>* and try to connect it to point *p<sub>j</sub>*, but find that *j* = *i*? We draw a line from a point to the very same point! These strange, self-referential spots deserve a deeper look.

When does *j* = *ik* = *i*? When *i* = 0, to begin with. Hence the very first step in constructing any spirograph according to this sort of pattern: just connect the initial point to itself. When else? When *j* = *ik* = *n*, since *p<sub>n</sub>* *is* *p<sub>0</sub>* (this is equivalent to the example above, where we started from *a*, took *n* steps, and ended up back at *a*).

Points which connect to themselves (save for the initial point) indicate that a *cycle* has been completed: that is, *p<sub>j</sub>* has "caught up to" *p<sub>i</sub>*. One will always find *k* - 1 cycles because *p<sub>j</sub>* ultimately "travels" *k* times farther than *p<sub>i</sub>*, and must "lap" it *k* - 1 times. If we were to plot *d* for the multiplier case, we would get something like a sine wave with domain [0, *n* - 1] and range [0, *n* / 2]. Intuitively, lobe count scales at *∝* *k* - 1 because each darkly-shaded cycle is a kind of "shadow" directly opposite a lightly shaded "lobe". "Shading" is lighter within each lobe and darker without because as you move away from a lobe center and nearer to a midpoint between lobes, each point's outgoing connection line departs further from the tangent and approaches orthogonality: that is, the line bisects the circle. At bisection, *p<sub>i</sub>* is maximally distant from *p<sub>j</sub>*: *d* = *n* / 2 (remainder 1 when *n* is odd). Outgoing connections swing back toward the tangent as you move toward the center of the next lobe. Dark shading occurs in small "shadows" near bisection initial points (opposite "lobes") and forms lobe boundaries where chords "stack" tightly around the tangent line.

Here's a little animation of the multiplier case, constructed in real-time. You can increment or decrement *k*, then watch how many times the green dot (*p<sub>j</sub>*) "laps" the red one (*p<sub>j</sub>*):

<div class="btn-group" role="group">
<button type="button" class="btn btn-dark" onclick="decK()">&lt;</button>
<button id="k" type="button" disabled class="btn btn-dark">
</button><button type="button" class="btn btn-dark" onclick="incK()">&gt;</button>
</div>
<svg id="multiplier" class="container-fluid"></svg>

**TL;DR:** "Lobes" correspond to cycles in the spirograph. Each cycle is preceded by a bisection. Each bisection entails a small, darkly-shaded "shadow" near its initial point, and a larger, lightly-shaded "lobe" directly opposite (for small values of *k*, that is: patterns tend to recede into noise then re-emerge in more complex formulations as *k* increases). Cool, right?

<script src="https://d3js.org/d3.v4.min.js"></script>
<script>
var k = 2;  // multiplier
var r = 40; // radius

var svg = d3
  .select("svg")
  .attr("width", 960)
  .style("height", 600);

var path = svg
  .append("path")
  .attr(
    "d",
    "M " +
      3 * r +
      ", " +
      2 * r +
      " a " +
      r +
      "," +
      r +
      " 0 1,1 " +
      -2 * r +
      ",0 a " +
      r +
      "," +
      r +
      " 0 1,1 " +
      2 * r +
      ",0"
  )
  .attr("fill", "none")
  .style("stroke", "gray")
  .style("stroke-width", "3")
  .style("stroke-opacity", 0.5);

var piDot = svg
  .append("circle")
  .attr("fill", "red")
  .style("fill-opacity", 0.5)
  .attr("r", 6)
  .attr("transform", "translate(" + 2 * r + "," + 3 * r + ")");

var pjDot = svg
  .append("circle")
  .attr("fill", "green")
  .style("fill-opacity", 0.5)
  .attr("r", 6)
  .attr("transform", "translate(" + 2 * r + "," + 3 * r + ")");

renderK();
transition();

function incK() {
    k += 1;
    renderK();
}

function decK() {
    if (k > 1) { 
        k -= 1;
    }
    renderK();
}

function renderK() {
    document.getElementById("k").innerHTML = "k = " + k;
}

function transition() {
  piDot
    .transition()
    .duration(5000)
    .attrTween("transform", translateAlong(path.node(), k))
    .on("end", function() {
      d3.selectAll("line").remove();
      transition();
    });
}

// adapted from https://bl.ocks.org/mbostock/1705868
function translateAlong(path, K) {
  var l = path.getTotalLength();
  return function(d, i, a) {
    return function(t) {
      var i = t * l;
      var j = i * K;
      var pi = path.getPointAtLength(i);
      var pj = path.getPointAtLength(j <= l ? j : j % l);
      svg
        .append("line")
        .attr("x1", pi.x)
        .attr("y1", pi.y)
        .attr("x2", pj.x)
        .attr("y2", pj.y)
        .style("stroke", "gray")
        .style("stroke-opacity", 0.25)
        .style("stroke-width", 1);
      pjDot.attr("transform", "translate(" + pj.x + "," + pj.y + ")");
      return "translate(" + pi.x + "," + pi.y + ")";
    };
  };
}

</script>
