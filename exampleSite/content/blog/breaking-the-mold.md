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
        <p style="background-color"#ca7">Full Width Content</p>
        <p>More Content</p>
    </div>