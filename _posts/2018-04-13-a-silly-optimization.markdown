---
layout: post
title:  "A Silly Binaryen Optimization"
date:   2018-04-13 10:52:56 -0700
categories: binaryen
---
The [Binaryen](https://github.com/WebAssembly/binaryen) optimizer compiles wasm to better (smaller, faster) wasm. A lot of the [optimizations](https://github.com/WebAssembly/binaryen/tree/master/src/passes) it has are very specific to WebAssembly, which is maybe not surprising since wasm is an 'odd' compiler target in many ways. So many of the optimization tricks are also odd!

Here's a tiny recent example: wasm encodes constant integers using signed LEB32. As a result, negative numbers are *slightly* favored, sometimes taking one less bit. For example,

{% highlight js %}
x + 64
{% endhighlight %}

takes one more byte than

{% highlight js %}
x - (-64)
{% endhighlight %}

However, the same is not true for 65 or 63 or most other numbers -  this only happens on [very specific powers of 2](https://github.com/WebAssembly/binaryen/blob/master/src/passes/OptimizeInstructions.cpp#L1143)! The reason is because of how signed LEBs and [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) work. As you can see on that Wikipedia page's examples, in 3-bit integers, 2 is `010` and -2 is `110`. For 2, we must emit 3 bits in signed LEB format, since we must emit a sign bit, but for -2 we just need 2 bits. So signed values have an advantage on powers of two, and this actually matters when the extra bit for the positive integer causes an extra LEB byte to be emitted (each byte contains 7 bits of payload, so which powers matter can be computed from that).

To keep things simple, maybe we can ignore the specific values where this matters and just always encode negative integers? That is,
{% highlight js %}
x + c  =>  x - (-c)
x - c  =>  x + (-c)
{% endhighlight %}
for any positive integer `c`? But this has a downside: on typical code, addition is much more common than subtraction, and positive integers are much more common than negative ones. So if we did this, we'd be shrinking `c` by replacing it with `-c` but we'd be making the code less compressible by replacing a lot of additions with subtractions (note that we aren't replacing all the additions, since many are not on a constant). In fact, on real-world code the downside outweights the benefit here: we reduce uncompressed size but increase compressed size.

To avoid that problem, Binaryen only does this [on constants where it actually matters](https://github.com/WebAssembly/binaryen/blob/master/src/passes/OptimizeInstructions.cpp#L1143). Most of those have an addition before them that we replace by a subtraction, so there is still a downside to compression, but it is outweighted by us saving a byte, which helps enough that we reduce both the uncompressed and compressed size.

How much does this affect binary size? Here are the diffs on the size of [`tanks.wasm`](http://webassembly.org/demo/):

|                       | uncompressed | compressed      |
|-----------------------|:------------:|:---------------:|
| All integers          |       -2552  |    +11347       |
| Just where it matters |       -2552  |      -544       |
|                       |              |                 |

Doing this on all integers increases compressed size by a lot more than it decreases uncompressed size! But doing it only on the integers where it matters lets us decrease both.

