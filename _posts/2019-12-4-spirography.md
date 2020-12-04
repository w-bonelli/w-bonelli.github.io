---
layout: post
title: spirography
date: 2019-12-4
---

I stumbled across a cool problem presented [here](http://codyraskin.com/research/?p=158) by Cody Raskin. Here's his post in full:

> "Suppose you had a circle made of a hundred or so equidistant points on its circumference. If you drew straight lines connecting distant points on this circle (chords) in some systematic way, you’d produce a kind of spirograph. I don’t know what these plots are actually called, and “spirograph” is actually just the brand-name of a popular toy, but in any case, most people have probably drawn something like this at some point in their lives.

> The most common form of these that I’ve encountered are ones where the pattern for connecting the points is to simply connect every point <code>p<sub>i</sub></code> to <code>p<sub>i + k</sub></code> where <code>k</code> is some fixed offset. What if instead we connected <code>p<sub>i</sub></code> to <code>p<sub>ik</sub></code> where <code>k</code> is now a fixed multiplier?

> I used the Desmos graphing calculator to play around with different values of <code>k</code> in real time. I did not expect the “lobe” count to scale as <code>∝ k−1</code>.

> I wonder if someone could come up with some hand-wavy argument as to why this behaves this way? In any case, you can try it out at Desmos [here](https://www.desmos.com/calculator/yjayzmpgzr)."

I don't recall ever drawing one of these when I was a kid. To make up for it, I've tried to reason through the question with the help of [D3.js](https://d3js.org/).

If you haven't clicked on the [Desmos link](https://www.desmos.com/calculator/yjayzmpgzr) yet, fiddle with it and see if you can get a sense for how it works!

We start at the beginning, which is also the end. Where's that? The rightmost point on the graph: <code>p<sub>0</sub></code> (that is, <code>p<sub>i</sub></code> where <code>i = 0</code>). We increment <code>i</code> (the index of the current point) as we progress counter-clockwise around the perimeter of the circle. We calculate <code>j</code> (the index of the connected-to point) as a function of <code>i</code> and <code>k</code>.

The spirograph is [<code>modular</code>](https://en.wikipedia.org/wiki/Modular_arithmetic), like a clock:  <code>i</code> and <code>j</code> "wrap around" after some maximum value, back to <code>0</code>. In Cody's graph that value is <code>500</code>, so we say we're using arithmetic modulus <code>501</code>. This is equivalent, in this to context, to saying <code>n = 501</code>; that is, <code>0</code> and <code>501</code> are congruent, or interchangeable. So are <code>1</code> and <code>502</code>, <code>2</code> and <code>503</code>, and so on.

We can say such a congruence relation exists when ([per Wikipedia](https://en.wikipedia.org/wiki/Modular_arithmetic#Definition_of_congruence_relation)):

> <code>a</code> and <code>b</code> have the same remainder when divided by <code>n</code>.

It's as if we've cut a slice (of length <code>n</code>) out of the number line, curled it into a loop, and glued the ends together. If you take <code>n</code> steps from <code>a</code> and call the place you arrive <code>b</code>, you'll discover <code>b = a<code>.

Before the multiplier case, consider the offset, where each point <code>p<sub>i</sub></code> connects to point <code>p<sub>j = i + k</sub></code>. This produces some recognizable forms: the pentagram (<code>n = 5</code>, <code>k = 2</code>), for example.

The operative difference between offset and multiplier is the relationship between <code>i</code> and the minimum distance between <code>p<sub>i</sub></code> and <code>p<sub>j</sub></code> (as measured in either clockwise or counter-clockwise "hops" between adjacent points on the perimeter of the graph); let's call this quantity <code>d</code>.

In the offset case <code>d = k</code> up to <code>i = n / 2</code>, then <code>d = n - k</code> up to <code>i = n</code>. If we plotted <code>d</code> as a function of <code>i</code> or <code>j</code>, we'd get a horizontal line.

What about the multiplier case, where <code>p<sub>i</sub></code> connects to <code>p<sub>j = ik</sub></code>? When (in standard arithmetic) <code>j &ge; n</code>, we substitute for <code>j</code> the congruent integer <code>c</code> such that <code>0 &le; c &lt; n</code>, given by <code>j mod n</code>.

When it comes time to connect <code>p<sub>i</sub></code> to <code>p<sub>j</sub></code>, we find ourselves back at the same point! These self-referential spots deserve a deeper look.

When does <code>j = i = i</code>? When <code>i = 0</code>, to begin with. Hence the first step in constructing any spirograph: just connect the initial point to itself. When else? When <code>j = ik = n</code>, since <code>p<sub>n</sub> = p<sub>0</sub></code>.

After the initial self-connection step, each subsequent reflexive connection indicates that a *cycle* has been completed: that is, <code>p<sub>j</sub></code> has "caught up to" <code>p<sub>i</sub></code>. We'll always find <code>k - 1</code> cycles because <code>p<sub>j</sub></code> "travels" <code>k</code> times farther than <code>p<sub>i</sub></code>, and "laps" it <code>k - 1</code> times. If we were to plot <code>d</code> for the multiplier case, we would get something like a sine wave taking domain <code>[0, n - 1]</code> to range <code>[0, n / 2]</code>.

Intuitively, lobe count scales at <code>∝ k - 1</code> because each densely-shaded cycle is a kind of "shadow" directly opposite a sparsely shaded "lobe". "Shading" is sparser within each lobe and denser without because as you move away from a lobe center and nearer to a midpoint between lobes, each point's outgoing connection line departs further from the tangent and approaches a clean bisection of the circle. There, <code>p<sub>i</sub></code> is maximally distant from <code>p<sub>j</sub></code>: <code>d = n / 2</code> (remainder <code>1</code> when <code>n</code> is odd). Outgoing connections swing back toward the tangent as you move toward the center of the next lobe. Dense shading occurs in small "shadows" near bisection initial points (opposite "lobes") and forms lobe boundaries where chords "stack" tightly around the tangent line.

**TL;DR:** "Lobes" correspond to cycles in the spirograph. Each cycle is preceded by a bisection. Each bisection entails a small, densely-shaded "shadow" near its initial point, and a larger, sparsely-shaded "lobe" directly opposite (for small values of <code>k</code>, that is: patterns tend to recede into noise then re-emerge in more complex formulations as <code>k</code> increases).

Here's a little animation of the multiplier case (which, when <code>k = 2</code>, yields the Rebel Alliance's insignia). You can also increment or decrement <code>k</code>, then watch how many times the green dot (<code>p<sub>j</sub></code>) "laps" the red one (<code>p<sub>i</sub></code>).

<div class="btn-group" role="group">
<button type="button" class="btn btn-dark" onclick="decK()">&lt;</button>
<button id="k" type="button" disabled class="btn btn-dark">
</button><button type="button" class="btn btn-dark" onclick="incK()">&gt;</button>
</div>
<div class="container-fluid" style="width: 100%">
<svg id="multiplier" style="width: 100%; height: 500px"></svg>
</div>

<script src="https://d3js.org/d3.v4.min.js"></script>
<script>
var k = 2;  // multiplier
var r = 150; // radius

var svg = d3
  .select("svg")
  .attr("width", 2000)
  .style("height", 2000);

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
