---
layout: post
title: "Unburying the Lede"
date: 2017-04-08 21:43:08 -0500
---

> Author's Note: I'm currently in the process of migrating old blog posts to this new system. That may mean some links, syntax highlighting, and other details are broken or missing temporarily. Sorry for the inconvenience!

[In the last post](/lets-start-reacting), we found a way to create static pages from React components. And in theory, we've defined a system by which we can create static pages from *any* source material. A file called `coolPage.html.md` will be processed by our `md` processor, but a file called `coolPage.html.whatever` would be equally valid, so long as we had a `whatever` processor.

One of the pages I'm most eager to build --- which shouldn't come as much of a surprise --- is the homepage! Ideally, it should contain some extracts from recent posts and links to things. But here we run into a problem with our fancy file processor system:

**If we don't use a consistent format for our blog posts, how do we extract the relevant information for the homepage?**

Okay, actually, this is two problems: getting the metadata for posts, and then using that metadata to make a nice homepage (or RSS feed, or archive pages, or anything else). Today, we're just gonna tackle the first part.

## Solving the first problem first

Up until now, this site has worked on a folder structure. You can append `/ls` to any directory and see all the files in it. Most of the posts have links at the bottom to previous and next, but we don't have any sort of centralized timeline of posts.

Until now.

It's kinda unavoidable. If we want to have a "recent posts" section; if we want to have an RSS feed; if we want to set up a mailing list one day...posts need to exist *in time*, and not just as random files on a website.

So! We need a single file that we can use to drive these things. It needs to have enough information about every post that we don't need to go hunting around in additional files to find stuff. For example: maybe we'll want to use the lead images as part of a clickable thumbnail on the homepage. That means a link to each posts' image needs to exist in our manifest.

I'm going to use [YAML](http://yaml.org/) as our file format to store this data (which is the same format we use in our Markdown files to specify details about the posts, incidentally). Here's an example of what I think our `manifest.yml` might look like:

```yml
---
- title: Unburying the Lede
  url: https://cheerskevin.com/unburying-the-lede
  image: https://cheerskevin.com/lead-images/html-example.jpg
  color: rgb(65,64,65)
  lede:
    In the last post, we found a way to create static pages...
- title: Being with Anger
  url: https://cheerskevin.com/being-with-anger.html
  image: https://cheerskevin.com/lead-images/meditation.jpg
  color: rgb(92,101,72)
  lede:
    Every so often – perhaps a few times a week – I suck in my lower lip
```

