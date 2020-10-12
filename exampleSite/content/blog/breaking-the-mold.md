+++
author = ["Bruce"]
categories = ["Font End"]
date = 2020-10-11T14:00:00Z
description = "How to break out of the restrictions of a column when using a framework."
image = "/images/busting-columns.png"
tags = ["CSS"]
title = "Breaking the Wrapper"

+++
## My Problem with Wrappers

So I often have a container that I put my content in with a max width that has a max width. However, sometimes I want to put a background image that expands the whole screen. That is what inspired me to write this.

## Get to the Code!

I have built a [Codepen](https://codepen.io/brucebrotherton/pen/ZgxGad) that will show the how it is supposed to work with some edge cases accounted for. You can pick at that until your heart's content. I'll get into the details below.

## What is Going on Here?

What I wanted to accomplish is having a section that you could put in content with a background image that spanned the whole screen. That is very doable if you break up the wrapper div with something like:

```html
<div class="wrapper">
	<p>Some Content</p>
</div>
<div class="fit-screen" background-color="#bad">
	<div class="wrapper">
		<p>Full Width Content</p>
    </div>
</div>
<div> class="wrapper">
	<p>More Content</p>
</div>
```

This will get the job done and have all the text line up nicely inside of the established max width of wrapper. However, this adds a lot of complexity to the HTML. Thankfully, we can fix it with some underused and super powerful attributes and selectors, namely the [inherit](https://www.w3schools.com/CSSref/css_inherit.asp) attribute and [pseudo-element](https://css-tricks.com/almanac/selectors/a/after-and-before/) selectors.

The idea we are going to run with is that we can inherit the background attribute into the pseudo-selectors and set them up to fill the whole width of the window. Ending up with the HTML structure that looks like this:

```html
<div class="wrapper">
	<p>Some Content</p>
    <p class="break-wrapper" style="background-image:url('flexbox.png')">Full Width Content</p>
    <p>More Content</p>
</div>
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/1/](https://jsfiddle.net/brucifer906/xkwuyah2/1/ "https://jsfiddle.net/brucifer906/xkwuyah2/1/")

## But How?

I'll go through the steps and guide you through how I got the content to break the wrapper. Be sure to read the comments in the code sections (the parts between /* */), they will help you understand a lot.

### Set up the wrapper breaker.

Alright, so assuming we have the same structure above, we will need to do some work with the `break-wrapper` class. First we are going to disguise the element to make it look like it doesn't have a background image at all.

```css
.break-wrapper{
	background-color: transparent; /* Needs to be here so the color won't cover the image. */
	background-repeat: no-repeat; /* Don't want our image to tile. */
	background-position: 0 1000%; /* Shove the image waaay out of view */
	background-size: 0 0; /* Make it 0px high and wide, pretty much invisible */
}
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/2/](https://jsfiddle.net/brucifer906/xkwuyah2/2/ "https://jsfiddle.net/brucifer906/xkwuyah2/2/")

After that we have to ensure that it will take up as much space as it can and serve as an anchor for anything that has the attribute `position: absolute` inside of it, spoiler: it will.

```css
.break-wrapper {
	display: table; /* Makes sure it can contain the pseudo-elements properly */
	position: relative; /* the pseudo-element are position absolute, we need an anchor */
    width: 100%;
}
```

The `display: table` is in there because the pseudo-element will be taking up 100% with `position: absolute` and without the table, it has gap at the bottom.

When you put it all together it'll look like this:

```css
.break-wrapper {
	display: table;
	position: relative;
    width: 100%;
	background-color: transparent;
	background-repeat: no-repeat;
	background-position: 0 1000%;
	background-size: 0 0; 
}
```

Example:  [https://jsfiddle.net/brucifer906/xkwuyah2/3/](https://jsfiddle.net/brucifer906/xkwuyah2/3/ "https://jsfiddle.net/brucifer906/xkwuyah2/3/")

Now that our `break-wrapper` class has effectively no background showing and can be an anchor for our pseudo-elements, we can get cracking on the `::after`

### The Pseudo Inheritance

[Sara Cope at CSS-Tricks](https://css-tricks.com/almanac/selectors/a/after-and-before/) has a good intro article about the ::before and ::after pseudo-elements. Specifically, when you would want to use them in relationship to content. We are going to be using both to make a pseudo structure in the HMTL like this

```html
<p class="break-wrapper" style="background-image:url('flexbox.png')>
	::before
    Full Width Content
    ::after
</p>    
```

The cool thing about this is we don't need to put the pseudo content into the HTML on the page. And because we are using them as blank elements they are going to work perfect.

We are going to start with the `::after` pseudo element because it will be the one that displays the background image.

What we want to accomplish is looking to the `::after`'s parent to see what background they are using and bring that into itself. The way we do that is to set the background-image to inherit.

```css
.break-wrapper::after {
	content: ""; /* Necessary attribute for pseudo elements to be valid */
    display: block; /* Need this so it will respect height and width attributes */
    background-image: inherit;
 }
```

The `content` field is there because it is required for a pseudo element to show, otherwise this will be invalid. But it isn't set to cover its container. We'll want to go ahead and define some more background attributes to make that happen.

```css
.break-wrapper::after {
	background-position: center center; /* Puts the image in teh middle of the container. */
	background-size: cover; /* Takes up the full space of the element */
	background-repeat: no-repeat; /* Makes sure it won't repeat itself */
}
```

So far our pseudo element looks like this. A good start but not quite showing anything.

```css
.break-wrapper::after{
	content: "";
	background-image: inherit; 
	background-position: center center;
	background-size: cover;
	background-repeat: no-repeat;
}
```

### Pseudo Placement

Now we have the background image set on our pseudo-element. But wait, we still can't see it! The problem is, we didn't tell it to take up any space and since the content is blank it won't show, the same way an empty `<div>` won't appear.

Lets go ahead and define the size of it.

```css
.break-wrapper::after {
	width: 100vw; /* This will ensure it is always the fulll Viewable Width of our screen */
	height: 100%; /* Take up the whole height of the parent */
 }
```

We used a fancy unit called [Viewport Widths](https://alligator.io/css/viewport-units/) here. This will ensure that our background takes up the full width of the screen, not it's container. Now this isn't the final version because `height: 100%` doesn't do anything to an element normally. It is usually because its hard for the browser to define how that 100% will be calculated. So, it will be nothing. There is a way around this, and that is setting the `::after` to `position: absolute`

```css
.break-wrapper::after{
	position: absolute;
}
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/4/](https://jsfiddle.net/brucifer906/xkwuyah2/4/ "https://jsfiddle.net/brucifer906/xkwuyah2/4/")

Sweet, now this is almost done, but right now it is still anchored to the left side of the wrapper and sailing way off to the right. In order to get this centered we will need to add a couple rules. While we are at it we will set the `.break-wrapper` to sit at the top of its parent instead of after the content of it.

```css
.break-wrapper{
	top: 0;
    left: 50%;
    transform: translateX(-50%);
}
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/5/](https://jsfiddle.net/brucifer906/xkwuyah2/5/ "https://jsfiddle.net/brucifer906/xkwuyah2/5/")

The last thing we need to account for is the fact that it is now in front of the content and we need it to be behind it. Luckily there is a CSS attribute just for this, z-index. If we set it to -1 it will put it behind our content and still be visible.

```css
.break-wrapper {
	z-index: -1;
}
```

### Pseudo Wrap-up

And there we have it, the background is behind our content and taking up the full width of the window. WHEW. Lets show it all together and see what we have got. Don't worry if your code isn't in the exact order, it will work just the same. But if you ARE worried, I suggest you look into [organizing your CSS](https://9elements.com/css-rule-order/) for the reason _why_ my styles are like this.

```css
.break-wrapper::after {
	position: absolute;
	top: 0;
	left: 50%;
    z-index: -1;
	transform: translateX(-50%);
    display: block;
	width: 100vw;
	height: 100%;
	background-color: inherit; 
	background-image: inherit;
	background-position: center center;
	background-size: cover; 
	background-repeat: no-repeat;
    content: "";
}
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/6/](https://jsfiddle.net/brucifer906/xkwuyah2/6/ "https://jsfiddle.net/brucifer906/xkwuyah2/6/")

### Before the After

Now this is great, but if you put any light colored text in there it will be nearly impossible to read, not to mention fail any accessibility checks. We will need to darken the background image a little to make this work. Good thing there are two pseudo elements we can use, now we can use the `::before` pseudo element. Following the same placement rules as the `::after` we can start the `::before`.

```css
.break-wrapper::before {
	position: absolute;
	top: 0;
	left: 50%;
	transform: translateX(-50%);
    display: block;
	width: 100vw;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.5);
    content: "";
}
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/7/](https://jsfiddle.net/brucifer906/xkwuyah2/7/ "https://jsfiddle.net/brucifer906/xkwuyah2/7/")

This will make it match up with our `::after` but there is one thing off, because it is technically above all our content it makes everything dark, the text included. We will need to add a z-index to it to make it show up properly. 

#### Place the Z-Index

We have a -1 in our `::after` and that puts it behind the text right now. We need our `::before` to be behind the text but in front of the image. So the transparent dark background will affect it. To do this we will need to decrease our `::after` to have a z-index of -2 and our `::before` to have a -1. This way they are both behind our text but ordered correctly with each other.

```css
.break-wrapper::after {
	position: absolute;
	top: 0;
	left: 50%;
    z-index: -2; /* Updated to be behind the ::before`
	transform: translateX(-50%);
    dsplay: block;
	width: 100vw;
	height: 100%;
	background-color: inherit; 
	background-image: inherit;
	background-position: center center;
	background-size: cover; 
	background-repeat: no-repeat;
    content: "";
}

.break-wrapper::before {
	position: absolute;
	top: 0;
	left: 50%;
    z-index: -1;
	transform: translateX(-50%);
    display: block;
	width: 100vw;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.5);
    content: "";
}
```

## Wrapping up the Break

Now everything is set here is our final CSS.

```css
.break-wrapper {
	display: table;
	position: relative;
    width: 100%;
	background-color: transparent;
	background-repeat: no-repeat;
	background-position: 0 1000%;
	background-size: 0 0; 
}

.break-wrapper::after {
	position: absolute;
	top: 0;
	left: 50%;
    z-index: -2; /* Updated to be behind the ::before */
	transform: translateX(-50%);
    display: block;
	width: 100vw;
	height: 100%;
	background-color: inherit; 
	background-image: inherit;
	background-position: center center;
	background-size: cover; 
	background-repeat: no-repeat;
    content: "";
}

.break-wrapper::before {
	position: absolute;
	top: 0;
	left: 50%;
    z-index: -1;
	transform: translateX(-50%);
    display: block;
	width: 100vw;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.5);
    content: "";
}
```

Example: [https://jsfiddle.net/brucifer906/xkwuyah2/9/](https://jsfiddle.net/brucifer906/xkwuyah2/9/ "https://jsfiddle.net/brucifer906/xkwuyah2/9/")

There are some final touch ups we could to for efficiencies sake, and if you are interested in that, check out the [Codepen ]( "https://codepen.io/brucebrotherton/pen/ZgxGad")for this project. There you can poke around and see what is going on. Delete parts push it to its limit. Put two inside themselves and see what happens. (it should break but, I've never tried it). That is all for now, hope you found this helpful.

Side Note: I didn't even think of calling the header image **FLEX**box until halfway through writing this.