---
layout: post
title: Fitting the "339 bytes of responsive CSS" in a tweet, with a twist
category: posts
---

Being amazed by how good and functional a web page can be by just
[using 339 bytes of css](https://blog.koley.in/2019/339-bytes-of-responsive-css),
I decided to play a bit with it and adapt it to my needs.

The styling of the post you are reading right now, (minified, except syntax highlighting css) fits in a tweet.
It has the following features:
- responsive - looks good on any screen
- is friendly to source code
- does not use external fonts to save bytes and the planet
- **it supports Safari's dark mode**

Here's how the dark mode switching looks in action:

![](/assets/fitting-339-bytes-of-responsive-css-it-a-tweet/dark-mode.gif)

And here's the unminidied, annotated source:

```css
body {
  font-family: Georgia, serif;
  line-height: 1.6;
  /* when you use relative units, stuff looks good on any screen */
  max-width: 40rem;
  padding: 2rem;
  margin: auto;
}

img {
  /* make sure that all images fit in the viewport */
  max-width: 100%;
}

pre {
  padding: 0.5rem 0;
  /* Pre blocks don't wrap (and shouldn't). Scroll their content in case
  they overflow so that the viewport size stays the same */
  overflow-y: scroll;
}

@media (prefers-color-scheme:dark) {
  body {
    /* the dark mode just inverts everything, using a black background */
    filter: invert(1);
    background-color: black;
  }
  img {
    /* the invert filter above inverts images as well, so we'll need
    to revert that effect */
    filter: invert(1);
  }
}
```
