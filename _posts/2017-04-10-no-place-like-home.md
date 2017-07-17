---
layout: post
title: "No Place Like Home"
date:   2017-04-10 23:12:36 -0500
---

> Author's Note: I'm currently in the process of migrating old blog posts to this new system. That may mean some links, syntax highlighting, and other details are broken or missing temporarily. Sorry for the inconvenience!

Our last two blog posts --- [Lets Start Reacting](/lets-start-reacting) and [Unburying the Lede](/unburying-the-lede) --- were rather anticlimactic. In the first one, we updated our file processor to generate HTML pages from React components. In the second one, we set up a crawler to fetch metadata from our posts and populate it into a `manifest.yml` file.

But we haven't actually done anything with those tools. That changes today!

## Let's build a homepage

By default, `cheerskevin.com` tries to serve the `index.html` file. Prior to this post, I had a manually-created file sitting there, just pointing to the most recent blog post. But now we can automatically build it as a React component:

```jsx
{% raw %}
import React from 'react';

function Homepage() {
  return (
    <html lang='en'>
      <head>
        <meta charset='utf-8' />
        <meta name='viewport' content='width=device-width, initial-scale=1' />
        <title>CheersKevin.com</title>
      </head>
      <body>
        <div style={{ maxWidth: '960px', margin: '0 auto' }}>
          <h1>Welcome to CheersKevin.com</h1>
          <p>Under construction - please come back soon!</p>
        </div>
      </body>
    </html>
  );
}

export default Homepage;
{% endraw %}
```

There's a lovely little static page. Though we don't have to keep everything in a single file like this. The only constraint for our renderer is that whatever we `export default` down at the bottom, needs to be a renderable React component. Content like the `<html lang='en'>` is stuff I think we're going to find ourselves reusing on most pages. So I'm going to go ahead and extract that to a new file, `PageWrapper.js`:

```jsx
import React from 'react';

function PageWrapper({ children, title }) {
  return (
    <html lang='en'>
      <head>
        <meta charset='utf-8' />
        <meta name='viewport' content='width=device-width, initial-scale=1' />
        <title>{ title }</title>
      </head>
      <body>{ children }</body>
    </html>
  );
}

export default PageWrapper;
```

Here, we're creating another component that expects `children` and a `title` field, which it conveniently inserts for us. Now we can go back to our Homepage component and shrink it down a little bit!

```jsx
{% raw %}
import React from 'react';
import PageWrapper from './PageWrapper';

function Homepage() {
  return (
    <PageWrapper title="CheersKevin.com">
      <div style={{ maxWidth: '960px', margin: '0 auto' }}>
        <h1>Welcome to CheersKevin.com</h1>
        <p>Under construction - please come back soon!</p>
      </div>
    </PageWrapper>
  );
}

export default Homepage;
{% endraw %}
```

Much nicer - and now we can reuse the PageWrapper component for other pages. We can just change the contents of that tag, and pass it a different title. `<PageWrapper title="About CheersKevin.com">` for an about page would work nicely!

Note that we use `import PageWrapper from './PageWrapper';` to pull in the wrapper component. Let's use the same technique to pull in our article metadata we gathered in [the last post](/unburying-the-lede).

## Snagging the metadata

Our article metadata lives in a file called `manifest.yml`, but if we try to import that outright, we're going to run into a small issue: we can't really just require a `yml` file as though it were JavaScript. Remember that the file itself looks like this:

```yml
- image: https://cheerskevin.com/lead-images/html-example.jpg
  color: rgb(65,64,65)
  url: https://cheerskevin.com/unburying-the-lede.html
  title: Unburying the Lede
  # ...etc
```

That's...not JavaScript. Instead, we need to grab a library to convert YAML into a nice JavaScript object. A quick Google search finds `js-yaml`, so let's install that!

```bash
yarn add --dev js-yaml
```

This library will convert YAML markup into a JS object, but importantly, we still need to read the YAML from the file ourselves. Thankfully, Node comes with the `fs` filesystem library which can do that for us. Let's update our Homepage component to pull in this information:

```jsx
{% raw %}
import React from 'react';
import yaml from 'js-yaml';
import fs from 'fs';
import PageWrapper from './PageWrapper';

const MANIFEST = yaml.safeLoad(fs.readFileSync('./manifest.yml'));

function Homepage() {
  return (
    <PageWrapper title="CheersKevin.com">
      <div style={{ maxWidth: '960px', margin: '0 auto' }}>
        <h1>Welcome to CheersKevin.com</h1>
        {
          MANIFEST.map((post) => (
            <div key={post.url}>
              <h2>
                <a href={post.url}>
                  { post.title }
                </a>
              </h2>
              <p>{ post.lede }</p>
            </div>
          ))
        }
      </div>
    </PageWrapper>
  );
}

export default Homepage;
{% endraw %}
```

We grab the `manifest.yml` file with `fs.readFileSync`, then pass that to `yaml.safeLoad`. That gives us the post information as a JavaScript array. That means that once inside our render method, we can use `map` to transform each record into some HTML of its own. Here's the result:

![BasicIndex](/images/starterIndex.jpg)

Not bad for a small amount of work! Of course, the manifest also has stuff like lead image and color scheme information, post dates, and more. So we can definitely expand on this (not to mention, style it up a fair bit). But now that we have our `manifest.yml` set up, and the ability to create static HTML from React components, we're all set to move forward with our great and awesome homepage!

That is, of course, outside of the scope of this article, so you'll just have to come back later to see the fancy styled page in a week or two!
