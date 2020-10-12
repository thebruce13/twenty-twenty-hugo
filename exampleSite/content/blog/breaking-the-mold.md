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

So I often have a container I put my content in that has a max with that's about 1200px. However, sometimes I want to put a solid color or a background image that expands the whole screen. That is where this code comes in.

## Get to the Code!

Okay, so [the CodePen](https://codepen.io/brucebrotherton/pen/ZgxGad) I have built will show the how it is supposed to work and where some edge cases are brought into account. You can pick at that until your heart's content. I'll get into the details below.

## What is Going on Here?

Okay, so what I wanted to accomplish is having a section where you could put in content and the element behind it would have a background color or image that spanned the whole screen. That is very doable if you break up the wrapper div with something like:

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

This will get the job done and have all the text line up nicely inside of the established max width of wrapper. But this adds a lot of complexity to the HTML. Thankfully we can fix it with some underused and super powerful attributes, inherit attribute and pseudo-selectors.

The idea we are going to run with is that we can inherit down the background attribute to the pseudo-selectors and set them up to fill the whole width of the window. Ending up with some structure that looks like this

    <div class="wrapper">
    	<p>Some Content</p>
        <p class="break-wrapper" style="background-image:url('flexbox.png')>Full Width Content</p>
        <p>More Content</p>
    </div>

## But How?

I'll go through the steps and guide you through how I got the content to break the wrapper. Be sure to read the comments in the code sections, they will help you understand a lot. 

### Set up the wrapper breaker.

Alright, so assuming we have the same structure above, we will need to do some work with the `break-wrapper` class. We will want to make the div hide that it even has a background-image. But only if it has a background-image specified, otherwise we don't need to do anything. So we will use an advanced selector to look if the the element has anything to do with a background defined in it's style. 

```css
.break-wrapper[style*="background-image"] {
	display: table; /* Makes sure it can contain the pseudo-elements properly */
	position: relative /* the psudo-elements are position absolute, we need an anchor */
    width: 100%;
	background-color: transparent; /* Needs to be here so the color won't cover the image. */
	background-repeat: no-repeat; /* Don't want our image to tile. */
	background-position: 0 1000%; /* Shove the image waaay out of view */
	background-size: 0 0; /* Make it 0px high and wide, pretty much invisible */
	color: #fff;
}
```

Now that our `break-wrapper` class has effectively no background showing we can hand it off to our pseudo-element, `::after`. Again, only if it has a background-image attribute.

    .break-wrapper[style*="background-image"]::after {
    	content: "";
    	background-color: inherit; 
    	background-image: inherit; /* Grabs the background-image from the parent */
    	background-position: center center;
    	background-size: cover; /* Takes up the full space of the element */
    	background-repeat: no-repeat;
    	z-index: -1; /* Puts the image behind the text */
    }

Awesome, now we have the background image set on our pseudo-element. But wait, It isn't centered, it starts on the left and ends on the right.

We need to make it so the pseudo element is positions absolutely. And because the .break-wrapper is positioned relative it'll be used as an anchor point for the pseudo-element. 

    .break-wrapper[style*="background-image"]::after {
    	position: absolute;
    	top: 0;
    	left: 50%;
    	transform: translateX(-50%);
    	width: 100vw; /* This will ensure it is always the fulll Viewable Width of our screen */
    	height: 100%; /* Take up the whole height of the parent */
     }

We'll be used View Widths for the width attribute because if we set the width to 100%, it will only be as wide as the parent container, not the window. Now this will give us a background image that sticks outside of our main width but doesn't add any extra HTML to the page. 

So with all this together it will go like this

    .break-wrapper[style*="background-image"]::after {
    	content: "";
    	position: absolute;
    	top: 0;
    	left: 50%;
    	transform: translateX(-50%);
    	width: 100vw; /* This will ensure it is always the fulll Viewable Width of our screen */
    	height: 100%; /* Take up the whole height of the parent */
    	background-color: inherit; 
    	background-image: inherit; /* Grabs the background-image from the parent */
    	background-position: center center;
    	background-size: cover; /* Takes up the full space of the element */
    	background-repeat: no-repeat;
    	z-index: -1; /* Puts the image behind the text */
    }

### Almost Done

Now this is great, but if you put any text in there it will be nearly impossible to read, not to mention fail any accessibility checks. We will need to darken the background image a little to make this work. Good thing there are two pseudo elements we can use, now we can use the `::before` pseudo element. 

    .break-wrapper[style*="background"]::before {
    	content: "";
    	position: absolute;
    	top: 0;
    	left: 50%;
    	transform: translateX(-50%);
    	width: 100vw;
    	height: 100%;
    	background-color: rgba(0, 0, 0, 0.5);
    }

This will make it match up with our `::after` but there is one thing off, because it is technically above all our content it makes everything dark, the text included. We will need to add a z-index to it to make it show up properly. We have a -1 in our `::after` and that puts it behind the text right now. We need our `::before` to be behind the text but in front of the image. So we will adjust the `::before` to be -1 and the `::after `to be -2. So with them together it will look like this.

```css
.break-wrapper[style*="background-image"]::after {
	content: "";
	position: absolute;
	top: 0;
	left: 50%;
	transform: translateX(-50%);
	width: 100vw; /* This will ensure it is always the fulll Viewable Width of our screen */
	height: 100%; /* Take up the whole height of the parent */
	background-color: inherit; 
	background-image: inherit; /* Grabs the background-image from the parent */
	background-position: center center;
	background-size: cover; /* Takes up the full space of the element */
	background-repeat: no-repeat;
	z-index: -1; /* Puts the image behind the text */
}

.break-wrapper[style*="background"]::before {
	content: "";
	position: absolute;
	top: 0;
	left: 50%;
	transform: translateX(-50%);
	width: 100vw;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.5);
}
```

Side Note: I didn't even think of calling this main image a **FLEX**box until halfway through writing this.