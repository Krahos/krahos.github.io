+++
title = "Test driven development in practice."
date = "2024-06-24"
description = "Some notes on how to integrate best practices in the software development workflow"
taxonomies.tags = ["rust", "domain-modelling", "test-driven-development", "domain-driven-design"]
+++

## Intro.

The purpose of this post is to present what test driven development (TDD from
now on) is and what are the advantages to apply it in a rigorous manner. I will
also touch several topics that are somewhat related to it, in order to give a
general overview.

I read several blog posts, listened to endless presentations and even attended
workshops on the matter, but TDD really clicked in my head when I actually
forced myself to use it. The content of this post is about what I understood
so far about it and it wants to be the starting point for you, then, if you're
convinced it's useful, open your text editor and try for yourself. That's really
the best way.

I'll go first through the theory, to at least understand what we're talking
about, then I'll cover the practice. The examples will be in Rust, but they
should be easy enough to be understood even with 0 Rust experience and you can
(and should) follow by writing the code step by step in any language you prefer.

## What is TDD about, really? And why should we care?
First of all, testing is a natural and mandatory endeavour while writing software, think about
it: what's the first thing you did right after you wrote your first program?
Well, I bet you probably ran it to see if it worked, aye? And then you wrote
your second program and did the same. Then, if you wrote enough software, at
some point you wrote that big(ger) program where simply running it and manually
inspecting if the behaviour was correct, wasn't efficient enough any more, so
you either spent too much time looking around and solving bugs, or you simply
gave up on them.

Software engineering in general is the discipline to manage the development of
software and to ensure its reliability throughout its lifecycle. And software
testing is just one of software engineering subtopics.

In particular, we talk about TDD when, as the name suggests, tests give us the
direction to follow while writing software: in other words, it's the technique
of writing tests *before* writing the actual code. This is generally the workflow:
- Write the test
- See the test fail
- Write the necessary code to make it pass
- Refactor, and clean up the code, while keeping that test passing

It's fundamentally important to see the test fail at the beginning: if you don't
see it fail, there is a non-zero chance it's garbage. Look at the following:
```Rust
#[test]
fn my_greatest_test() {
  my_probably_buggy_function();
  assert!(true);
}
```
- Does it run the code? Yes
- Does it have an assertion? Yes
- Is it any good at all? No!

This test never fails, regardless of what we write in our function, so we cannot
realy on it at all.

Why write the tests before the code you want to test though? If you first
write your program and only then write the tests, you're going to be biased
by what you have already written. This way, instead, you see what the user of
your program is going to see for the first time. You will be unbiased and look
for the highest quality possible. it's just like being a kid again and write a
letter to Santa:
```
Dear Santa,
since I've been so very good this year, I'd love to have a function that takes
an array of numbers and gives me back to me with its elements sorted from the
smallest to the largest.

Thanks a lot, yours,
Seb.
```
Then, sadly, we are big boys and gals now and we are our own Santas, so we have
to write our own code and satisfy our own wishes, don't we? Maybe not for long:
have you ever heard of that thing called GenAI, all the cool kids are talking
about nowadays? Just a speculation from someone ignorant on the matter, but
maybe this could be the way to use it if it ever becomes good: you write the
tests, it writes the code to satisfy those tests. But in order to do this and in
order to still deliver high quality software we need to be good at writing tests
and we need to be good at refactoring ugly code. Infact this is what I will do
for the examples provided.

