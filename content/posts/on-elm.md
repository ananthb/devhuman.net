+++
title = 'Elm makes me unreasonably happy'
date = 2024-10-04T02:24:35+05:30
tags = ['elm', 'webdev']
tldr = 'Write something in Elm, I implore you.'
+++

[Elm](https://elm-lang.org) is _still_ a delightful language
for writing reliable web applications.

<!--more-->

## Elm is for beginners

Elm is a well thought out language and ecosystem.
The tooling is straightforward and unfailing.
Their documentation game is on point.
The Elm Architecture is a complete game-changer for building reactive
web applications. Trouble believing me?; ask React and Redux how it's done.

Having no background in Functional Programming during my learning days,
Haskell enchanted me with its strange symbol-like code.
I was unable to get anywhere with it but my breakthrough came from Elm.
It's FP on easy mode. The starting tutorial guides you through a range of topics
such as program structure, serialisation, and interacting with the wider
non-Elm world.

## Elm is for debuggability

Elm's compiler lulls you into a euphoric state of confidence,
whose absence you feel as soon as you leave it behind.
The error message design [inspired Rust](https://blog.rust-lang.org/2016/08/10/Shape-of-errors-to-come.html)
to do better.
The compiler is pointedly structured to help you write correct code.
It also compiles to an efficient JavaScript representation
with (basically) no runtime exceptions. If you use regex or non-JSON ports,
then you're on your own. This is such a dramatic difference from JavaScript's
runtime errors, that Elm feels alien at first.

Error messages are fantastic, as I've mentioned already.
They're verbose, contain links to documentation, and usually
include the correct code you need to proceed.
At the least they contain pointers to fixing your code.

The other bombshell feature of the Elm compiler that blew me away
was time-travel debugging. I'm not sure if this is a thing anywhere else,
but why the hell isn't it?
Since Elm is a pure functional language, you can step your application's
state forward and backward to any representation.
This includes everything from the data to the rendered HTML.
Most of the time you don't even end up using it because if your program compiles,
then it's probably correct.

## Elm is for fun(ctional programming)

Elm introduced me to the FP way of thinking that I feel broadened my thinking
as a programmer. Once I acclimatised, I found myself wondering why FP wasn't
introduced to me sooner. Especially for reactive UIs, structuring your entire
program as a state machine is a delightful way to reason about your code.
The type system and compiler always get in your way for quick refactors,
but you realise that they're in your way because your code is wrong.

Having well typed GraphQL and JSON decoders makes webapps around 50000% more reliable.
Citation needed, but it sure feels like it.
What I came to love about FP is how your code ends up reading like
the solution to a problem rather than a series of steps to get there.
It brings great joy to use `|>` and `>>` to compose function and data pipelines.

## Elm might be dead

Elm has been on version 1.19.1 for a while now.
Stability is (was?) valued highly in the Elm community so this by itself
isn't cause for alarm.
Besides one typing soundness bug in 0.19, Elm doesn't have a lot of issues.
You can read plenty of criticism of Elm's typesystem on the internet.
Season Haskell developers are unhappy with the lack of typeclasses.
But Elm isn't a Haskell replacement, it's a JavaScript replacement,
and in that lens it does an excellent job.

The real reason Elm is probably dead is that its not profitable to
create and run Elm. Running programming platforms is as thankless and
unforgiving as running restaurants.
Evan spoke about his struggles monetising Elm this year:
<https://www.youtube.com/watch?v=XZ3w_jec1v8>.
You may watch it if you will, but its a sad story
with little solace at the end.
