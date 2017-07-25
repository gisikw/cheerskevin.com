---
layout: post
title: "Let's Start Reacting"
date: 2017-04-01 21:43:08 -0500
---

> Author's Note: I'm currently in the process of migrating old blog posts to this new system. That may mean some links, syntax highlighting, and other details are broken or missing temporarily. Sorry for the inconvenience!

Okay, so I said before that I wanted to start using React to build pages. Why bring in another language instead of continuing to write stuff in Markdown? Good question!

First off, Markdown is great. Here's what this post looks like when I'm writing it:

```md
---
version: 0.4.0
title: Let's Start Reacting
description: An update to our processors, and JS time!
lead-image: garbage.jpg
url: https://cheerskevin.com/lets-start-reacting
---

# Lets start reacting

Okay, so I said before that I wanted to start using React to build pages. Why bring in another language instead of continuing to write stuff in Markdown? Good question!
```

> *As a brief aside, pasting the content again in here broke a bunch of the logic that grabs all that front-matter. Use `grep -m 1`, Kevin!*

Anyway, it's wonderful. But...it's wonderful for writing posts. Posts are supposed to look mostly the same. The only extent to which they vary is configured between those two `---` sandwich buns.

But let's imagine I wanted to create a personal health dashboard. Pull in data from wearable devices and show off how stunningly handsome and healthy I am. Markdown probably isn't the right medium then. For one thing, this template is all wrong. So we'd need to add more layers of customization to the front-matter, and it'd be a whole mess.

Markdown is great for expressing content, but not so great at expressing design. React, however, is *very* good at expressing design. Here's a stupid simplified and optimistic imagining of what things could looks like:

```jsx
class HealthPage extends React.Component {
  render() {
    const data = getAllTheData();
    return (
      <div>
        <h1>My Health Page</h1>
        <NutritionWidget data={data.nutrition} />
        <ExerciseWidget data={data.exercise} />
        <SleepWidget data={data.sleep} />
      </div>
    );
  }
}
```

That sounds fantastic. So, now all we need to do is add another processor. Whenever we detect a `.js` file, we can say "DO SOMETHING WITH THIS TO MAKE IT HTML!".

# OH WAIT NO THAT'S TERRIBLE

![Mistake!](/images/mistake.jpg)

Okay, so here's the problem: sometimes we *want* `.js` files. Just like `.html` and `.css` files, they have their uses as their own filetype. For that matter, what happens in a week or two when we decide we want to be able to generate `html` files from...I dunno, `jpg`s?

Okay, brief diversion to our processing logic. We'll no longer automatically process `file.something`. We'll process `file.html.something`! This is an idea we can steal from Rails (who might have stolen it from someone else). If we're writing Markdown that should be translated into HTML, we'll now use `post-name.html.md`.

If we're writing JavaScript that should be translated to HTML, `page-name.html.js`. As an aside, we could even chain these things if we somehow made a JavaScript file that would create Markdown.

`something.html.md.js` is a stupid example BUT! We could look at creating our own custom processors that aren't related to filetypes at all! Like `page.html.addColors.makeFancy.swoosh.md`. Or something. I dunno, let's not go too crazy. Okay, so let's update our `handleFileSave.sh` script:

```bash
#!/bin/bash
FILENAME=$1
NUMBER_OF_EXTENSIONS=$(echo $FILENAME | sed 's/[^\.]//g' | wc -c)

if [ "$NUMBER_OF_EXTENSIONS" -gt "2" ]; then
  EXTENSION=${FILENAME##*\.}
  TARGET=${FILENAME%.*}
  [ -x ./processors/$EXTENSION ] && ./processors/$EXTENSION $FILENAME > $TARGET
fi

./build-indexes.sh
rsync -ru --del . cheerskevin.com:/var/www/cheerskevin.com/
```

Now, what does this mean? It means none of our existing `post.md` files are going to be automatically converted to `post.html` files anymore (we can still manually run `./processors/md some-post.md > some-post.html`). But on the other hand, we can now distinguish between JavaScript written to be JavaScript, and...well, pages in the form of React components.

## Back to our regularly scheduled JavaScript...

Okay, now we just need to write a JS processor. Here are the rules for our processors:

- They get the $FILENAME as the first argument
- Whatever text they spit out gets written to the $FILENAME minus one extension (so `foo.html.js` goes to `foo.html`, `foo.html.md.js` goes to `foo.html.md`)

Alrighty, so let's use the power of our imagination to assume this already works, and write a file. We'll call this file `react-test.html.js`, and write it as though whoever wrote the processor was trying to make our lives easy.

```jsx
import React from 'react';

class ReactTest extends React.Component {
  render() {
    return (
      <div style={{ maxWidth: '960px', margin: '0 auto' }}>
        <h1>Huzzah! We made it work!</h1>
        <p>
          I mean, it's not pretty or anything, but it works!
        </p>
      </div>
    );
  }
}

export default ReactTest;
```

(For you React pure function purists out there; yes, this could be a pure function. Shaddap :P).

## Let's race to make it work now, k?

This post is getting rather long and rambly, and it's always good to make things work, and then get them right later. So let's just see how far we can get:

Okay, so we're using React, and we're using JSX syntax (and `import`/`export`), so we'll need to install some `babel-cli` and `react` and `react-dom`.

```bash
yarn add --dev babel-cli react react-dom
```

Oh geez, now our scripts are trying to upload the libraries to the site. A quick `--exclude 'node_modules'` will fix that. Back to JS!

So we have to write a script called `js`. I think it'll be easier if we just make that a really simple script that calls `babel-node` and passes along the filename.

```bash
#!/bin/bash
./node_modules/.bin/babel-node ./processors/jsProcessor.js $1
```

So it's going to run `jsProcessor.js` (which we still need to write), and pass along the first argument it got. Cool, let's make a dummy `jsProcessor.js` which just writes out some text.

```js
process.stdout.write('Hello, world!\n');
```

Now, if we just save our `react-test.html.js` file to trigger an update...

*Failure*.

```bash
SyntaxError: Unexpected token import
```

Right, we need to configure Babel to tell it what flavor of JavaScript it needs to support (in our case, ES2015 and JSX/React).

```bash
yarn add --dev babel-preset-es2015 babel-preset-react
echo '{ "presets": ["react", "es2015"] }' > .babelrc
```

Run it again, and a lovely "Hello, World!" pops up :) Now to change our `jsProcessor.js` to output more than just a test message, and actually convert our page:

```jsx
import ReactDOMServer from 'react-dom/server';
import React from 'react';

const FILENAME = process.argv[2];
const Component = require(`../${FILENAME}`).default;
process.stdout.write(
  `<!DOCTYPE html>
  ${ReactDOMServer.renderToStaticMarkup(<Component />)}\n`
);
```

And with that, [boom!](/react-test), we have our test page!

## What did we just do?

Welp, we've now get static page generation using React components. We can now write files like `about.html.js` or `recent-posts.html.js`, and provided that they define valid React components, those components will be rendered to static markup and written out to the corresponding HTML files. Not too bad for a day's work! And now our site will be much easier to build out, so stay tuned for that.

[&larr;Previous Post](/i-love-react.html)
|
[Next Post&rarr;](/unburying-the-lede.html)
