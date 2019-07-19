---
title: Recreateing Dev.to social images with javascript (canvas introduction)
published: false
description: Getting to know the canvas api on a practical example.
tags: javascript, canvas, dev.to, showdev
---

## Motivation

I recently had the idea to show my Dev.to posts on my website. The Dev.to api is really straight forward to use, a GET request to `https://dev.to/api/articles?username=<username>` returns an array of all your posts. Every post object has title, url, cover image user info and more.
What it doesn't have however is a url to a default image for when the user hasn't specified a cover image. One like this:


![Dev.to social preview example](https://thepracticaldev.s3.amazonaws.com/i/s74a36gd0u0m0rt5nu88.png)


By looking through the html of some posts i found the url 
`http://dev.to/social_previews/article/<post_id>.png` returns such an image. At first I was just hot-linking to this url from my blog. It turned out thought that those pictures take a few seconds to load. This looks really unprofessional and I suspect the pictures might even be generated every time this url gets called. I really like Dev.to and I don't want to put unnecessary load and cost on them even its just a little blog page with a couple visitors a day. This gave me the idea to generate those images myself with javascript.

## Solution

![Recreation of the preview](https://thepracticaldev.s3.amazonaws.com/i/xmpc281ahqf1wulz11wo.png)

Lets recreate such a social card with javascript and the canvas api. 

I never actually worked with the canvas api before but since our goal seems pretty achieveable I figured it would be a good starting point. 

If you want to checkout the code how I use it in vue on my personal page [it's here on githup](https://github.com/Tiim/Tiim.github.io/blob/source/src/components/BlogImage.vue).


Lets break down what needs to be done:

* A box containing all the content
* A solid drop shadow of the box
* The post title
* The author name
* The profile picture thumbnail
* The published date

Great, every one of these is provided by the api.

I will implement this example with Vue, but most of the code is vanilla javascript and can be used with any framework or even with vanilla js.

[The full vue js component like I use it on my webpage can be found here.](https://github.com/Tiim/Tiim.github.io/blob/source/src/components/BlogImage.vue)

### Step 0: Clean start ✨

Let's start with an almost empty component:

```vue
<!-- BlogImage.vue -->
<template>
  <div></div>
</template>

<script>
export default {
  name: 'BlogImage',
  props: {
    article: Object,
  },
  data() {
    return {
    };
  },
  mounted() {
  },
};
</script>

<style>
</style>

```

I already defined the prop article that will hold the information of an article as returned by the Dev.to api.


### Step 1: Laying the foundation 🚜

First we need to define our canvas element in html. We define the class for the next step and the ref so we can easily use the dom element from our code.
The sizing is set to 1064 x 588 because that's the size the original pictures are as well.

I also defined a img element for the case that the author specified an image manually. In that case lets just use this.

```html
<template>
  <div>
    <canvas v-if="!article.cover_image" class="canvas" ref="canvas" width="1064" height="588"></canvas>
    <img :src="article.cover_image" v-else :alt="article.title"/>
  </div>
</template>
```

The only styling we need is to set the width to 100% so the canvas fills the parent div.
```html
<style>
.canvas {
  width: 100%;
}
</style>
```

### Step 3: Getting the canvas dom element

Since vue uses a virtual dom we can't use the canvas element in our component right away. We have to wait for the component to be mounted first.

```js
  mounted() {
    const canvas = this.$refs.canvas;
    if (!canvas) return;
    

    const date = new Date(this.article.published_at);
    const month = date.toLocaleString('en-us', { month: 'short' });
    const year = date.getFullYear();
    const dateText = `${month} ${year}`;
    renderCanvas(canvas, {
      title: this.article.title,
      author: this.article.user.name,
      date: dateText,
      pic: this.article.user.profile_image_90,
    });
  },
```
First we get the canvas dom element and in case we don't have one we don't do anything.

While we're at it, lets format the date and call the function renderCanvas().

### Step 4: Rendering