## Our first TDD attempt.
For our first practical example, let's indeed take the letter to Santa we wrote
above. Let's start a new project from scratch and let's write our favourite
implementation of a sorting algorithm. But... Wait! we are doing TDD! We are
not going to even think about the implementation yet. We want to translate the
requirements into a test first.
```Rust
/// sorting.rs
#[test]
fn my_sorting_fn_works() {
  // Our input.
  let mut arr = vec![3, 6, 2, 4, 9, 1, 5];
  // Our expected output.
  let sorted_arr = vec![1, 2, 3, 4, 5, 6, 9];
  // We run the function we want to test.
  sort(&mut arr);
  // We assert to check if it worked.
  assert_eq!(arr, sorted_arr);
}
```
Now that we wrote the test, we have to at least write the minimal function
definition, so that we are able to run it.
```Rust
/// sorting.rs
fn sort(arr: &mut [u8]) {}
```
Our function does exactly nothing, but the test is running and it's failing
as we expect and want. At this point, we could also thing about the API: do
we like it like this? Maybe we would prefer to call our sorting function as
`arr.sort()`, for example. So we can really shape the way our code presents
itself even before writing it. In this case we are fine the way it is.

We're really lazy, so let's ask some GenAI model to generate a code that passes
the test. By passing the test and the function definition and asking for bubble
sort, this is what I got:
```Rust
/// sorting.rs
fn sort(arr: &mut [u8]) {
    let len = arr.len();
    for i in 0..len {
        for j in 0..len - 1 - i {
            if arr[j] > arr[j + 1] {
                arr.swap(j, j + 1);
            }
        }
    }
}
```
This passes the test already and I don't see any meaningful refactorings we can
do, it's a simple example anyway, so we can be satisfied.

## Dealing with GenAS, or GenAE.
Is our test really that good though? What if we are letting our artificial
stupidity generate the code? And what about artificial evil? Can we rely on the
test we wrote? Not really! Look at this implementation:
```Rust
/// sorting.rs
fn sort(arr: &mut [u8]) {
  let sorted = [1, 2, 3, 4, 5, 6, 9];
  arr.copy_from_slice(&sorted);
}
```
Our test is passing, however, it didn't save us from the stupidity and/or evil
of this implementation. We can agree that our test was garbage to begin with.
Now, you can imagine this doesn't happen, nobody is really that evil or stupid and, sure, it doesn't for problems as
simple as this. But there are cases where the situation is not so clear, 

## Overall architecture.

Almost every program is a mix of inputs, outputs, business logic, external
dependencies, data structures and operations on those data structures. These
are often mixed and matched together without a particular criterion by both
beginners and experts, however, finding a structured way to treat them will
make our life easier later on, so I want to briefly describe my approach.
The architectural pattern I find working for me is nothing new and it's based
on some already well known theory: onion architecture, clean architecture,
hexagonal architecture... Call it however you want, but it basically boils down
to this diagram:

![Clean Architecture](/blog/arch.png "Clean Architecture")

This is a very general idea that can guide the construction of most software
applications: if you think about the classical backend service or a mobile /
desktop application, a daemon, some firmware, they can all be built having this
architectural patern in mind.

In the I/O block we will put the logic that allows external systems to call
into or interact with our application. It could be an HTTP API, it could be a
TUI/GUI (yes, the user counts as an external system) or it could even not be
there, for example in the case of a daemon.

In the business logic and domain blocks we will describe the problem the
application is trying to solve and there is a fundamental difference between
them: the domain layer never interacts with external systems (nor with anything
other than itself, really), while the business logic layer might.

Finally, it's important to distinguish the layer through which our application
interacts with external systems: be it the file system, a database, an HTTP
endpoint... anything our application actively needs to call. And it's also
fundamentally important to stress that this layer has to be always be called
through an interface (or equivalent construct in the language you're using).

## Testing strategy.

We structured the application this way mainly because this will make our life
easier when writing tests. Let's go layer by layer to see the testing strategy
adopted.

In the domain layer, we have no external dependencies, so we can abuse textbook
unit testing.

In the bysiness logic layer, we call external dependencies, but we do so through
interfaces, so we can again abouse unit testing, with the added burden of
mocking all those interfaces.

In the external connection layer, we will mostly do integration testing: we want
to see if our application crashes when the file system goes crazy, what happens
when the database doesn't answer and so on.

Finally, in the I/O layer we will also mostly do integration testing, since we
want to see if our entire application, together with external services, behaves
as expected.
