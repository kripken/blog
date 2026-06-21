---
layout:     post
title:      We Don't Understand Neural Networks At The Algorithmic Level
date:       2026-06-20 00:00:00 -0700
categories: neuroscience
---

The largest ongoing debate about AI is "*Are [Large Language Models (LLMs)](https://en.wikipedia.org/wiki/Large_language_model) intelligent?*" That makes sense, at least: the evidence is ambiguous and the stakes are high. Far more confusing is the disagreement on whether we ***understand*** LLMs - how can scientists studying a topic not broadly agree on how much we know about it? Yet respected academics have conflicting opinions, and it is worth seeing why, because it underlies the question of intelligence.

On one side, prominent figures, scientists among them, have [described AI models](https://futureoflife.org/open-letter/pause-giant-ai-experiments/) thus:

> powerful digital minds that no one – not even their creators – can understand

Skeptics are right to scoff at the marketing hype of "powerful digital minds", but many scientists agree that our understanding of LLMs is very limited. Here is a [more subdued](https://openaipublic.blob.core.windows.net/neuron-explainer/paper/index.html) example:

> Language models have become more capable and more widely deployed, but we do not understand how they work.

The other side of this debate is presented well in a new preprint by Olivia Guest, Nancy Abigail Nuñez Hernández, and Mark Blokpoel, ["**Understanding Artificial Neural Networks: Mysterianism about Known Mechanism is Mysticism**"](https://zenodo.org/records/20071869). Disagreeing with the above two quotes, which they provide as examples, they write:

> We do know the mechanistic structure of [artificial neural networks (ANNs)] because we designed and built them. We also do know their functional role (what they are for) as well as the mathematical function they are asked to approximate (map inputs to target outputs).

As they correctly say, there is a lot that we know. But is it enough?

They also write this:

> ANNs, as turbo-charged statistical models [..] can only but provide correlations.

This position is, I think, representative of a particular line of AI skepticism, related to the "[**Stochastic Parrot**](https://en.wikipedia.org/wiki/Stochastic_parrot)" and "[**Blurry JPEG of the Web**](https://www.newyorker.com/tech/annals-of-technology/chatgpt-is-a-blurry-jpeg-of-the-web)" metaphors. There is a lot of truth in those views! However, this preprint is an example of how they can be taken too far: **we do *not* fully understand ANNs, and in part that is *because* they can do more than provide correlations.**

Here are the main points I'll be making in this post:

* Even though we understand ANNs' goals at the **top** level, and their math at the **bottom**, we don't understand the algorithms in the **middle**.
* This is so despite the simple math in each artificial neuron. **Enough such simple units are provably [Turing complete](https://en.wikipedia.org/wiki/Turing_completeness)**: capable, in principle, of any algorithm. There is a vast number of things that ANNs could be doing internally. Scientists are still figuring out which.
* The preprint, like Ted Chiang's ["Blurry JPEG of the Web" piece](https://www.newyorker.com/tech/annals-of-technology/chatgpt-is-a-blurry-jpeg-of-the-web), views ANNs as **storing data**, a compressed version of the training set. Still, we must ask - as Chiang does - ***how*** do ANNs compress? And that question brings us back to algorithms.

I am not saying anything new here, but I think it's worth seeing this all together, and in the context of whether we understand ANNs.

(Note: LLMs are ANNs, that is, large language models are artificial neural networks. I'll mostly use the term ANN, following the preprint, except where the distinction matters.)

## Levels of analysis

The cognitive scientist and neurobiologist [David Marr](https://en.wikipedia.org/wiki/David_Marr_(scientist)) defined [three important levels of analysis](https://en.wikipedia.org/wiki/Level_of_analysis#Marr's_tri-level_hypothesis):

* **Computational**: What the system aims to do at the *highest* level, what tasks it is built to solve.
* **Algorithmic / Representational**: How, in an abstract sense, the system represents information, and how it manipulates those representations to complete its tasks.
* **Physical / Implementation**: The actual details of how the system is built, at the *lowest* level.

Tragically, David Marr died at a young age in 1980. That was before computers were ubiquitous, but today, computers are probably the easiest way to explain his levels. Let's focus on one type of computer program, a [chess engine](https://en.wikipedia.org/wiki/Chess_engine), which plays chess. Today, chess engines often use ANNs, but for now I just mean a "traditional" engine - some ordinary code that people wrote, that plays chess (most engines were like that until a few years ago). Here is what Marr's levels mean for such a chess engine:

* **At the *computational* level, a chess engine plays chess**. Given a board, it picks the next move. Its goal is to pick the best move so as to eventually win the game.
* **At the *algorithmic* level, a chess engine picks the next move using some specific approach.**
  * It may have a list of *memorized positions and outcomes*, which is useful at the start and [end](https://en.wikipedia.org/wiki/Endgame_tablebase) of a game. Looking at possible moves, it sees which are memorized, and picks the one with the best outcome.
  * It may use *calculation*. While it is impossible to calculate all possible moves to any useful depth, heuristics can focus on promising ones (according to some rules, like checking all captures), while estimating the outcomes (according to some other rules, perhaps giving [knights 3 points and rooks 5](https://en.wikipedia.org/wiki/Chess_piece_relative_value)). It then picks the move with the best estimated outcome.
* **At the *implementation* level, a chess engine is written in some programming language like C++, Rust, or Python.** It is compiled to binary code that runs on a physical CPU and memory.

Note how many options there are at the algorithmic level. Some are more robust, some less. Some are more sophisticated, some less. Some are more similar to how humans play chess, some less. Some can defeat top human player [Magnus Carlsen](https://en.wikipedia.org/wiki/Magnus_Carlsen), some can't.

Of course, Marr's three levels are a simplification: even in the bottom level here, we have both human-readable code and the physical CPU that runs a binary version of that code. That is, each level has sub-levels. And sometimes the boundaries between them are not crisp. Still, this is a very helpful way to look at things: **the goal of playing chess can be achieved using many algorithms.**

## The algorithmic level matters

The preprint authors claim that ANNs are not "*outside our current expert understanding*". In support of that, here again are the things the authors say we ***do*** understand about ANNs:

> We do know the mechanistic structure of [ANNs] because we designed and built them. We also do know their functional role (what they are for) as well as the mathematical function they are asked to approximate (map inputs to target outputs).

Indeed, in terms of Marr's levels, we certainly understand ANNs at the **top** and the **bottom**. What remains to consider is the ***middle*** level, the algorithms and representations.

Let's leave ANNs for the moment and return to our chess engine, where things are simpler: it is just some software that people wrote.

Say we want to beat the chess engine, with money riding on the game. We understand it at the **highest** level - it plays chess. Suppose that we also have access to it at the **lowest** level: it is running on a computer right in front of us, and we are allowed to use the keyboard and mouse and so forth. Does this help us defeat it? Well, given it is a machine, we can pull the plug if the game goes badly. But if we want to beat it fair and square then we should look at the *algorithmic* level in the **middle**.

Say that we know the chess engine only calculates up to a depth of 3 moves. We could strategize around that, because it will not see threats that take 4 moves to materialize. In other words, **different algorithms have different Achilles' heels** - and knowing them can make the difference between winning and losing.

Note that even if we were given access to the chess engine's source code, we would try to infer higher-level things from it. That is, we might scour the code for hints about what depth it searches to. Or, we could test the machine's observable behavior and measure its average calculation depth that way. Either way, we are aiming for an *algorithmic*-level understanding.

**We are not talking about anything metaphysical or mysterious here.**[^marr] We can measure calculation depth with human players too.[^human] (The average depth Magnus Carlsen can calculate to is far better than my own, for example.)

Calculation depth may seem like a small matter. But far greater differences are possible: if we know that the engine only uses memorized moves - without flexible calculation - then we can defeat it easily, because there are [too many](https://github.com/tromp/ChessPositionRanking) chess positions to memorize. All we need to do is play odd-enough moves to wind up in territory it has nothing memorized for, and it will blunder.[^toomany] This would be a guaranteed way to win (and perhaps make a lot of money).

There are, of course, situations with far higher stakes. Consider an app that helps people count carbohydrates and vitamins.[^consume] This can be very important for diabetes and other medical conditions. The app lets you pick the things you eat from a list and add estimated portion sizes. It then prints out nutritional values.

The app could be doing this in many ways. Does it use an expert-curated database of nutritional values? Does it take into account interactions (some nutrients inhibit the absorption of others)? Unless the app's creators document such things (and we trust them), we would need to carefully test the app or piece things together from its binary code. These internal details could have medical implications, so I would not want to use such an app without knowing them!

Clearly such algorithmic-level details - in Marr's middle level - matter. Why do the preprint authors focus on the top and bottom levels instead? For good reasons: **we do not need to be *aware* of everything about a system in order to understand it.** Also, nothing changes if the app crashes sometimes: **we do not need to be able to *predict* everything about a system we understand.** I agree on both counts: yes, there is a fundamental, important level of understanding that we have just by knowing the top and bottom levels of a system, its goals and its implementation. The details in the middle are just details (which, often, we can't know all of anyhow).

And the preprint authors argue this for a good reason. AI fans sometimes suggest that there is something mysterious about ANNs that we do not or even *cannot* understand, something as special as human thought - perhaps even the very same thing itself. We do not understand human consciousness, and if we do not understand ANNs... perhaps they are conscious, too? This is, of course, wildly speculative.

It is fine to speculate about these things. Some of the smartest people in the world have [debated the connections between algorithms, understanding, mechanism, consciousness, and meaning](https://plato.stanford.edu/entries/chinese-room/) - but they arrived at no clear philosophical conclusions. So it is best to separate the metaphysics from the obvious practical reasons to investigate chess engines and medical apps at an algorithmic level.

And, at that practical level, **the more algorithmic details we learn about a chess engine or a medical app, the more we understand it, with potential impact to our finances and health.** "Understanding" is not a binary, but a matter of ***degree***, in the way the term is commonly used.

And all this applies to ANNs as well. When we [build an ANN](https://en.wikipedia.org/wiki/Neural_network_(machine_learning)#Learning), we pick the low-level mathematical details, and we provide it examples of what we want - for a given input, we tell it what the right output is. But we don't tell it ***how*** to produce that output. When training an ANN for chess, we don't tell it whether to use memorization or move calculation. We don't suggest what depth to calculate to or which Achilles' heels it should or should not have.[^datahow] We must investigate the ANN to figure out which algorithms it uses.

But, wait - *do* ANNs use algorithms? Yes.

## ANNs can implement any algorithm

Let me re-quote the preprint authors:

> ANNs, as turbo-charged statistical models [..] can only but provide correlations.

This is a strong claim. Let's consider it.

Philosophically, we can debate whether showing observations to a computer [or a human](https://en.wikipedia.org/wiki/Humean_definition_of_causality) limits them to correlations, but we will not find any clear answer. Instead, let's focus on the practical and scientific aspects here.

Can a **laptop** do more than provide correlations? I think so, since we can play Elden Ring on it or watch cat videos. Even if we focus on software that returns an output for an input - like trained ANNs do - then a computer can run a mathematical proof-checking program or calculate chess moves. Surely those go beyond correlations? If so, what are ANNs missing?

Nothing, it turns out, from the standpoint of computer science. This might be surprising! While it is true that ANNs often do poorly at tasks which laptops do with ease, like adding large numbers, mathematical proofs show that ***in principle*** ANNs have the same computational power as laptops. Specifically, **ANNs are [universal approximators](https://en.wikipedia.org/wiki/Universal_approximation_theorem), meaning they can compute any function between inputs and outputs, to any precision (if large enough).**[^traintoo]

ANNs are also [**Turing complete**](https://proceedings.iclr.cc/paper_files/paper/2025/hash/123d3e814e257e0781e5d328232ead9b-Abstract-Conference.html),[^Qiu2025] meaning they can compute anything any computer can (if large enough, and when run in a loop[^iflarge]). **ANNs can calculate chess moves and implement any algorithm, just like laptops.**[^church]

When Marr talked about algorithms, he did not only mean interesting ones like [quicksort](https://en.wikipedia.org/wiki/Quicksort) and [A\*](https://en.wikipedia.org/wiki/A*_search_algorithm) that college professors teach on whiteboards. Any way of *processing information* is relevant here, which also includes very simple things like memorizing examples. **But we do have plenty of scientific evidence of ANNs doing *interesting* things, algorithmically:** this is the focus of the field of [**neural computation**](https://en.wikipedia.org/wiki/Neural_computation), and in recent years, work along those lines that focuses on LLMs has been termed [**mechanistic interpretability**](https://en.wikipedia.org/wiki/Mechanistic_interpretability). A great deal has been discovered inside ANNs, things like accurate internal [representations](https://openreview.net/forum?id=DeG07_TcZvT)[^li2023] and complex [analyses](https://dl.acm.org/doi/10.5555/3692070.3692961)[^jin2024].

Chess is particularly well-studied. ANNs [represent the board internally](https://openreview.net/forum?id=PPTrmvEnpW)[^karvonen] (a "world model") and can compete at a level [comparable to top human players](https://papers.nips.cc/paper_files/paper/2024/hash/78f0db30c39c850de728c769f42fc903-Abstract-Conference.html).[^amortized] Systems using ANNs can [learn to play chess at a superhuman level, even without human guidance or data](https://www.science.org/doi/10.1126/science.aar6404).[^azero] And, remarkably, the [things chess ANNs compute after training](https://www.pnas.org/doi/10.1073/pnas.2206625119)[^ACK] often correspond to human concepts around chess, things like piece values and "*can the queen be captured?*"

That ANNs process information in interesting ways is perhaps not surprising. When an ANN plays chess at a high level, it does so after being trained on far more games than it can memorize, so simple recall is not enough. Likewise, when an LLM speaks so fluently that it passes the [Turing Test](https://en.wikipedia.org/wiki/Turing_test#Large_language_models), then large as it is, it was trained on a far greater volume of text:

<img src="/blog/assets/llm_data.svg" alt="graph of an LLM compared to its training data. the data is far, far larger" style="width: 100%; border: 2px solid black; box-shadow: 5px 5px 5px black;"/>

To predict general patterns and regularities in that amount of data requires some form of information processing. It may be more or less sophisticated, but there is something to investigate, algorithmically.

To be clear, ANNs can only implement any algorithm *in principle*. The math in ANNs can implement quicksort and A\* and chess calculations to a depth of 5 and anything else. But that does not mean that ANNs do so *in practice*, since a particular ANN may not be large enough. Furthermore, even given a sufficiently large network, it is an open scientific question whether ANN training actually searches the space of all algorithms. That we do not know the answer to this fundamental question - whether ANNs reach their theoretical potential, or not - is one reason why ANNs remain outside our current expert understanding.[^major]

We have no reason to think any of this is an unsolvable mystery. Papers are constantly being published that show significant progress, like the examples in this section. It is just that, currently, much remains unknown.

## Logistic regression

One of the major arguments the preprint makes is a comparison to [**logistic regression**](https://en.wikipedia.org/wiki/Logistic_regression):

<figure>
  <math xmlns="http://www.w3.org/1998/Math/MathML" style="margin: 1em; font-size: 1.5rem">
    <mrow>
      <mi>p</mi>
      <mo stretchy="false">(</mo>
      <mi>x</mi>
      <mo stretchy="false">)</mo>
      <mo>=</mo>
      <mfrac>
        <mn>1</mn>
        <mrow>
          <mn>1</mn>
          <mo>+</mo>
          <msup>
            <mi>e</mi>
            <mrow>
              <mo>&#x2212;</mo>
              <mo stretchy="false">(</mo>
              <mi>x</mi>
              <mo>&#x2212;</mo>
              <mi>&#x03BC;</mi>
              <mo stretchy="false">)</mo>
              <mo>/</mo>
              <mi>s</mi>
            </mrow>
          </msup>
        </mrow>
      </mfrac>
    </mrow>
  </math>
  <figcaption>
    <small>logistic regression</small>
  </figcaption>
</figure>

We don't need to get into the actual equation, but this is an important statistical model which uses simple math to predict data based on examples. **The authors correctly say that we understand logistic regression.** Furthermore, the authors point out that ANNs are composed of many elements - many artificial neurons - each of which does basically the same math as that logistic regression equation, hence **ANNs and logistic regression are closely related.** They write:

> If we do indeed understand how a single unit in an ANN works, what is the difference when there are many hundreds or thousands? Does understanding a single unit not imply mechanistic understanding of more than a single unit? Is ‘mechanistic understanding’ not exactly this form of understanding?

The answer is that researchers from the fields of neural computation and mechanistic interpretability also mean the ***algorithmic*** level,[^mech] and, **at that level, there *is* a fundamental difference between a single unit and a group.**

Logistic regression is not Turing complete. It cannot approximate any function like ANNs can. The math in logistic regression is just too simple for that, it turns out.

This is a classic result in neural computation: **a single neuron is not a universal approximator.** Even multiple neurons in a single layer are not - *another* (hidden) layer [is needed](https://en.wikipedia.org/wiki/Perceptron#Perceptrons_(1969)). So, yes, logistic regression is equivalent to a single artificial neuron, but a single neuron has far less power than a group.

And that difference in power is incredibly vast. Being Turing complete means the ability to do anything that the smartest software engineers could possibly make a computer do. And more: there is no reason to limit ourselves to things humans have thought of. The space of all algorithms includes every possible rule, pattern, logical inference, approximation, mathematical proof, computation, all combinations of those, and more.

As mentioned before, this is in theory. But, even if ANNs search only a small region in the space of all algorithms, plenty of evidence shows that area is far, far larger than what logistic regression covers. For example, a single logistic regression can't even identify legal moves in chess, much less play competently, but ANNs have expert ability there.

I am certain that the preprint authors are aware of ANNs' Turing completeness and universal approximation and everything else I said above. My point is not that they have made some kind of oversight. I believe the root of the disagreement is in how the term "understanding" is used: for them, understanding a single logistic regression implies the understanding of many of them. Of course, in a way it does: understanding one billiard ball is enough to understand a game of pool or even larger numbers of them as in the figure below, even if we can't predict every movement because of complexity or because of details like an uneven pool table.

<figure>
  <img
    src="https://upload.wikimedia.org/wikipedia/commons/6/6d/Translational_motion.gif"
    alt="colliding particles bouncing around. each ball just follows the simple rules of physics, but as a whole it looks rather chaotic"
    style="border: 2px solid black; width: 40%"
  >
  <figcaption>
    <small>animation of colliding particles, similar to billiard balls, from <a href="https://en.wikipedia.org/wiki/File:Translational_motion.gif">Wikipedia</a> (public domain)</small>
  </figcaption>
</figure>

But there is actually more to understand even there. It is uncontroversial in physics to say that [statistical mechanics](https://en.wikipedia.org/wiki/Statistical_mechanics) improved our understanding of the world, teaching us things about how **large collections** of billiard balls behave.[^math] Sometimes understanding a single unit does ***not*** imply understanding of more than a single unit.

And it is uncontroversial among researchers in the fields of computer science, neural computation, and mechanistic interpretability to say that investigating algorithms improves our understanding of large systems of artificial neurons. Marr stressed the importance of that, and we have seen why it makes sense: logistic regression does **one** thing, and we understand it. ANNs can do ***any***thing, in theory, and in practice it is very hard to find out what. 

I don't think the preprint authors' definition of "understanding" is wrong. I think there are other valid ways in which scientists use that term.

## Science, ANNs, and Marr's middle level

So far we mostly focused on **practical** reasons for understanding the algorithms that ANNs use: it helps us find Achilles' heels, etc. We also briefly mentioned **scientific** reasons. Let's look into that in more depth, using this prompt and LLM response:

<img src="/blog/assets/crev.png" alt="claude query: Below is a scene from a high fantasy novel. Describe in one sentence what is happening. Crevyzik's eyes remained on the ground before him. The large man did not seem to hear the shouted commands. 'Run, Crevyzik!' the soldier yelled. 'Our position will be overtaken soon!'. LLM response: 'A soldier is desperately urging a large, unresponsive man named Crevyzik to flee before their position is overrun by the enemy.'" style="border: 2px solid black; box-shadow: 5px 5px 5px black;"/>

"Crevyzik" is a bizarre character name (as befits [high fantasy](https://www.reddit.com/r/Fantasy/comments/aq5smx/whats_the_deal_with_fantasy_authors_giving_their/)). Surely it does not appear in any text the LLM was trained on. But the LLM responds well here, and would do so no matter what random name you give.[^names] How?

Do LLMs parse sentences into parts of speech like verbs and nouns? Using that representation, they could infer which words were character names. Since names are arbitrary to a large extent, operations on that representation could then work the same way no matter what the names are - names would be "variables" as in math or programming. This would explain how LLMs are so consistently good at such prompts, and it would also show that they reason properly about character names at least. But is any of this true?

There is [some](https://direct.mit.edu/coli/article/50/4/1441/123789/Can-Language-Models-Handle-Recursively-Nested)[^gram1] [evidence](https://www.pnas.org/doi/10.1073/pnas.2400917121)[^gram2] of grammatical ability in LLMs. It does appear that LLMs process language in a general way, [beyond superficial correlations](https://www.researchgate.net/publication/383119605_Language_Models_as_Models_of_Language).[^gram3] However, even if so - and this is still contested! - we don't know any of the details, because there are many possible ways to parse text into parts of speech and so forth. **This is the sort of thing scientists want to understand - and still don't.**[^stat]

This matters for practical reasons. Some text-processing algorithms are more robust, some less. Some are more sophisticated, some less. Some are more similar to how humans reason, some less. Some can summarize (or translate or rewrite) in useful ways, some can't.

Of course, if you have no interest in using ANNs, this won't matter to you. And that is fine - ANNs can't help everyone or every field. But for those considering ANNs, it is of great practical importance to know which algorithms they use. And **it is science's job to figure out how ANNs work** - who else is going to do it?

**And this also matters scientifically in a direct sense.** Scientists have found ANNs useful as *tools*, in areas like [protein folding](https://en.wikipedia.org/wiki/AlphaFold) and [math](https://openai.com/index/model-disproves-discrete-geometry-conjecture/). Humans have also learned [new tactics in chess](https://www.pnas.org/doi/10.1073/pnas.2406675122)[^new] from the ANN component of the [AlphaZero](https://en.wikipedia.org/wiki/AlphaZero) chess engine. When an ANN solves a problem, it uses some algorithm - potentially one that humans never thought of. The algorithm may be interesting, and its results as well.

**I want to stress that there is nothing metaphysical going on here.** That we can learn new things from ANNs is not mysterious - we can also do so from [monkeys banging on keys](https://en.wikipedia.org/wiki/Infinite_monkey_theorem), after all, since they will (eventually) print out a description of every possible algorithm. But ANNs do this in a timeframe that is actually *useful* to us, unlike the monkeys - and no other technique has come close to this achievement.

## Data

The preprint authors do accept that there is *something* we do not know about ANNs:

> What we do not understand is the data compressed or stored, lossily or losslessly, inside the model.

**The data in the model comes from the world, and there are things we do not understand about the world, after all.** To make this point clear, they compare ANNs to databases:

> [databases and ANNs] do not embody an understanding of the world, nor the phenomena within it [..] What they do is in fact exclusively house data

I find this view fascinating! The idea that it is the *data* we need to understand, and not the algorithms the model derived from the data - that is, honestly, a perspective so different from mine that I struggle to follow it. My own, of course, is what I presented so far, the 'algorithm' framing which is common among neuroscientists and computer scientists (my own fields).

But even though the 'algorithm' and 'data' perspectives seem at odds, I believe they are in fact equivalent, for the following reasons.

Note, first, that **the same data can lead to different ANNs.** For example, training larger ANNs on the same chess games often leads to [better performance](https://openreview.net/forum?id=PPTrmvEnpW)[^karvonen]. But how *can* the same data lead to different ANNs? Because, in the case of large amounts of data, it is not simply stored - it must be ***compressed***, as the preprint authors say.

There are many ways to compress data: **compression uses algorithms.** And, as we saw, the math in ANNs is Turing-complete, meaning they can use any conceivable algorithm to do so. Even if we see ANNs as databases, they are databases with very complex mechanisms that we haven't completely figured out yet. **Compression is part of ANNs, and we do not understand that compression, so even in the 'data' view we do not fully understand them.**

Everything I wrote before can be reframed in this 'data' view, without anything fundamental changing. For practical reasons, **to learn the Achilles' heels of an ANN, we still need to study it**: whether it calculates 2 moves ahead or 4 on average is determined both by the data and how it is compressed. And the compression in ANNs is scientifically interesting because it can use any algorithm, and we don't know which in advance - **the data does not tell the ANN *how* to compress it, after all.**

Why do some ANNs play chess or summarize text better than others? More data can help, a point the preprint authors make. However, architectural differences matter too, by influencing how the data is compressed (e.g., [convolutional neural networks](https://en.wikipedia.org/wiki/Convolutional_neural_network) vs. [transformers](https://en.wikipedia.org/wiki/Transformer_(deep_learning))).

How can an ANN be better than humans at chess? The ANN may have simply been shown more games than any human ever has.[^autogen] However, recall that no amount of memorization is enough to play chess well, so it must also be compressing all those games into a form that handles **novel** situations properly too. Notably, no other machine learning technique - or any other software, for that matter - handles novelty so well across a broad range of tasks, [not only chess but vision and text and more](https://www.nature.com/articles/nature14539). That ANNs do so while also responding **efficiently** to queries in a reasonable time is even more remarkable. Something very interesting is going on in the compression here!

While interesting, this is not mysterious. We can make it very concrete with an example: imagine a chess ANN that evaluates a board by counting how many pieces each side has, giving [knights 3 points and rooks 5](https://en.wikipedia.org/wiki/Chess_piece_relative_value) and so forth. This is a simple algorithm that works well in many cases. We can see this as a form of compression, a very lossy one in fact, as any two boards with the same pieces end up compressed to the same thing, no matter how the pieces are positioned or moved.[^datahow2] Of course, this is just one algorithm - or one form of compression - that a chess ANN might be using. Even in the 'data' perspective, we must consider algorithms.[^sql]

## On metaphors

In **Ted Chiang's 2023** [**piece on ChatGPT**](https://www.newyorker.com/tech/annals-of-technology/chatgpt-is-a-blurry-jpeg-of-the-web), he writes:

> Think of ChatGPT as a blurry JPEG of all the text on the Web. It retains much of the information on the Web, in the same way that a JPEG retains much of the information of a higher-resolution image

Like the preprint authors, Chiang considers ANNs from the 'data' perspective. But there is a very large difference. The authors mention compression but do not consider it in detail. Chiang does, and correctly points out that there are better and worse ways to do so:

> To grasp the proposed relationship between compression and understanding, imagine that you have a text file containing a million examples of addition, subtraction, multiplication, and division. Although any compression algorithm could reduce the size of this file, the way to achieve the greatest compression ratio would probably be to derive the principles of arithmetic and then write the code for a calculator program. Using a calculator, you could perfectly reconstruct not just the million examples in the file but any other example of arithmetic that you might encounter in the future.

In other words, when one recognizes the principle behind something, its data can be compressed very well. Compression therefore *might* indicate some level of understanding, at least in theory. Chiang's conclusion is that LLMs in 2023 did not show true understanding, but he kept an open mind.[^reuse] And that is the right thing to do. **LLMs *can* be seen as databases, but we shouldn't conclude that they are as unintelligent as ordinary databases are.**

Another popular metaphor in this space can be taken the wrong way as well, the [**Stochastic Parrot**](https://en.wikipedia.org/wiki/Stochastic_parrot):

> "Contrary to how it may seem when we observe its output, an [LLM] is a system for haphazardly stitching together sequences of linguistic forms it has observed in its vast training data, according to probabilistic information about how they combine, but without any reference to meaning: a stochastic parrot."

I would quibble with the word "haphazard",[^hap] but otherwise this is useful and accurate. Imagine we ask an LLM this:

> What did you have for breakfast?

If it responds compellingly about having had orange juice and cereal, we still can't take it seriously, because it did not eat anything. Humans have a tendency to interpret fluent text as meaningful, but LLMs are still only machines that predict words.

And LLMs certainly use probabilistic information about their data in order to produce that fluent text, as the Stochastic Parrot metaphor states. But this can be taken too literally,[^literal] as if LLMs *only* provide correlations and nothing more. Here this topic is intertwined with the one before, and with the question of intelligence: if LLMs only do statistics, then they are really just databases that use statistics to compress their data. The details are not interesting. (Are statistics ever interesting?) Only the summarized data is.

**LLMs *do* use statistics - during their training and often after - but they might also be doing more.** Computer scientists have proven that combining many logistic-regression-like correlations leads to Turing-completeness, which means ANNs could be using any algorithm whatsoever to compress their data.[^electricity] As of 2026, those algorithms remain an open scientific question, so we do not understand ANNs at Marr's middle level.

## Conclusion

ANNs may be scientifically interesting and sometimes useful, but also have risks and cause harm. I agree with the preprint authors on many things,[^agree] because AI skepticism has a very important role today, both in outlining the dangers and clarifying the misconceptions. But our skepticism must be accurate.

When **startup CEOs** say "*We don't understand LLMs*", that can be a marketing line, a cynical way to boost the valuation of their companies. And when **AI fans** say "*We don't understand LLMs*", they might be justifying speculation about those machines being conscious or alive. The preprint authors are correct to criticize both cases. Yes, we ***do*** understand LLMs in important ways, despite ridiculous AI hype.

But when **scientists** say "*We don't understand LLMs*", they are being humble in the face of all that we really do not know about incredibly challenging questions in their field. There is a lot we ***don't*** understand here.

I have suggested that we separate any metaphysical speculation about ANNs from the science. But, of course, an individual might talk about both. Nobel Prize laureate [Geoffrey Hinton](https://en.wikipedia.org/wiki/Geoffrey_Hinton) probably knows more about ANNs than anyone else - and he has also said that he believes AI models [might be conscious](https://www.psychologytoday.com/us/blog/the-mind-body-problem/202502/have-ais-already-reached-consciousness). Is his metaphysical speculation related to the current gaps in the science? Perhaps yes, but perhaps no - I can't say what is in his mind.

Putting Hinton aside, there probably are people who make inferences like this: "*We don't know how ANNs work, so they could be conscious.*" If so, then an analysis like the preprint authors' could explain why such a leap is unsound.[^leap] But it is the *leap* that is unsound, not the starting point: Hinton and other scientists are absolutely right to say that ANNs are outside our current expert understanding.

## Acknowledgements

Thank you to Gian-Carlo Pascutto, Vincent Carchidi, SE Gyges, and Marcia Cohen-Zakai for helpful comments and suggestions on drafts of this post.

## Comments

Feel free to respond [on Bluesky](https://bsky.app/profile/kripken.com/post/3mosqh5zsa22d).

## Footnotes

[^marr]: The preprint authors are of course aware of Marr's levels, but they mention them in a very different context than this post:

    > Bridging the Leibnizian gap is the general case of so-called Marrian bridging laws. All this is impossible in principle, as mentioned above, in both engineered and biological systems

    [Leibniz's gap](https://en.wikipedia.org/wiki/Leibniz%27s_gap) is an interesting philosophical question: we cannot explain thoughts, feelings, and perceptions in a mechanistic way, because looking at a brain from a scientific standpoint, we do not see thoughts - we only see chemistry and physics. There is certainly a metaphysical mystery here, involving consciousness, qualia, and the mind-body problem.

    I agree with what I think is the preprint authors' point around this: if someone looks for thoughts, feelings, and perceptions in ANNs, then they will not find them; and if AI fans claim that we do not understand ANNs for this reason, then the fans are making a poor argument.

    In this post, however, we are only considering Marr's levels from a practical and scientific standpoint (which, I believe, is also Marr's original intention). We'll focus on algorithms and math, not topics in philosophy of mind.

[^human]: Of course, human players do not have simple brute-force calculation to a fixed depth. There are many [jokes and quips](https://www.chesshistory.com/winter/extra/movesahead.html) about this: "I only calculate one move ahead - but it is the *right* one"; "The stronger the opponent, the more moves I can foresee because he makes the correct moves".

    Still, some players simply calculate faster and to a greater depth. This can be measured in chess puzzles, for example. However, the most common measurement used on chess players is not calculation depth but [Elo](https://en.wikipedia.org/wiki/Elo_rating_system). I use depth as the example in the main text as I hope it is more intuitive for non-chess players.

    Note that computer chess engines also do not have a fixed depth. An engine may calculate more moves when more time is available or when the confidence of its heuristics is low, since time must be managed carefully (if it runs out, you lose). But, again, we can still measure the *average* depth (which might be a fraction), or things like Elo.

[^toomany]: It is well-known that memorization is not enough to be good at chess. On [lichess](https://lichess.org/), for example, you can see when games reach a novel position, that is, one not in the database. Channels like [ChessNetwork](https://www.youtube.com/user/ChessNetwork) also mention this in analysis videos of tournament games.

[^consume]: This is [not entirely](https://www.diabettech.com/i-asked-ai-to-count-my-carbs-27000-times-it-couldnt-give-me-the-same-answer-twice/) a hypothetical.

[^datahow]: It may seem that, in some sense, the data *does* suggest how to process it: if we train an ANN on chess games from a player that only calculates to a depth of two, wouldn't the ANN end up doing so itself? But even in this case, the ANN may simply end up memorizing all the examples and nothing more, and if so, it would show a calculation depth of two on those examples but do far worse on other positions. That is, the real question is how well an ANN generalizes *outside* the training set. And the training set, by definition, cannot tell the ANN what to do outside of it.

    The data is very important here, however. We will return to it in a later section.

[^traintoo]: To be precise, these results refer to any "reasonable" function (continuous, etc. - we don't need to get into the mathematical details, which don't cause any significant real-world limitations).

    Note also that this is for the *trained* network. There are also results about the universality of the [training process](https://www.sciencedirect.com/science/article/abs/pii/S0893608022000570),[^welper] gradient descent. Interesting as well are findings that the trained network can do [something like gradient descent itself](https://neurips.cc/virtual/2023/poster/71931),[^ahn] and so perhaps both can be analyzed as one.

[^Qiu2025]: "*Ask, and it shall be given: On the Turing completeness of prompting*" Ruizhong Qiu, Zhe Xu, Wenxuan Bao, Hanghang Tong. ICLR 2025.

[^iflarge]: To be precise, for any algorithm and any problem size, there is some size of ANN which can execute that algorithm on problems of that size.

    To be literally Turing-complete, a system needs an infinite amount of time and space, which of course no actual physical system has, neither ANNs nor laptops. But, in both cases, as we give the system more resources, it becomes able to compute more things, and without theoretical limit.

    Making this a little more complex is that common ANN architectures like the [transformer](https://en.wikipedia.org/wiki/Transformer_(deep_learning)) cannot "loop". As a result, they cannot literally implement a general algorithm, but only an "unrolled" version of it for a particular problem size. This is still powerful enough in the limit, but it means that just giving a *fixed* LLM more time and more space will not help it - in a sense a time and space limit is "baked in".

    However, that is only a limitation when an LLM is executed once. Modern "reasoning" models are run several times, producing intermediate output ("chain of thought") that the next execution consumes to continue work on a longer task. This, effectively, gives the model more time and space, [avoiding](https://openreview.net/forum?id=NjNGlPh8Wh)[^cot] the computational complexity [limits](https://openreview.net/forum?id=3EWTEy9MTM)[^cot2] of a [single](https://icml.cc/virtual/2023/poster/23713)[^cot3] run. Running ANNs in a loop like this makes even a fixed model Turing-complete, though there may be limits here as well (context size).

[^church]: To be precise, the claim that any Turing-complete machine can compute any algorithm is the [Church-Turing thesis](https://en.wikipedia.org/wiki/Church%E2%80%93Turing_thesis). This post doesn't depend on that, however: we could everywhere replace "algorithm" with "algorithm computable by a Turing machine", without changing anything (i.e., the set of all algorithms computable by a Turing machine is the set of all algorithms that we can currently conceive of, and I am of course only talking about that set).

[^li2023]: "*Emergent World Representations: Exploring a Sequence Model Trained on a Synthetic Task*" Kenneth Li, Aspen K Hopkins, David Bau, Fernanda Viégas, Hanspeter Pfister, Martin Wattenberg. ICLR 2023.

[^jin2024]: "*Emergent representations of program semantics in language models trained on programs*" Charles Jin, Martin Rinard. ICML 2024.

[^karvonen]: "*Emergent World Models and Latent Variable Estimation in Chess-Playing Language Models*" Adam Karvonen. COLM 2024.

[^amortized]: "*Amortized Planning with Large-Scale Transformers: A Case Study on Chess*" Anian Ruoss, Grégoire Delétang, Sourabh Medapati, Jordi Grau-Moya, Li Kevin Wenliang, Elliot Catt, John Reid, Cannada A. Lewis, Joel Veness, Tim Genewein. NeurIPS 2024.

[^azero]: "*A general reinforcement learning algorithm that masters chess, shogi, and Go through self-play*" David Silver, Thomas Hubert, Julian Schrittwieser, Ioannis Antonoglou, Matthew Lai, Arthur Guez, Marc Lanctot, Laurent Sifre, Dharshan Kumaran, Thore Graepel, Timothy Lillicrap, Karen Simonyan, Demis Hassabis. Science, 7 Dec 2018.

[^ACK]: "*Acquisition of chess knowledge in AlphaZero*" McGrath, Kapishnikov, Tomašev, Pearce, Wattenberg, Hassabis, Kim, Paquet, Kramnik. PNAS, 14 Nov 2022.

[^major]: We don't even know if this is a matter of probability or not. There is always *some* chance to find the best algorithm for a given network and training set, since the initial random weights might start out close enough to it (maybe even exactly there). This chance may be extremely small in general. But it may also be a virtual guarantee: while small networks tend to get "stuck" in an imperfect solution (a local minimum), this plagues large networks less, and it is conceivable - but we do not know! - that, as network size tends to infinity, gradient descent never gets stuck (and if it does not, then we would always find the best algorithm, eventually). This would be incredibly useful for those using ANNs, if so, but we do not have a mathematical proof either way.

[^mech]: Note that the term "mechanistic" is used differently by the preprint authors and by mechanistic interpretability researchers.

[^math]: The preprint authors give the [double pendulum](https://en.wikipedia.org/wiki/Double_pendulum) as an example of a system whose math we grasp, and that therefore we understand it even if we cannot predict its chaotic behavior (click to animate):

    <figure>
      <img
        id="pendulum-gif"
        src="https://upload.wikimedia.org/wikipedia/commons/c/c9/Double-compound-pendulum-dimensioned.svg"
        data-animate="https://upload.wikimedia.org/wikipedia/commons/3/32/Double_pendulum_simulation.gif"
        data-still="https://upload.wikimedia.org/wikipedia/commons/c/c9/Double-compound-pendulum-dimensioned.svg"
        alt="double pendulum"
        style="border: 2px solid black; width: 40%; cursor: pointer"
        onclick="togglePendulum()"
      >
      <figcaption>
        <small>animation of a double pendulum from <a href="https://commons.wikimedia.org/wiki/File:Double_pendulum_simulation.gif">Wikipedia</a> (<a href="https://creativecommons.org/publicdomain/zero/1.0/deed.en">CC0 1.0</a>)</small>
      </figcaption>
    </figure>

    On a single double pendulum I agree. However, as just mentioned, statistical mechanics is a case where we knew some amount of math - how individual particles behave - but did not understand for a long time important consequences of that math for larger collections of particles.

    There are of course many other examples of simple equations which were very hard to understand and prove one way or another, like [Fermat's Last Theorem](https://en.wikipedia.org/wiki/Fermat%27s_Last_Theorem), which took centuries to settle. The math behind ANNs is likewise simple, but large questions remain open - hopefully it won't take as long!

    A related example, but of a different type, is the [Mandelbrot set](https://en.wikipedia.org/wiki/Mandelbrot_set) equation:

    <math xmlns="http://www.w3.org/1998/Math/MathML" style="margin: 1em; margin-top: 0; font-size: 1.5rem">
      <mrow>
        <mi>z</mi>
        <mo stretchy="false">&#x2192;</mo>
        <msup>
          <mi>z</mi>
          <mn>2</mn>
        </msup>
        <mo>+</mo>
        <mi>c</mi>
      </mrow>
    </math>

    Like the double pendulum, this is very simple math. But here it is well known that there is a lot to learn beyond the equation, things we cannot immediately see from the math itself (click to animate):

    <figure>
      <img 
        id="mandelbrot-gif"
        src="https://upload.wikimedia.org/wikipedia/commons/2/21/Mandel_zoom_00_mandelbrot_set.jpg" 
        data-animate="https://upload.wikimedia.org/wikipedia/commons/a/a4/Mandelbrot_sequence_new.gif"
        data-still="https://upload.wikimedia.org/wikipedia/commons/2/21/Mandel_zoom_00_mandelbrot_set.jpg"
        alt="mandelbrot set zooming" 
        style="border: 2px solid black; width: 60%; cursor: pointer"
        onclick="toggleMandel()"
      >
      <figcaption>
        <small>animation of zooming into the Mandelbrot set from <a href="https://en.wikipedia.org/wiki/File:Mandelbrot_sequence_new.gif">Wikipedia</a> (public domain)</small>
      </figcaption>
    </figure>

    I think it is fair to say that we understood something new the first time the Mandelbrot set was visualized like this.

    My point in this footnote is that even in math, there are various notions of what it means to "understand" an equation. Also, this is not purely a mathematical issue, because the advances in statistical mechanics led to [philosophical questions](https://plato.stanford.edu/archives/win2019/entries/statphys-statmech/) as well.

[^names]: This is not just anecdotal: LLMs are fairly robust against name changes, see e.g. section 4.2 in ["GSM-Symbolic: Understanding the Limitations of Mathematical Reasoning in Large Language Models"](https://openreview.net/forum?id=AjXkRZIvjB), Seyed Iman Mirzadeh, Keivan Alizadeh, Hooman Shahrokhi, Oncel Tuzel, Samy Bengio, Mehrdad Farajtabar. ICLR 2025. (Interestingly, that paper found LLMs were significantly less robust against numerical changes.)

[^gram1]: "*Can Language Models Handle Recursively Nested Grammatical Structures? A Case Study on Comparing Models and Humans*" Andrew Lampinen. Computational Linguistics, 1 Dec 2024.

[^gram2]: "*Language models align with human judgments on key grammatical constructions*" Jennifer Hu, Kyle Mahowald, Gary Lupyan, Anna Ivanova, Roger Levy. PNAS, 26 Aug 2024.

[^gram3]: "*Language Models as Models of Language*" Raphaël Millière. Chapter in ["*The Oxford Handbook of the Philosophy of Linguistics*"](https://global.oup.com/academic/product/the-oxford-handbook-of-philosophy-of-linguistics-9780198879640?cc=us&lang=en&#), 2026.

[^stat]: In particular, this is the sort of thing computer scientists want to understand. Consider the following assertion (variations of which are common online):

    > In the 'Crevyzik' example, LLMs handle arbitrary names using statistics and correlations, and we understand all that well enough, even if the details escape us.

    If so, then we can write this:

    1. LLMs only do statistics.
    2. But LLMs are Turing complete, and can implement any algorithm.
    3. Are all algorithms statistics?

    <p><!-- why is space needed here? --></p>

    The conclusion is obviously faulty, because the field of computer science is devoted to the study of algorithms: seeking new ones, analyzing their [theoretical complexities](https://en.wikipedia.org/wiki/Computational_complexity), and pondering large mysteries like [P vs NP](https://en.wikipedia.org/wiki/P_versus_NP_problem). This is a large field with many open questions. Collapsing all of that into "statistics, with unknown details" can't be right. So 1 or 2 must be wrong.

    2 may well be wrong: as mentioned earlier, there might be a gap between what the math of ANNs can theoretically achieve (any algorithm) and what ANNs do in practice. Perhaps ANN training only leads to statistical algorithms? It would be a wonderful result if someone managed to prove this one way or the other, but so far, no one has. That we do not know this - which of 1 and 2 is right, and there is a huge gulf between them! - proves we do not understand how ANNs work.

    If we lack a proof for 1, can we at least point to the preponderance of the evidence? I agree that small ANNs often appear limited to correlations and simple pattern matching. However, I am not sure about LLMs, since the "Crevyzik" example seems more easily explained in a non-statistical way, to me at least.

    But, more importantly, even if most of the evidence so far *were* consistent with statistics, that would not content scientists: all the evidence is consistent with [P ≠ NP](https://en.wikipedia.org/wiki/P_versus_NP_problem#Reasons_to_believe_P_%E2%89%A0_NP_or_P_=_NP), but it remains open. And, conversely, the evidence seemed to be in favor of [Erdős' planar unit distance conjecture](https://en.wikipedia.org/wiki/Unit_distance_graph#Number_of_edges), but it [turned out to be false](https://openai.com/index/model-disproves-discrete-geometry-conjecture/).

    With that said, it is a perfectly valid hypothesis that LLMs rely on their vast amounts of data and only do some amount of statistics on top. But it is one hypothesis among others, not a known fact.

[^new]: "*Bridging the human–AI knowledge gap through concept discovery and transfer in AlphaZero*" Schut, Tomašev, McGrath, Hassabis, Paquet, Kim. PNAS, 26 Mar 2025.

[^autogen]: Rather than just being shown games, an ANN can also generate them. That is the case with AlphaZero, which was trained by playing against itself. Without seeing a single human game, just using the rules of chess, AlphaZero explored the space of possible games independently and ended up with superhuman ability.

[^datahow2]: I said earlier that the data does not tell the ANN how to compress it, and in a previous footnote[^datahow] I mentioned that the data does not tell the ANN how to generalize, i.e., what to do on things outside of the training set. Generalization and compression are in fact closely related. The lossy compression just described also defines exactly how the ANN generalizes outside the data: when given a chessboard, no matter if it was in the data or if it is novel, the ANN counts piece values.

    It seems that compressing the data well, as opposed to memorization, should help generalize. However, the connection between [memorization and generalization](https://infinitefaculty.substack.com/p/memorization-vs-generalization-in) is complex and yet another thing that we do not fully understand about ANNs.

[^sql]: Another point the preprint authors make is to separate the data from the method of ***querying*** it. They compare a library of physical documents with a digitized version that can be searched using a query language, something like [SQL](https://en.wikipedia.org/wiki/SQL) (which can handle searches like "*find all documents containing the word 'apple'*"). The authors state that what is interesting to understand is not the physical paper in the first case, nor the sophisticated query language in the second, but the data. I agree.

    SQL, however, is not a mystery because we understand SQL already. It might be complex (perhaps even Turing complete), but it is a familiar technology, separate from the data. Many of us have used SQL on many datasets. But a particular chess ANN will use some query mechanism that we did not define and are not familiar with, and given the Turing-completeness of ANNs, it could be anything.

    Let's make this more concrete. Imagine someone builds a chess database, in which they store pairs of positions and outcomes, that is, boards and which player went on to win the game from there. The database uses SQL, which makes this a useful system already: we can query things like "*among positions where one player has a rook and the other has two knights, and no other pieces but the kings, who won?*" (Of course, we would need to write that out formally in SQL.)

    Now, imagine the database developer goes a step further and makes the system handle *unfamiliar* positions, that is, ones not in the data: given any query that we submit, the program automatically augments it, internally generating a more complex query and executing that, but without showing us that full, final query. Specifically, the augmented query does the following: when a position is unfamiliar, it does brute-force lookup of all possible moves, 2 levels deep. If all those calculated moves end up in a victory for one player (either in the database, or a literal checkmate), it reports that outcome, and otherwise a draw.

    This makes the overall system more useful in some ways: we will often get valid results not just for familiar positions, but ones close to them or close to a checkmate. That is a much larger set, and if we use this database as a chess engine in a tournament, it will do better. (Of course, this change also makes the system *less* useful in other ways, like for accurate analysis of historical data.)

    Given such a system, and given the response to a query, how much of that response is due to the data, and how much is due to the augmented querying mechanism? We can't tell. The position or positions we queried might have been in the data, but they might not, and the system does not indicate this to us. We also can't easily figure out what the querying does: in this example it calculates 2 moves ahead, but it could have been 3, or using some other approach entirely. In other words, this is a similar situation to a chess ANN: a system with algorithmic details that we do not immediately know (but can investigate).

[^reuse]: Chiang's entire piece is worth re-reading today, and one thing that stuck out to me is this:

    > I’m going to make a prediction: when assembling the vast amount of text used to train GPT-4 [the successor to ChatGPT, at the time], the people at OpenAI will have made every effort to exclude material generated by ChatGPT or any other large language model. If this turns out to be the case, it will serve as unintentional confirmation that the analogy between large language models and lossy compression is useful. Repeatedly resaving a JPEG creates more compression artifacts, because more information is lost every time. It’s the digital equivalent of repeatedly making photocopies of photocopies in the old days. The image quality only gets worse.

    > Indeed, a useful criterion for gauging a large language model’s quality might be the willingness of a company to use the text that it generates as training material for a new model. If the output of ChatGPT isn’t good enough for GPT-4, we might take that as an indicator that it’s not good enough for us, either. Conversely, if a model starts generating text so good that it can be used to train new models, then that should give us confidence in the quality of that text.

    While it is possible that AI companies make an effort to avoid training on low-quality 'slop', there are also examples of them *intentionally* training on high-quality AI-generated content:

      * Top AI companies see "distillation" - competitors training models using outputs from their own - as a [strategic danger](https://www.anthropic.com/news/detecting-and-preventing-distillation-attacks).
      * Production ANNs are trained using artificial data from other models, such as [cars trained using ANN-generated worlds](https://waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation/).
      * Scientifically, the ["Textbooks Are All You Need"](https://www.microsoft.com/en-us/research/publication/textbooks-are-all-you-need-ii-phi-1-5-technical-report/) approach compares an LLM trained on raw data versus on LLM-generated "textbook-style" content about the topic. The compressed summary, in this case, was better input.

[^hap]: Certainly LLMs often behave haphazardly, but - as in the links we have seen - plenty of research shows them solving tasks in methodical ways.

[^literal]: To be clear, I do not think the original authors of the Stochastic Parrot metaphor do so. There is a lot of nuance in their [recent](https://medium.com/@margarmitchell/no-ai-is-not-a-stochastic-parrot-a99e57766bed) [writings](https://medium.com/@emilymenonbender/stochastic-parrots-frequently-unasked-questions-49c2e7d22d11). But I see other people misinterpreting the original paper (and even the two links in this footnote) as proof that we understand LLMs fully and that they do nothing more than statistics.

[^electricity]: Here is [another way to put it](https://www.verysane.ai/p/do-we-understand-how-neural-networks):

    > When we say [neural networks are] "just statistics", it is correct but misleading. Most statistics people think of are simple, one or two numbers. Lots and lots of connected statistics, all at once, are not a simple thing at all. The neural network is just statistics in the same way that it's just electricity.

[^agree]: For example, one of the authors' many valid points is this, something I very strongly agree with:

    > the absconding of scientific duty by those who purport to use ANNs to replace human participants becomes blindingly obvious as such a dereliction in light of our exposition above.

    It makes no sense to conduct surveys and psychological experiments on ANNs and to think the results represent actual humans. That scientists might do this is, frankly, horrifying.

[^leap]: That this particular leap is unsound does not rule out the conclusion, of course. As I mentioned before, very smart people have debated machine consciousness from different positions. I linked to the history of the Chinese Room Argument as an example because I think the [encyclopedia article](https://plato.stanford.edu/entries/chinese-room/) does an excellent job of covering the discussion, but of course there is a lot more that is relevant, from [Dennett's denial of qualia](https://en.wikipedia.org/wiki/Consciousness_Explained) to [IIT's attempt to mathematically model experience](https://en.wikipedia.org/wiki/Integrated_information_theory). Personally I find all this interesting but also far too speculative for me to have a firm opinion on. In any case, nothing scientific turns on this.

    Incidentally, to the best of my understanding, in Dennett's view the leap from "we don't understand ANNs" to "ANNs might be conscious" is actually sound! If consciousness is a matter of information, not qualia or metaphysics, then whether a machine is conscious actually *does* depend on which algorithms it uses to process information. If we don't know those algorithms then they might be ones that, in Dennett's position, *do* qualify as "conscious." (I don't find Dennett's position terribly convincing, though.)

[^welper]: "*Universality of gradient descent neural network training*" G. Welper. Neural Networks, Jun 2022.

[^ahn]: "*Transformers learn to implement preconditioned gradient descent for in-context learning*" Kwangjun Ahn, Xiang Cheng, Hadi Daneshmand, Suvrit Sra. NeurIPS 2023.

[^cot]: "*The Expressive Power of Transformers with Chain of Thought*" William Merrill, Ashish Sabharwal. ICLR 2024.

[^cot2]: "*Chain of Thought Empowers Transformers to Solve Inherently Serial Problems*" Zhiyuan Li, Hong Liu, Denny Zhou, Tengyu Ma. ICLR 2024.

[^cot3]: "*Looped Transformers as Programmable Computers*" Angeliki Giannou, Shashank Rajput, Jy-yong Sohn, Kangwook Lee, Jason Lee, Dimitris Papailiopoulos. ICML 2023.

<script>
  function togglePendulum() {
    const img = document.getElementById('pendulum-gif');
    if (img.src === img.dataset.still) {
      img.src = img.dataset.animate;
    } else {
      img.src = img.dataset.still;
    }
  }

  function toggleMandel() {
    const img = document.getElementById('mandelbrot-gif');
    if (img.src === img.dataset.still) {
      img.src = img.dataset.animate;
    } else {
      img.src = img.dataset.still;
    }
  }
</script>

