---
layout: post
title: "Responsive Cross-Origin Iframes"
date: 2018-02-13
---

Love them or hate them, iframes provide a great way to securely include external content on a webpage. But for security reasons, iframes aren't allowed to dictate their own size, which can cause problems when it comes to responsive design.

```html
<iframe src="your-url-here" style="width: 80%"></iframe>
<!-- What should the height be??!??! -->
```

The contents of iframes are just as subject to reflows as the main page, yet their height is fixed (except on iOS Mobile Safari, which is a whole can of worms we won't open). How can we allow iframes to have a height that displays all the content, after specifying their width?

With a same-origin iframe, we can peer in an look at the content height with JavaScript, and so can theoretically monkey-patch some resizing logic that way. But what if we want to provide an embed for third-parties to use? Turns out, we can still make this work, with a little `<script>` tag to go along with the iframe.

## Cross-origin communication

We can leverage [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) to communicate our height requirements from the iframe to the parent window. We'll update this snippet later, but for now we'll just add a script to the bottom of our iframe:

```html
<script>
  window.parent.postMessage({ resizeMe: 100 }, "*");
</script>
```

The parent page needs to listen for messages and respond appropriately. We'll listen for messages, and if the message contains a `resizeMe` key, we know to resize the iframe:

```js
window.addEventListener('message', function(message) {
  if (message.data.resizeMe) {
    document.getElementsByTagName('iframe')[0].height = 
      message.data.resizeMe;
  }
});
```

### How to find *which* iframe to resize

This works well enough, assuming we only have a single iframe on the page. But how does the parent browser know *which* iframe sent the message?

The `message` object that we get in our callback has a `.source` property, which points to the iframe's `window` object. This feature is designed to allow two-way postMessage communication. We could send a reply with `message.source.postMessage`. But we can use this object as a way to uniquely identify the sending iframe!

Unfortunately, it's not tidy. With same-origin iframes, we can get the iframe element by calling `.frameElement` on the iframe's window object. If we try that with cross-origin iframes, it's a cross-origin violation. So instead, we'll need to loop over all the iframes and find the one whose window object matches the sender:

```js
window.addEventListener('message', function(message) {
  if (message.data.resizeMe) {
    var iframes = [].slice.apply(
      document.getElementsByTagName('iframe')
    );
    var sender = iframes.find(function(iframe) {
      return iframe.contentWindow === message.source;
    });
    sender.height = message.data.resizeMe;
  }
});
```

### Tidying up the code

Up until now, we've used `resizeMe` as the key which triggers the resize behavior. Ideally, we can use something a little more obscure, just to be confident we're not running into naming collisions with any *other* postMessage senders the parent site may have on the page. We can also revert to a plain ol' `for` loop in checking the iframe elements, for maximum backward compatibility. We'll also make sure we don't add duplicate event listeners, in case the parent page uses our snippet more than once.

Minimize our code, and we end up with something like this:

```html
<script>
(function(f,r,a,m,e){
  f[r]=f[r]||f.addEventListener('message',function(v){if(a=v.data[r]){
    m=document.getElementsByTagName('iframe');
    for(e=m.length;e--;){if(m[e].contentWindow===v.source)m[e].height=a}
  }})
})(window,'i-r')
</script>
<iframe src="your-url-here"></iframe>
```

It's worth noting that the script tag only *needs* to be included once. So if we were encouraging users to use multiple embeds, we might ask them to insert the script tag at the top of the page, and iframes wherever they like.

Finally, we need to update our iframe code to use our new unique message key:

```html
<script>
  window.parent.postMessage({ "i-r": 100 }, "*");
</script>
```

## Resizing from within the iframe

Now we need to do the heavy lifting inside our iframe to determine what height to send to postMessage, and *when* to actually send it. Let's tackle the first problem:

```js
document.body.style.margin = 0;
document.body.parentElement.style.overflow = "hidden";

function requestResize() {
  window.parent.postMessage({ 'i-r': document.body.clientHeight }, '*');
}

requestResize();
```

We're going to use the body tag as our master measuring container - which means we want it flush with the edges of the iframe - so we strip the margins. We also set `overflow: hidden` on the HTML tag, so we don't have to worry about those nasty scrollbars in our iframe. Then we can query the `clientHeight` on the body tag to determine the height of the iframe's contents - even if it's taller than the iframe itself!

Before we start adding additional `requestResize` calls, we should first ensure we don't spam the parent page with messages.

### Debouncing

Debouncing ensures that we only make a postMessage call once in a particular chunk of time. We can update our `requestResize` function to use a timer to selectively ignore calls:

```js
var timer;
function requestResize() {
  if (timer) return;
  timer = setTimeout(function() {
    window.parent.postMessage({ 'i-r': document.body.clientHeight }, '*');
    timer = null;
  }, 250);
}
```

Now we can be sure that we won't call `postMessage` more than once every 250ms. If there's already a call queued up, the function returns immediately. After it executes, it clears the timer, and the next call is able to get queued. With that out of the way, we can look at the two cases where we need to set our height: external resizing, and internal resizing.

### External Resizing

External resizing happens when the parent page decides to set a new width on the iframe. This might be because the iframe has a responsive width and the user has resized the browser, or it could be that the parent page has some JavaScript of its own that's tweaking the iframe for some reason.

In any case, because the iframe is its own document, any change to the iframe's size is going to fire off a resize event. So we can listen to that and call `requestResize` whenever it happens:

```js
window.addEventListener('resize', requestResize);
```

It's worth noting here that when we send a postMessage to the parent page, the parent page will resize our iframe. Which...will trigger a requestResize call (assuming the size has changed). Thankfully, we have the debouncing in place, so we're not stuck in an infinite loop of doom!

### Internal Resizing

Our iframe may have reasons why the total height would change independent of the browser window. We could have slow-loading images, or toggleable elements. In any case, we want to be able to observe our body element to determine when we need to ask to have our height adjusted.

In the most recent version of Chrome, this is easy! We can leverage the brand-new, flashy [ResizeObserver](https://developers.google.com/web/updates/2016/10/resizeobserver)!

```js
if (window.ResizeObserver) {
  (new ResizeObserver(requestResize)).observe(document.body);
}
```

Out of the box, we get an event that fires off when the body element changes dimensions, and can immediately call our `requestResize` function. For other browsers, we'll want to use [resize-observer-polyfill](https://github.com/que-etc/resize-observer-polyfill). Or, since we're lazy and hate laptop batteries, we can resort to evil polling:

```js
if (window.ResizeObserver) {
  (new ResizeObserver(requestResize)).observe(document.body);
} else {
  var height;
  setInterval(function() {
    requestAnimationFrame(function() {
      var bodyHeight = document.body.clientHeight;
      if (height !== bodyHeight) {
        requestResize();
        height = bodyHeight;
      }
    });
  }, 1000);
}
```

Shame on us! Shame! Shame! Shame!

### Wrapping that up

Putting together all the iframe code gives us this script tag to place at the bottom of our body tag:

```js
(function() {
  document.body.style.margin = 0;
  document.body.parentElement.style.overflow = "hidden";

  var timer;
  function requestResize() {
    if (timer) return;
    timer = setTimeout(function() {
      window.parent.postMessage({ 'i-r': document.body.clientHeight }, '*');
      timer = null;
    }, 250);
  }

  if (window.ResizeObserver) {
    (new ResizeObserver(requestResize)).observe(document.body);
  } else {
    var height;
    setInterval(function() {
      requestAnimationFrame(function() {
        var bodyHeight = document.body.clientHeight;
        if (height !== bodyHeight) {
          requestResize();
          height = bodyHeight;
        }
      });
    }, 1000);
  }

  requestResize();
}());
```

You can get this down to about half a kb with minification, but since the iframe code doesn't need to be shared as part of the embed, best to leave that to automated minifiers.

## Hurray for iframes!

With a little massaging, we're able to get iframes to behave like first-class citizens! We get our lovely CSS sandbox (still waiting on cross-browser Shadow DOM support), and a way around cross-origin issues without requiring the parent page to be overly permissive. For cases when asking your clients to use insecure script embeds raises a few eyebrows, consider the magic of the responsive iframe.
