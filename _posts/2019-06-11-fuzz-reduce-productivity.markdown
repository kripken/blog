---
layout: post
title:  "Fuzzers & Reducers as Productivity Tools"
date:   2019-06-11 10:52:56 -0700
categories: binaryen
---

Fuzzers generate random testcases in order to find interesting ones, like an input that causes a crash, and reducers take interesting testcases and find equally-interesting ones which are smaller and easier to debug. Fuzzers in particular are extremely important to modern software security, with massive amounts of fuzzing going into hardening important codebases like web browsers. But aside from background and introduction, this post is **NOT** about that stuff! Instead, it'll talk about how fuzzers and reducers can be good for **developer productivity**, that is, helping programmers - most obviously compiler developers, but maybe others. Specifically, I'll talk about how I use the [Binaryen](https://github.com/WebAssembly/binaryen/) fuzzer and reducer.

# The Binaryen Fuzzer

Binaryen is a toolkit for [WebAssembly](https://webassembly.org/) compilers. It has tools like `wasm-opt` which optimizes WebAssembly and `wasm2js` which compiles WebAssembly to JavaScript, and includes fuzzing and reducing as well. Specifically, the fuzzer is a parameter to to `wasm-opt`,

```shell
$ wasm-opt -ttf input.dat -o output.wasm
```

`-ttf` is short for "translate to fuzz testcase". It makes `wasm-opt` treat the input as the input to the internal fuzzer, which means that whatever is in `input.dat` will lead in some mysterious-but-deterministic manner to a valid WebAssembly module. We can then save that to `output.wasm` as in that command. We can also run some passes on it (that's what `wasm-opt` is for),

```shell
$ wasm-opt -ttf input.dat -Os --print
```

That will optimize the wasm (for size) and then print it out as text.

`-ttf` is not a complete fuzzer, but it's all you need to create a simple one: just create files containing random data and read those, for example:

```shell
$ cat /dev/urandom | head -c 200 > input.dat
$ bin/wasm-opt -ttf input.dat -o output.wasm
```

We can then use the wasm just created in various ways. For example, we can run it on a VM to check for a crash. A more interesting way to fuzz is to let a tool like [afl-fuzz](http://lcamtuf.coredump.cx/afl/) drive things, creating the random inputs and deciding which to pursue. Since wasm-opt deterministically connects the random input to the output wasm, such tools can be very effective. You can of course just use afl-fuzz without Binaryen, and the two approaches are complementary: Binaryen's `-ttf` mode is guaranteed to emit a valid wasm, so it's nice for fuzzing optimizations, while afl-fuzz without Binaryen will mostly emit wasm binaries that fail to validate, so it's nice for fuzzing parsing.

As mentioned earlier, fuzzers are very useful at finding compiler bugs. Binaryen has a [driver script](https://github.com/WebAssembly/binaryen/blob/master/scripts/fuzz_opt.py) that can check for wasm VM crashes, compare outputs between wasm VMs, and compare outputs before and after Binaryen optimizations. This has helped to find a bunch of bugs in Binaryen itself and in wasm VMs.

# The Binaryen Reducer

`wasm-reduce` is Binaryen's reducer. An obvious use case for it is reducing crash bugs. For example, imagine that `bad.wasm` is very large and that `wasm-opt bad.wasm -Os` crashes. We can reduce it using this:

```shell
$ bin/wasm-reduce bad.wasm '--command=bin/wasm-opt t.wasm -Os' -t t.wasm -w w.wasm
```

What this does is start from a given "interesting" wasm file, here `bad.wasm`. It then tries to find a smaller wasm on which the command `bin/wasm-opt t.wasm -Os` has the same result (same output and same return code), which it assumes means the new wasm file is equally "interesting". Note that the command runs on `t.wasm`, which we define by `-t t.wasm` as the "test file" (so that we don't modify the input file). There is also the "working file", here `w.wasm`, which is the current reduction (you can inspect it as the reducer runs if you get bored, and it's where the final output will be).

In an example of this, [a 45 MB testcase](https://github.com/WebAssembly/binaryen/issues/1838#issuecomment-470037568) was provided which crashed the optimizer at `-O3`. On such a massive file, and at the very highest optimization level, just running the optimizer takes 40 seconds. Instead, I ran wasm-reduce on it and after an hour it produced a tiny 235 byte testcase, which let me understand and fix the bug in just a few minutes. The reduced testcase itself could even be reused as a [testcase](https://github.com/WebAssembly/binaryen/commit/1a5b410701542413f497b3030c0b87f3600dc1bc#diff-93ee8bf71b92ab18ed2c479dd77d54c5R201) for the patch.

How `wasm-reduce` works is to interleave destructive and non-destructive changes to the wasm. "Destructive" changes are things like removing an instruction or changing it - this is destructive in the sense that it can alter what the code does. "Non-destructive" changes are Binaryen's [various optimization passes](https://github.com/WebAssembly/binaryen/tree/master/src/passes), which can reduce code size without changing the code's behavior; the reducer basically throws passes at the code until it sees it can't shrink it any further. In theory just destructive changes are enough, but in practice passes are very fast to run, and also destructive changes only alter a single instruction at a time, which is more likely to get stuck in a local minimum.

Other interesting reducers are [C-Reduce](https://embed.cs.utah.edu/creduce/) which works on C/C++ source files, and LLVM's [bugpoint](https://llvm.org/docs/Bugpoint.html) which works on LLVM IR. The main difference between Binaryen and those is that it works on WebAssembly (although maybe the approach of using compiler passes to reduce is novel, I'm not sure).

So far we've seen the obvious ways in which fuzzers and reducers are helpful. But there's more!

# Automatically Testing Small Changes

Let's imagine that we have the following idea: maybe Binaryen `-O1+` should run the "lots of little peephole optimizations pass" after computing & propagating constants. That means a diff like this:

```diff
-  add("optimize-instructions");
   add("precompute-propagate");
+  add("optimize-instructions");
```

This seems reasonable, and imagine that we get good results on some real-world codebases. Before we commit this, though, we need a testcase for the test suite - we want to make sure the code is right, and that we don't regress it later.

One option is to (drum roll) **write a testcase**. You don't need this post for that :) But note that it’s not that easy to do: we made a change to `-O1` which runs many passes, so it's not like we are crafting the actual input that the relevant passes will see, as they are somewhere in the middle of the pipeline. Previous passes will modify things before them, and so our testcase must remain interesting along the way.

Another option is to **let your computer generate one for you**, as follows:

1. Run the fuzzer to find a testcase.
2. Run the reducer to make it small.

To do this, we can instrument Binaryen's code (where that diff from before was) like this:

```c
  if (getenv("BEFORE")) add("optimize-instructions");
  add("precompute-propagate");
  if (!getenv("BEFORE")) add("optimize-instructions");
```

This lets us set an environment variable `BEFORE` to quickly switch between the old behavior and the new one - if `BEFORE` is set, we run `optimize-instructions` first, like we used to, and otherwise after. Then we can modify Binaryen's fuzzer driver script that was mentioned earlier to add something like this:

```python
  os.environ["BEFORE"] = '1'
  run(["bin/wasm-opt", "a.wasm", "-O1", "-o", "b.wasm"])
  del os.environ["BEFORE"]
  run(["bin/wasm-opt", "a.wasm", "-O1", "-o", "c.wasm"])
  assert open("b.wasm").read() == open("c.wasm").read()
```

This runs `-O1` with both behaviors and sees if the output wasm is different. If it is, the fuzzer will halt, and we have finished the first step of getting an interesting testcase. To run the fuzzer, we run `python scripts/fuzz_opt.py` and go do something else for a while. Then we run the reducer with a script that does something similar so that it keeps the testcase interesting, and wait until it produces a small testcase.

The results sometimes need some manual editing, but they are often pretty readable simply because they are so small. Here's one result I found from the above fuzzing and reduction (in wasm stack format):

```slim
block i32
i32.const 0
global.set 0
i32.const -1
end
i32.const 1
i32.and
i32.load offset=1
```

That gets optimized into:

```slim
i32.const 0
global.set 0
i32.const 2
i32.load
```

What happens here is that we `and` 1 and -1, giving us 1. Together with the offset of 1, the load is at address 2. And we don't need the block of course. Various optimization passes get rid of everything, but we only get to that `and` fairly late, because of the block and how it interacts with other optimizations. So running `optimize-instructions` late is useful as it lets the optimizer see it can combine the load's offset with an arriving constant.

# Automatically Testing New Code

The above approach can work with new code too. Imagine that we have something like this in a new optimization pass we are writing:

```c
  [..]
  // Don’t do this optimization if it's not valid.
  if (isBadCornerCase) return;
  [..]
```

We can instrument that with

```c
  [..]
  // Don’t do this optimization if it's not valid.
  if (getenv("CHECK") && isBadCornerCase) return;
  [..]
```

We can then fuzz and reduce to find a small testcase on which the results are different when we run the pass with and without the check for that corner case. In other words, with very little work we can get our computer to find an example for that corner case.

Never Write Tests Again a.k.a. How Far Can We Take This?
========================================================

We've seen that we can make the computer generate testcases for us based on what difference it makes to the output. And we can in many cases define that difference pretty easily as exactly what we want, something like "where this check for a corner case is important," or "where this new pass reduces code size." Telling the computer **what we want** can be much nicer than manually writing testcases ourselves, and by asking for such testcases we might learn something - maybe the testcase shows the code is wrong or can be improved.

But of course this won't always be useful. Sometimes you do know exactly what tests you want, or it may be faster to figure them out than to use the above technique. Or perhaps you're writing tests first as in test-driven development.

Still, I find that automatically generating testcases helps me quite a lot in practice. It could be even better if it were quicker to instrument the code, and if there were no need to instrument the fuzzer, and if it were faster. The workflow I'm imagining is like this: as you write or modify some code, you ask the computer what amounts to "**why does this line matter?**" which it answers by quickly giving you a small example of exactly why it matters. You can then add that as a test, or use it to guide you in what to write next.

Possibly there are already such tools out there? The closest things I can find are in the Java world, like [randoop](https://randoop.github.io/randoop/) and [EvoSuite](http://www.evosuite.org/). They appear to be more focused on generating sets of tests for a given set of classes, as opposed to the more "targeted" approach I'm thinking of. Maybe I missed something?