So, we need to get the title, the url, the image url, the theme color (I feel like we might want to use this, so I'm adding it here), and some initial text of the post. That feels like it ought to be difficult, but we're going to cheat a little bit: we're going to use Pismo.

## What is Pismo?

[Pismo](https://github.com/peterc/pismo) is a RubyGem that does some pretty clever extraction of information from webpages. So far on the blog, we've talked HTML, CSS, JavaScript and Bash, so I'm not particularly eager to add Ruby to the mix. Fortunately, Pismo has a command-line interface, so we can use good ol' Bash. Let's test it out:

```bash
$ pismo https://cheerskevin.com/lets-start-reacting title lede url
---
:url: https://cheerskevin.com/lets-start-reacting
:title: Lets Start Reacting
:lede: Okay, so I said before that I wanted to start using React to build pages. Why bring in another language instead of continuing to write stuff in Markdown? Good question! First off, Markdown is great.
```

Nice! It extracted the actual text, rather than the actual HTML markup! There are some leading `:`s on the keys that I don't like that exist for Ruby reasons we're not gonna go into, and there's some tidying we need to do to make it match our output, but we can apply some command line magic to clean that up:

```bash
$ pismo https://cheerskevin.com/lets-start-reacting title lede url | awk '!/---/' | sed 's/^://; s/^/  /; s/lede:/lede:\n   /'
  url: https://cheerskevin.com/lets-start-reacting
  title: Lets Start Reacting
  lede:
    Okay, so I said before that I wanted to start using React to build pages. Why bring in another language instead of continuing to write stuff in Markdown? Good question! First off, Markdown is great.
```

Okay, to be fair, that's some messy Bash. First, we use `awk` to say "skip the --- line". Then we're doing three different find-and-replaces with `sed`. First, we replace "<start of line>:" with nothing. Then, we replace "<start of line>" with two spaces. Finally, we replace "lede:" with "lede:<newline>    ". It's a little messy, but it works!

We're missing the image and color keys from our output, however. It's time to roll up our sleeves and build our own crawler.

## Rolling our own stuff

Pismo did the hard work; now we're just going to have to do some naive pattern-matching. We'll say that the Pismo output will be the second-half of our record, which means we need to generate the first part ourselves:

```yaml
- image: https://cheerskevin.com/lead-images/html-example.jpg
  color: rgb(65,64,65)
```

The key order doesn't matter in our record, so we can put the starting `-` on the image, and just throw the Pismo output underneath. So let's work on actually getting that output!

We need to start with `curl`, which will get all of our HTML content. We'll pass it the `-s` flag to make sure it doesn't print out any extra stuff.

```bash
curl -s https://cheerskevin.com/lets-start-reacting
<!DOCTYPE html>
<html lang='en'>
<head>
<meta charset='utf-8'>
...
```

...I'll spare you the rest. We need to filter down the response to just the lines we need. For that, we'll use `awk`:

```bash
curl -s https://cheerskevin.com/lets-start-reacting | awk '/^<meta.*(theme-color|og:image)/'
<meta property='og:image' content='https://cheerskevin.com/lead-images/jsx-splash.jpg'>
<meta name='theme-color' content='rgb(215,208,203)'>
```

Much more manageable! The image URL and the color are nestled in those tags. Time to bust 'em out! Purists might have some fancy way to do that by modifying the `awk` command, but we'll use `sed` here.

```bash
curl -s https://cheerskevin.com/lets-start-reacting | awk '/^<meta.*(theme-color|og:image)/' | sed "s/.*content='\(.*\)'>/\1/"
https://cheerskevin.com/lead-images/jsx-splash.jpg
rgb(215,208,203)
```

We have the values! Now we just need to read them into varibles so we can write them out with their keys. To do that, we're going to wrap that whole command in a subshell, and read what it spits out. Take a look:

```bash
read image color <<<$(curl -s https://cheerskevin.com/lets-start-reacting | awk '/^<meta.*(theme-color|og:image)/' | sed "s/.*content='\(.*\)'>/\1/"); echo -e "- image: $image\n  color: $color"
- image: https://cheerskevin.com/lead-images/jsx-splash.jpg
  color: rgb(215,208,203)
```

We did it! Though wow that is a mess! Time to extract this to a proper script and clean things up.

## Finalizing the script

It's fun to explore Bash command-by-command, but it's hell to maintain it. Sweep up some of the mess and combine our own code with the Pismo stuff, and one last thing - the publish date! We'll create two fields: published_at and modified_at, just in case we want to come back to those later. Thankfully, we can use the `date` command to generate those for us on-the-spot. Here's the final script:

```
#!/bin/bash
URL=$1

# Make sure the post isn't already in there!
if grep -q $URL manifest.yml; then
  echo "That post was already added!"
  exit 1;
fi

# Lock the timestamp
TIMESTAMP=$(date -R)

# Grab theme color and image from HTML if available
read IMAGE COLOR <<<$(
  curl -s $URL |
  awk '/^<meta.*(theme-color|og:image)/' |
  sed "s/.*content='\(.*\)'>/\1/"
)

# Write content out to file
echo "- image: $IMAGE
  color: $COLOR
  published_at: $TIMESTAMP
  modified_at: $TIMESTAMP" >> manifest.yml

# Use Pismo to write extracted title, url, and description
pismo $URL title lede url |
  awk '!/---/' |
  sed 's/^://; s/^/  /; s/lede:/lede:\n   /' >> manifest.yml
```

After running `./publish.sh https://cheerskevin.com/unburying-the-lede`, and then running the script on the previous post, here's our lovely `manifest.yml` file:

```yml
---
- image: https://cheerskevin.com/lead-images/jsx-splash.jpg
  color: rgb(215,208,203)
  published_at: Sat, 08 Apr 2017 21:47:18 -0500
  modified_at: Sat, 08 Apr 2017 21:47:18 -0500
  url: https://cheerskevin.com/lets-start-reacting
  title: Lets Start Reacting
  lede:
    Okay, so I said before that I wanted to start using React to build pages. Why
    bring in another language instead of continuing to write stuff in Markdown? Good
    question! First off, Markdown is great.
- image: https://cheerskevin.com/lead-images/html-example.jpg
  color: rgb(65,64,65)
  published_at: Sat, 08 Apr 2017 21:43:08 -0500
  modified_at: Sat, 08 Apr 2017 21:43:08 -0500
  url: https://cheerskevin.com/unburying-the-lede.html
  title: Unburying the Lede
  lede:
    In the last post, we found a way to create static pages from React components.
    And in theory, we've defined a system by which we can create static pages from any
    source material. A file called coolPage.html.md will be processed by our processor,
    but a file called coolPage.html.whatever would be equally valid, so long as we had
    a whatever processor. One of the pages I'm most eager to build — which shouldn't
    come as much of a surprise — is the homepage!
```

Not bad! Granted, going back to old posts is going to require some tweaking of dates here and there, but now we've got all of the information about our posts in a nice central location, and we can start building a proper homepage next time!

[&larr;Previous Post](/lets-start-reacting.html)
|
[Next Post&rarr;](/no-place-like-home.html)
