---
layout: post
title: "Responsive Friendly Iframes"
date: 2018-03-02
---

In a previous post, we walked through [how to have responsive cross-origin iframes.](/2018/02/13/responsive-cross-origin-iframes.html) We had to apply a little elbow grease and use `postMessage` to support communication across the documents, but fixed the big problem with iframes: that they don't have dynamic height.

In practice, you can often get away with asking people to embed a little bit of JavaScript. In fact, even our previous example required a little bit to handle the communication layer. If we can do this, then perhaps we don't need to concern ourselves with cross-domain restrictions at all!

## Why iframe at all?

If we're not relying on iframes for security, then why use iframes in the first place? One reason is to avoid conflicting stylesheets. You never know what `!important` attributes you're going to encounter, and throwing your content into an iframe ensures that the parent page's styling rules won't cascade into your own.

*Note: better yet, you could leverage the Shadow DOM! [Except &mdash; sadly &mdash; you can't](https://caniuse.com/#feat=shadowdom)*

## Creating friendly iframes

So let's create a friendly iframe. That is: an iframe which won't throw up nasty cross-origin complaints if we run JavaScript inside of it. First, we'll open an iframe to a blank page.

```html
<iframe id="unique" src="about:blank"></iframe>
```

Then we can write content into it with JavaScript:

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

  // Write content to our iframe once it loads
  iframe.onload = () => {
    iframe.contentDocument.body.innerHTML = 
      "<p>It works!</p>";
  };

  // Insert the iframe and remove the original script
  script.parentNode.insertBefore(iframe, script);
  script.parentNode.removeChild(script);
});
```

## Dynamic resizing time

We can use similar tricks to ensure our iframe resizes with the content. Only this time, we don't need to rely on `postMessage` and crazy listeners. This time, we can set the iframe height directly. We'll modify our `onload` script to handle all the logic for us.

One minor quirk: in some browsers, the body element's clientHeight may use the size of the iframe &mdash; rather than the size of its contents &mdash; when using a friendly iframe. To combat this, we'll use a top-level wrapper div to  measure the true height of the iframe's contents.

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

  // Write content to our iframe once it loads
  iframe.onload = () => {
    const { body } = iframe.contentDocument;
    body.innerHTML =
      "<div id='root'><img src='https://picsum.photos/480/853/?random' width=100%></div>";
    body.style.margin = 0;
    body.parentElement.style.overflow = "hidden";

    // Define our resize function
    const root = iframe.contentDocument.getElementById("root");
    const resize = () => { iframe.height = root.clientHeight; };

    // Add event listeners
    iframe.contentWindow.addEventListener("resize", resize);
    new ResizeObserver(resize).observe(root);
  };

  // Insert the iframe and remove the original script
  script.parentNode.insertBefore(iframe, script);
  script.parentNode.removeChild(script);
});
```

## A few caveats / next steps

We cut a few corners for this example, but here are some further items to consider if you go this route:

- We're assuming that we have access to [ResizeObserver](https://developers.google.com/web/updates/2016/10/resizeobserver) here. For cross-browser support, you'll want to look at some polyfill options!
- Matching the `script` tag by checking the src is possibly a bit brittle. Perhaps you might consider using a data-attribute instead.
- Now that we've removed the iframe from the client-side entirely, the client doesn't have an easy way to apply their own styles (aside from some namespaced `iframe {}` CSS) Perhaps we could add some additional attributes on the script tag that we could tack on to the iframe when appending it to the page.

## Friendly iframes: all the fun with none of the security!

Our resulting embed code (`<script async src="https://our-own-domain.muwahaha.com"></script>`) ends up being pretty darned tiny for our clients. But in exchange, we are asking for the client to trust us not to be evil with our JavaScript. In practice, we can probably get away with this, since many clients are used to similar embed styles from Facebook, Twitter, and the like. The result is a tiny embed code that still avoids concerns over CSS conflicts.

For skittish hosts with greater security needs, keep in mind that we can still provide a responsive experience with secure iframes via `postMessage`. [It just requires more code on the client side](/2018/02/13/responsive-cross-origin-iframes.html).
