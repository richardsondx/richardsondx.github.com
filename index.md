---
layout: page
title: Hello World!
tagline: and those who knew how to code before they knew how to walk
---
{% include JB/setup %}

I am a `Ruby on Rails Developer` and the co-founder of a few baby startups [Treepoll](http://www.railsview.com); the first micropolling social network, [HelpMeDateYou](http://www.helpmedateyou.com); a place where girls and guys can share advice, [Railsview](http://www.railsview.com); a ruby on rails theme marketplace,[Railscircle](http://www.railcircle.com); Canada Ruby on Rails Job Board.

I love to code and among the languages and tools I often play with are: *HTML5*,*CSS3*,*Javascript*, *Jquery*, *Rubymotion*, *node.js*, *Adobe Photoshop CS6*... of course :) did I mention that I love web design? Check my [Dribbble](http://www.dribbble.com/richardsondx)

## Why you should follow this blog

I like to share anything that is often not clear enough to understand right away or usefull tools and tricks that I often use.  My blog is also a cool way to show off my work and skills and especially get feedback from the community in order to improve my code. I'm passionate about what I do so I want to give back. `_posts/core-samples` folder.

## How you should read this blog

This blog is pretty straight foward. When you see something like this...

    $ Sometime I'll write terminal commands or ruby code

It's mean that it's a code block. I'm going to share some of my favorite snipeets here so pay a close attention to this.

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


