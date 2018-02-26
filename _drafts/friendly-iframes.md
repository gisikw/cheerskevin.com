---
layout: post
title: "Friendly Iframes"
---

In a previous post, we walked through [how to have responsive cross-origin iframes.](/2018/02/13/responsive-cross-origin-iframes.html) We had to apply a little elbow grease and use `postMessage` to support communication across the documents, but fixed the big problem with iframes: that they don't have dynamic height.

But in practice, you can often get away with asking people to embed a little bit of JavaScript. In fact, even our previous example required a little bit to handle the communication layer. If we can do this, then perhaps we don't need to serve something cross-domain at all!

## Why iframe at all?

If we're not relying on iframes for security, then why use iframes at all? One reason is to avoid conflicting stylesheets. You never know what `!important` attributes you're going to encounter, and throwing your content into an iframe ensures that the parent page's styling rules won't cascade into your own.

*Note: better yet, you could leverage the Shadow DOM! [Except &mdash; sadly &mdash; you can't](https://caniuse.com/#feat=shadowdom)*

## Creating friendly iframes

So let's create a friendly iframe. That is, an iframe which won't throw up nasty cross-origin complaints if we run JavaScript inside of it. First, we'll open an iframe to a blank page.

```html
<iframe id="unique" src="about:blank"></iframe>
```

And then, we can just write content into it with JavaScript:

```html
<iframe id="unique" src="about:blank"></iframe>
<script>
  var iframe = document.getElementById("unique");
  iframe.contentDocument.body.innerHTML = "<p>Welcome to the jungle!</p>";
</script>
```

An `about:blank` iframe is going to inherit the domain from the parent page, so we can pry it open and write things to the document with ease. If you're using a framework, your code might look something more like:

```html
<iframe id="unique" src="about:blank"></iframe>
<script>
  ReactDOM.render(
    <SomeFancypantsComponent />,
    document.getElementById("unique").contentDocument.body
  );
</script>
```

## Simplifying that call

Now, this does mean that we're requiring anyone who uses our embed to implicitly trust whatever JavaScript we throw at their page. Since we're doing that, we can at least minimize the amount of code they need to paste!

```html
<script async src="https://our-own-domain.muwahaha.com"></script>
```

Now we can put the heavy lifting in a script on our own server. We can even write the friendly iframe on-demand, and do it all with ES6, throwing up polyfills if necessary. Here's what that might look like:

```js
const OUR_SRC = "https://our-own-domain.muwahaha.com";

// Find the script tags in the page that match our domain
const targets = 
  Array.from(document.getElementsByTagName('script'))
    .filter(script => script.src.startsWith(OUR_SRC));

targets.forEach(script => {
  // Prepend a friendly iframe
  const iframe = document.createElement('iframe');
  iframe.src = "about:blank";
  script.parentNode.insertBefore(iframe, script);

  // Remove the original script tag
  script.parentNode.removeChild(script);

  // Write content to our iframe once it loads
  iframe.onload = () => {
    iframe.contentDocument.body.innerHTML = 
      "<p>It works!</p>";
  };
});
```

## Dynamic resizing time

We can use similar tricks to ensure our iframe resizes with the content. Only this time, we don't need to rely on `postMessage` and crazy listeners.

TODO
