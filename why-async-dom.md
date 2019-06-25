There have been multiple [recent proposals](https://github.com/chrishtr/async-dom/blob/master/current-proposals.md) to add asynchrony to the DOM. They are aimed at different subsets of the problems of "slow rendering". "Slow rendering" generally falls into three categories:

1. "Slow rendering" could happen because the browser or web page is inefficient. For example, maybe the browser has a bug leading to slow layout in certain cases, or it scales poorly with the number of composited layers. Or the web page has too much DOM, or an overly complicated structure that leads to slow algorithms.

2. "Slow rendering" could also happen because (user input and) rendering are delayed due to other work happening that uses up the CPU. Examples of this include time spent running GC, decoding an image, or running a slow ad analytics script.

3. Finally, "slow rendering" can happen because there are simply too many things changed in the DOM or application state since the last render to expect it to update quickly. A good example is a global change of font size or page width that requires a complete re-layout of the entire DOM. Another is a transition of a large part of the page to a new rendering state.

If we're in one of the above situations, what should be done? One course of action coud be to try to address part of the problem. If #1 is the problem, it's often simplest if the inefficiency can just be fixed directly, such as by optimizing layout or the size of the DOM. At a high level, that sounds straightforward. If #2 is the problem, then the solution may be to do non-rendering work on a different thread, or run it at a lower priority than rendering work. Seems harder than #1, but potentially doable.

If #3 is the problem, then the solution becomes more murky. If the problem is "DOM changes too much", one possibility is to simply make the DOM smaller, to limit the scope of the problem. Another is to try to structure the DOM into pieces that can be processed independently, and avoid touching too much at a time. If the problem is "application state changes too much", then similar approaches can be applied to the script code that renders views of the state (make it smaller, break into independent pieces).

However, these aren't the only solutions to problem #3! Other ideas include:
 a. Using more CPU cores to do work in parallel
 b. Predicting future work and doing it in advance
 c. Breaking the DOM into pieces, and updating the pieces asynchronously from each other

** Now what?

There are a bunch of issues bound up here that make the above ideas challenging to follow up on. The issues include that there is a single monolithic DOM for each frame, that the timing & implementation of its rendering to the screen is implemented by the browser (not the developer), that JavaScript is a single-threaded programming language, that the DOM APIs are not designed for multi-threaded access, and that CSS is a very expressive, wide and powerful API, and therefore quite challenging to implement efficiently.

But these are not all the issues! This does not yet touch issues on the developer side, such as that web applications use frameworks which [provide value](https://rendering.chrishtr.org/2019/01/what-is-value-of-virtual-dom.html) by abstracting *away* from the details of the DOM, that business realities limit how much work can go into optimizing a single web app, that there are multiple browsers -- not to mention multiple versions & variants of the same browser -- to contend with, that browser performance bugfixes or APIs may not be deliverable on timeframes that are acceptable to a business, and that features sell (or are perceived to sell) web apps more than performance does.

What is needed is solutions that address the problem of "slow rendering", but that strategically minimize the issues mentioned above. Desirable properties of solutions include:

* Minimimal difference in API and behavior from the existing web APIs
* Does not preclude any use cases
* Easy enough to adopt
* Admits an incremental path of launch & iterate from a first MVP version to much more advanced ones
* Compatible with established and popular ways of writing web apps
* Provides progressive enhancement on top of rendering approaches that work with browsers that do not support any new APIs
* Makes it easier - not harder - to build web apps.

** Why not just make the browser faster?

A legitimate question is: why not just make the browser faster/smarter? After all, a rising tide lifts all boats, and a sufficiently fast computer makes all programs fast. Similarly to the idea that making the hardware of a computer faster over time is a good approach to these types of problems, we should make the application platform becomes faster over time also.

This argument is particularly enticing for the web because of the inversion-of-control programming model, whereby the developer provides content to the browser, and the browser can use any number of clever strategies to optimize how that content gets to the screen. It's surprisingly hard to come up with examples where a sufficiently clever browser can't in principle optimize away that content that is offscreen, or notice that there is a shortcut to the layout algorithm in the particular pattern of mutations in a given example, or add a caching layer at just the right place to avoid some expensive work.

However, there is a question of feasibility: how long will it take to implement these fancy optimizations? Are they over-fitting for special cases? How hard will it be to implement these optimizations without bugs? Will all browsers be able to implement the fancy optimizations equally? (These drawbacks were also mentioned or hinted at in the previous section.)

Another counterpoint is [Wirth's law](https://en.wikipedia.org/wiki/Wirth%27s_law), a version of which is "sofware will grow to fit the space available" - in this case space meaining performance budgets. This means that even if the browser optimizes to make one flexbox faster, the UI of web apps over time will add in a second flexbox, because it's convenient to do so, and solves the business problem at hand.

It does make sense to make the browser faster and more scalable over time, by using a better [rendering code architecture](https://github.com/chrishtr/rendering/blob/master/rendering-architecture.md), by utilizing available [technologies](https://github.com/chrishtr/rendering/blob/master/rendering-technologies.md) such as GPUs, threading and process concurrency, multi-core machines, clever algorithms, and caching. Such work is actually necessary to make it *possible* to make the fancier and more immersive UIs of the future. But it doesn't really answer the question of Wirth's law, or in general the complex ecosystem effects resulting from the huge diversity of the web and the fact that ultimately the web is for expressing information, not worring about optimizing code.

In addition to making the browser faster and adding some APIs to solve particular common UI use cases, we also need techonologies to help web apps *manage scale efficiently*. This is what the space of Async DOM is about.

** The problem of coordination. Analogy with pre-emptive multitasking

Before the advent of pre-emptive multitasking for Windows and MacOS in the mid-90s, those two dominant operating systems used a cooperative multi-tasking approach to running multiple programs simultaneously on the same computer. 

