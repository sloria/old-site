---
layout: post
title: "The .ideas file"
description: "tips"
category: 
tags: [terminal, tips]
---
{% include JB/setup %}
> The best way to a good idea is to have lots of ideas. <cite>Linus Pauling</cite>

If you're a regular Terminal user, chances are you've had a stroke of inspiration while your command prompt was up.

I find plain text in [Markdown](http://daringfireball.net/projects/markdown/) to be the quickest way to record these spontaneous ideas. Here's one way to do it:

Create a hidden file called `.ideas.md` in your home directory.

`$ touch ~/.ideas.md`


Add an alias to your `.bash_profile` or `.zshrc` file so you can open the file quickly 

`alias idea="subl ~/.ideas.md"  # for Sublime Text users`

**OR**

`alias idea="vim ~/.ideas.md"  # for Vim users `



Now when you have an idea, just type

`$ idea`

at the command line to open up the `.ideas.md` file in your text editor.

What you put in this file is completely up to you. Mine looks something like this:

<script src="https://gist.github.com/sloria/5895802.js"> </script>

You can do this for todo lists, schedules, or any other text document that you open regularly.