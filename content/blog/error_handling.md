+++
title = "Advanced Rust error handling"
date = "2024-06-24"
description = "A short blog post about idiomatic error handling in Rust with anyhow and thiserror"
taxonomies.tags = ["rust", "anyhow", "thiserror",]
+++

## Intro.
The starting point to learn about error handling is, of course, the Rust book:
chapter 9 gives a clear overview of how error handling works and the difference
between recoverable and unrecoverable errors. Especially for larger projects
though, that's not enough, so this blog post starts from there to then go a
little bit deeper on error handling and how to do it in a more advanced way, by
leveraging both the standard library and the ecosystem.

One of the great things about (safe) Rust is the absence of segfaults[^1] and
exceptions, the bane of my existence, and probably of many other people: anyone
with a bit of experience could tell good horror stories about hours of debugging
because a hidden segfault or exception coming out of the blue popped up somehow
in production. However, error handling in general is a complex topic and in Rust
it's also an ongoing process[^2], so we will go through what the current best
practices are.

## Requirements.
There aren't many requirements on your side to be able to follow this post.
It's probably enough if you already went through the book and wrote a few lines
of rust. However, you shouldn't worry too much about it if you're a beginner: in
my case, I had already written thousands of lines of Rust when I started digging
deeper on this topic.

## Not to panic! or Not to panic!
Nope, it's not a typo, I didn't misquote Shakespere, I'm just quoting chapter
9.3 of the book, which quotes Shakespere in its title, but then goes ahead to
list the very few cases when it's fine to panic! (or unwrap, or expect):
1) Examples
2) Tests
3) Cases in which you have more information than the compiler

And that's pretty much it. The book also mentions prototypes, but I disagree.
To be very clear, in my opinion, if your production code is panicking
it should probably be considered a bug.
First of all because it's too convenient to split a Rust application in various
modules and/or crates and by doing so we want to achieve reusability. To achieve
reusability we have to design modules in such a way that we make no assumptions
on where those are used. There are cases where it's just not acceptable to
panic[^3]. In general, it's also really not nice to publish code that crashes
the caller's code.
Secondly, if we are writing a user facing application, it's also not nice
to crash on the user's face, it's better to show an error message and return
to a state where the user can interact with the application. If really the
application needs to be shut down, we should consider how to gracefully shut
it down.

## The lazy way.
Let's assume we are working on a quick prototype, a proof of concept where
delivering something that works in the shortest possible time, one might be
tempted to fill the code with unwraps at any Result or Option returned by the
dependencies called. If we don't take into account help from the IDE or any AI
copilot-like thing, the cost of adding a single unwrap is exactly 9 keystrokes.
Can we do better? Can we use the ? operator to "brainlessly" tell the compiler to stop annoying?
Turns out we can. My approach when I need to go fast is to include [anyhow](https://docs.rs/anyhow/latest/anyhow/) in my project (`cargo add anyhow`), 
___
[^1]: Well... except for [bugs in the compiler](https://github.com/Speykious/cve-rs)...

[^2]: To know more about this you can start by looking [here](https://github.com/rust-lang/project-error-handling).

[^3]: One notorious example is the linux kernel, see [this discussion](https://lkml.org/lkml/2021/4/14/1099).
