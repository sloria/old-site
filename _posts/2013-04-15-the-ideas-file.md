---
layout: post
title: "The .ideas file"
description: "tips"
category: 
tags: [terminal, tips]
---
{% include JB/setup %}

If you're a regular Terminal user, chances are you've had a stroke of inspiration while your command prompt was up.

I find plain text in [Markdown](http://daringfireball.net/projects/markdown/) to be the quickest way to record these spontaneous ideas. Here's one way to do it:

Create a hidden file called `.ideas.md` in your home directory.

<pre><code class="bash">$ touch ~/.ideas.md      
</code>
</pre>

Add an alias to your `.bash_profile` or `.zshrc` file so you can open the file quickly 

<pre><code class="bash">alias idea="subl ~/.ideas.md"  # for Sublime Text users 
# OR
alias idea="vim ~/.ideas.md"  # for Vim users 
</code>
</pre>

Now when you have an idea, just type

<pre><code class="bash">$ idea  
</code>
</pre>

at the command line to open up the `.ideas.md` file in your text editor.

What you put in this file is completely up to you. Mine looks something like this:

<pre><code class="markdown">~/.ideas.md
# Software
    - doomed startup idea 1
    - some feature for an open-source project 
    - doomed startup idea 2
    - ...

# Blog posts
    - the .ideas file
    - stuff about nothing...
    - ...
</code>
</pre>

You can do this for todo lists, schedules, or any other text document that you open regularly.