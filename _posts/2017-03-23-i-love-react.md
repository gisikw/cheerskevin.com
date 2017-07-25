---
layout: post
title: "I Love React"
date: 2017-03-23 21:43:08 -0500
---

> Author's Note: I'm currently in the process of migrating old blog posts to this new system. That may mean some links, syntax highlighting, and other details are broken or missing temporarily. Sorry for the inconvenience!

React is an amazing JavaScript framework, and one that kinda transformed how many people think about making websites.

A lot of websites *do* things nowadays, right? Like there's a chat, or a notification feed, and fancy crap like that. The web wasn't initially designed with that in mind. The idea was that you'd visit a URL, and then you'd HAVE THE THING!

> "Hurrah! I went to Facebook.com, I got my Facebook, and I'm done, lemme send that off to the printer and I'll read it at my leisure"

k, so then when people wanted to stay on a page for a long time, we had to write extra code. This code was basically:

> "Crap, they're still here. stuff.addToPage();"

Except then y'all got fancy and wanted to access email and crap, so we had to start doing "stuff.removeFromPage();" too. All because you were too lazy to click the damned refresh button. Shame on you!

## An Analogy

Four people walk into a bar and order a beer. Two pay their tab and leave. Three more people walk in and order a beer. All the patreons bemoan the lack of a joke here.

How many beers are at the bar?

### Old-Fashioned Web Code

> drink.pour();

> drink.pour();

> drink.pour();

> drink.pour();

> drink.remove();

> drink.remove();

> drink.pour();

> drink.pour();

> drink.pour();

> ...whatever the bar looks like after doing these things

### React Code

> The same number as the number of people drinking, obviously. Who cares what happened before or after? It's irrelevant

That is to say, React is build on one particular concept:

`user interface = f(data)`

The interface displayed is *always* based on some underlying data. If we coded up some underlying website, we might decide that `f` in this case would be a function that counts the number of drinking people, and renders out a little beer image for each one.

We don't care how we *got* there. We care what the *current* state is. We express how the website should look based on how many ~~alcoholics~~ responsible drinkers are drinking *now*.

# I lied before

Okay, so this isn't really a React thing. Don't get me wrong --- I still love React for making this kind of thinking easy. But here's my real love: *pure functions*

Pure functions are really cool. You know how in math class they sat there and made you deal with the `y = mx + b` stuff, and then changed it to `f(x) = mx + b` for no real reason? That was dumb. And isn't really related to this. But here's a cool thing. `f(x)` doesn't have to give back a number. It can give back a word! Or a list! Or a toaster!

Here's the rule for pure functions: a given input must always result in the same output. That's it. `f(1) = Toaster! ` `f(2) = Bagels! ` `f(1) = Toaster! ` Cool. Pure function!

Going back to our earlier example, we might write some code for our bartender that looks like `f(2) = 2 Beer Emoji! f(3) = 3 Beer Emoji!` It's the bartender's job to change the *input* value, but he never has to worry about how to emojify the number of beers on the counter ever again!

...this metaphor is dying a slow and horrible death

## The point is that React is really cool because it made it easy to think about procedural logic in a different way

Programming, at it's core, is about describing procedures. It's not "what is the 53875831th prime number", but "how do you reliably find a prime number?". We used to write those programs in procedural format ("step one: start with one. step two: is that divisible by anything? step three: increment the number by one..."). Then we all started getting obsessed with objects ("create a pretend little universe with a bunch of little tiny programs that interact with each other. Let the universe run for a while and hope it tells us the answer"). Now, we're moving toward a new approach - the idea of expressing things in terms of pure functions - things that can transform an input into an output.

It's a really simple idea. It's the same as a waiter/waitress transforming your order into the actual food. Yes, she may have to go to the kitchen, and yell at Mike for overcooking the salmon, but that's all hidden away from us. We know that if we place our order, we expect a certain deliverable.

All of which is to say, I'd like to start using React to generate blog posts on this site. So, yeah, keep an eye out for that.

[&larr;Previous Post](/what-is-this-rubbish.html)
|
[Next Post&rarr;](/lets-start-reacting.html)
