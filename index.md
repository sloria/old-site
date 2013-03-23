---
layout: page
title: Me
tagline: My personal page
---
{% include JB/setup %}

# The name's Steve. Just Steve.

### I like science, programming, and octopuses.

I am a student at the University of Wisconsin-Madison, a research assistant at the [Marler Laboratory][marlerlab], a programmer at the [Waisman Center for Brain Imaging][waisman], and a life-long learner.

You can also find me on [Github][github] and [LinkedIn][linkedin].

<!-- Here are some recent posts:

<ul class="listing">
  {% for post in site.posts limit:4 %}
  <li>
    <span>{{ post.date | date: "%B %e, %Y" }}</span> <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
  {% endfor %}
</ul> -->

[marlerlab]: http://psych.wisc.edu/marler/index.htm
[waisman]: http://brainimaging.waisman.wisc.edu/
[github]: http://www.github.com/{{ site.author.github }}
[linkedin]: http://www.linkedin.com/in/{{ site.author.linkedin }}
