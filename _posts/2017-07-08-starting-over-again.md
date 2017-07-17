---
layout: post
title: "Starting Over Again"
date:   2017-07-08 19:00:33 -0500
---

Welp, here we are. Again. I'm redoing the site yet again. Let me explain...

## Build your own thing

I like building our own tooling. It's fun. It helps develop a better understanding of what it is that we're making. But it has a downside, in that you have to maintain it.

When I relaunched cheerskevin.com, we started with a .txt file. Then we migrated to basic HTML; added some CSS and JS; built some Bash, and eventually Node and Ruby tools. It was fun. And the idea was that everything would exist as it existed when we wrote it. We could incrementally develop new things, while maintaining backward compatibility.

That was, erm, dumb.

## Legacy sucks

Here's the thing: we could do that. We could make a website that's backward compatible and supports all our old systems, and keep the ideas in our head. But then every time I go to write something, I'd need to worry about what that might break. Just worrying about getting a homepage set up has kept me from putting out anything on the site for months.

**It's not that building our own stuff isn't cool. It's that it adds friction to writing real things**

## Back to dumb ol' Jekyll

I'd still like to entertain the fantasy that we'll roll our own system again, but I'll tell ya this: the content is going to be Markdown files. That's it. Period. Because we can carry those around. They're portable. We have to worry about links and stuff, sure. But we're not gonna try and let every single page maintain its old style and format. We're using Jekyll-style Markdown posts with YAML frontmatter.

And for now, we're going back to Jekyll. No fancy Bash filetype detectioney stuff. No React static rendering fancypants stuff. Plain old static files of boringness. The details got in the way. And I stopped writing stuff. I like writing stuff. It's fun.

## Eventually...

- Migrate the old posts into the Jekyll posts directory, adding little editorial notes where things are no longer relevant (like "look at the design of this page!")
- Create our own Jekyll theme - I did like how our post page was. Would like to port that over, eventually
- Port over the OLD old posts - those still exist too, and it'd be nice to bring them back
- Maybe, *maybe* some minor tooling. But the tooling should probably be stuff that just helps populate the frontmatter. Not more junk that has to go in the build step. I dunno.

## Listen to the friction

Two key things here, both to do with friction:

First, pay attention when friction is guiding you away from doing things you intended to do. I wasn't updating this site at all, even though I intended to. Lack of mindfullness is bad.

Second, let friction guide where we focus future efforts. I'm gonna be annoyed at the lack of comments, or at worrying aobut synchronizing content between machines, or maybe something else I don't know. But I'm not gonna prematurely optimize on that. We'll work on adding site features as they become annoying, and not a moment before.

Okay, that's all. Rambly Kevin is rambly. But hey, in my defense, I'm a bit rusty.
