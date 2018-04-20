---
layout: post
title:  "Small WebAssembly Binaries with Rust + Emscripten"
date:   2018-04-18 10:52:56 -0700
categories: binaryen
---

The Rust language is one of the earliest adopters of WebAssembly, and it has more than one way to compile to it:

 * `wasm32-unknown-unknown` which uses the LLVM WebAssembly backend directly to compile **dynamic libraries**.
 * `wasm32-unknown-emscripten` which uses Emscripten to compile **whole programs**.

`wasm32-unknown-unknown` makes sense if you have code that does some pure computation and you want to call it from JavaScript. For example, maybe you're writing some Rust code for your website that does math that benefits from the speed of WebAssembly. On the other hand, `wasm32-unknown-emscripten` makes sense if you have a complete program that uses APIs like libc, SDL, OpenGL, etc. For example, maybe you're writing a game in Rust and you want to compile that same codebase to both a native build and to the Web.

Both are valid use cases, but it's possible the Rust community is more interested in one than the other, which seems to be the case given all the excitement about `wasm32-unknown-unknown`. Part of that excitement is about code size: really small wasm binaries are possible that way. It makes sense those binaries would be smaller given that they are dynamic libraries, as opposed to complete programs that link in system support etc., so I didn't think much about this until I saw [this tweet](https://twitter.com/rickycodes/status/983558934056329216) which mentioned `.wasm` and `.js` sizes far, far larger than I'd expect! That led to some investigation that I'll summarize in this post, a lot of which surprised me. The good news, summarized at the end, is that it **is** possible to generate small wasm binaries using Rust + Emscripten, but as we'll see, it needs to be done carefully.

Initial Comparison
------------------

Consider this hello world program in C:

{% highlight c %}
#include <stdio.h>

int main() {
  printf("hello, world!\n");
  return 0;
}
{% endhighlight %}

`emcc -Os` compiles it to 2,389 bytes of WebAssembly (mostly libc stdio stuff) and 21K of JavaScript (which can load that wasm and run it in various environments with various options etc. - you can also [replace it with your own](https://github.com/kripken/emscripten/wiki/WebAssembly-Standalone)). And here is a naive hello world in Rust, as shown in the [Rust tutorial](https://doc.rust-lang.org/book/second-edition/ch01-02-hello-world.html):

{% highlight rust %}
fn main() {
    println!("Hello, world!");
}
{% endhighlight %}

We can build it by getting [rustup](https://rustup.rs/) and doing this:

{% highlight shell %}
# setup - only needed once
rustup default nightly
rustup target add wasm32-unknown-emscripten
# compile the file
rustc --target=wasm32-unknown-emscripten hello.rs -o hello.js -C opt-level=s
{% endhighlight %}

The result is 116K of WebAssembly and 109K of JavaScript. Much larger than C - very surprising! Clearly something is going on here that we need to understand.

Rust and Code Size
------------------

The Rust FAQ has an [entry on code size](https://www.rust-lang.org/en-US/faq.html#why-do-rust-programs-have-larger-binary-sizes-than-C-programs), which states:

> There are several factors that contribute to Rust programs having, by default, larger binary sizes than functionally-equivalent C programs. In general, Rust’s preference is to optimize for the performance of real-world programs, not the size of small programs.

And indeed, compiling that same hello world program natively (using `rustc hello.rs  -C opt-level=s` and `strip -g hello`) leads to 532K, which is in the same ballpark as the Web build of the Rust code. Compiling the C code natively (using `gcc -Os`) leads to 8K, which again, is in the same ballpark as the Web build of the C code. So in a sense the issue here is not the Web nor Emscripten, rather it's more a general difference between Rust and C.

Perhaps, then, we shouldn't expect to compile Rust programs using C APIs like SDL or OpenAL and get small code sizes, on the Web or otherwise? But let's keep going and see how much we can improve on the numbers we saw before.

Making the Rust More Like C
---------------------------

The Rust FAQ mentions that we can write Rust that uses C APIs like C does, avoiding `println!` and just directly calling `printf`. We can also use the `start` feature in order to implement the C `main()` ourselves, avoiding the extra runtime support Rust would otherwise generate. Here is what that looks like:

{% highlight rust %}
#![feature(libc)]
#![feature(start)]

extern "C" {
    fn printf(fmt: *const u8, ...) -> i32;
}

#[start]
fn start(_argc: isize, _argv: *const *const u8) -> isize {
    unsafe {
        printf(b"hello, world!\n\0".as_ptr());
        0
    }
}
{% endhighlight %}

Also, we can compile with `-C panic=abort` to avoid Rust trying to handle panics with nice stack traces, which would bring in a bunch of code. With these changes, things improve a lot! Rust's `.js` is basically the same size as C's `.js`, and Rust's `.wasm` is 13K, which is almost 10x smaller than before. It's still 5x larger than C's `.wasm`, though, so let's keep going.

C API Optimizations
-------------------

It turns out that the remaining difference is because `clang` will optimize a call to `printf` into `puts` when possible. But due to a [current issue on Rust nightly builds](https://github.com/rust-lang/rust/issues/50035#issuecomment-382105213) that isn't happening. We can call `puts` ourselves, though, by changing the 2 relevant lines:

{% highlight rust %}
#![feature(libc)]
#![feature(start)]

extern "C" {
    fn puts(fmt: *const u8) -> i32;
}

#[start]
fn start(_argc: isize, _argv: *const *const u8) -> isize {
    unsafe {
        puts(b"hello, world!\0".as_ptr());
        0
    }
}
{% endhighlight %}

Compiling this, Rust's `.wasm` file is 2,416 bytes - basically the same size as C's `.wasm`!

Conclusion
----------

It is possible to use Emscripten to compile Rust programs using C APIs to small WebAssembly binaries. They can be just as small as corresponding C programs, in fact, and those can be [quite compact](https://hacks.mozilla.org/2018/01/shrinking-webassembly-and-javascript-code-sizes-in-emscripten/). But while that's a nice result, we did have to make some changes to our Rust to get there:

 * **Call the `puts` C API directly**, to avoid Rust's `println!()` overhead (for more info, see the [Rust-wasm notes on string operations causing bloat](https://rust-lang-nursery.github.io/rust-wasm/game-of-life/code-size.html#avoid-string-formatting)).
 * **Use the `start` feature**, to avoid Rust's `main()` overhead.
 * **Compile with `-C panic=abort`**, to avoid Rust's stack trace overhead.

Overall, the main price we had to pay to get a small `.wasm` file was to avoid idiomatic Rust code like `println!()`. It's disappointing we can't use Rust in the most natural way and expect tiny code sizes. However, it wouldn't be fair to criticize Rust on this. For one thing, idiomatic C++ using `std::iostream` etc. can also be much larger than C. And for that matter, even simple C code may be much larger than JavaScript, for example if it brings in a few K of malloc. And it's not easy to work around this stuff - you can write a more [compact](https://github.com/kripken/emscripten/pull/6249) [malloc](https://github.com/rustwasm/wee_alloc), but it comes at the price of performance.

Perhaps the fundamental issue here is that JavaScript is the only language for which the Web runtime is a perfect fit. Close relatives that were designed to compile to it, like TypeScript, can be very efficient as well. But languages like C, C++, Rust, and so forth were not originally designed for that purpose. So it is not surprising we hit inefficiencies when compiling them to the Web - maybe what should surprise us is how well things work despite some inefficiency!

In addition, we may be able to do better here. For example, string conversions between JavaScript and WebAssembly may be helped by the [host bindings proposal](https://github.com/WebAssembly/host-bindings/blob/master/proposals/host-bindings/Overview.md) - but if your language's strings differ from JavaScript's, maybe not (see also: [IronPython's strings are different than Python's](https://nedbatchelder.com/blog/201703/ironpython_is_weird.html)). And languages that need to ship a garbage collector may be able to avoid that and [use the host's GC](https://github.com/WebAssembly/gc/blob/master/proposals/gc/Overview.md) - but if you need different GC features, maybe not, and also that won't help languages that need `malloc`. Overall, I expect things to improve, but also we'll keep seeing issues like these as WebAssembly adoption increases.

<br>
<hr>
<br>

Acknowledgements
================

Thanks to acrichto, fitzgen, and sunfish for help with figuring this stuff out!

