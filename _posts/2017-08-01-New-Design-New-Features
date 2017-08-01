---
layout: post
title: New Design and New Features
tags: [blog, design]
---

After much tinkering with Jekyll (the engine that runs Github pages) I have finally got the blog to a level in which I am happpy with its look and feel. Since I am not the greatest web designer, I decided I would go for ease and simplicity and find a Jekyll theme that was close to what I wanted for this site. It took some time, but I finally decided upon [Beautiful Jekyll](http://deanattali.com/beautiful-jekyll/) by Dean Attali. As you will see there isn't a lot of customisation from the original theme and that is really a credit to the themes creator. 

However, the theme was lacking in one thing that I wanted and that was the ability to add tags. The theme does accomodate for them, however it states not to use these on Github pages and use them if you are running Jekyll on a local machine. So I figured I would integrate this on my own. 

After a big of googling around I found a few sites, however a lot of them had manaual ways of creating tags or the need to have pre-defined tags with a lot of manual processing for new ones. This didn't suit me at all, I want something nice clean and simple and then I found [Codinfox's page](https://codinfox.github.io/dev/2015/03/06/use-tags-and-categories-in-your-jekyll-based-github-pages/). Following along through this site allowed me to get the tags I want in my page. So here is how I did it.

### Create Tag Page
I created a tags.html page in a folder called blog/ in my root directory of the site. This page will collate all the tags used in each of my posts and then list the posts that correspond to those tags - basically a summary of tags and posts. (still confused? click on a tag and you will go to the page!). Inside tags.html I added the following code.

~~~~
---
layout: page
title: Tags
---

{% comment %}
=======================
The following part extracts all the tags from your posts and sort tags, so that you do not need to manually collect your tags to a place.
=======================
{% endcomment %}
{% assign rawtags = "" %}
{% for post in site.posts %}
	{% assign ttags = post.tags | join:'|' | append:'|' %}
	{% assign rawtags = rawtags | append:ttags %}
{% endfor %}
{% assign rawtags = rawtags | split:'|' | sort %}

{% comment %}
=======================
The following part removes dulpicated tags and invalid tags like blank tag.
=======================
{% endcomment %}
{% assign tags = "" %}
{% for tag in rawtags %}
	{% if tag != "" %}
		{% if tags == "" %}
			{% assign tags = tag | split:'|' %}
		{% endif %}
		{% unless tags contains tag %}
			{% assign tags = tags | join:'|' | append:'|' | append:tag | split:'|' %}
		{% endunless %}
	{% endif %}
{% endfor %}

{% comment %}
=======================
The purpose of this snippet is to list all the tags you have in your site.
=======================
{% endcomment %}
<div class="blog-tags">
{% for tag in tags %}
	<a href="#{{ tag | slugify }}" class="post-tag"> {{ tag }} </a>
{% endfor %}
</div>
<hr/>


{% comment %}
=======================
The purpose of this snippet is to list all your posts posted with a certain tag.
=======================
{% endcomment %}
{% for tag in tags %}
	<h2 id="{{ tag | slugify }}">{{ tag }}</h2>
	<ul>
	 {% for post in site.posts %}
		 {% if post.tags contains tag %}
		 <li>
		 <h3>
		 <a href="{{ post.url }}">
		 {{ post.title }}
		 <small>{{ post.date | date_to_string }}</small>
		 </a>
		 </h3>
		 </li>
		 {% endif %}
	 {% endfor %}
	</ul>
{% endfor %}
~~~~
The comments in the code should really explain how this all works. 

From here, I went to my Beautiful Jekyll _config.yml file and set link-tags to true
~~~~
# Use tags pages (not recommended if you are deploying via GitHub pages, only set to true if deploying locally with ruby)
link-tags: true
~~~~

and then modified the same bit of code on the index.html file and post.html layout file (located in _layouts) to point to my own tags.html page. All that is needed to be modified is the <a href> component of the following code.

~~~~
{% if post.tags.size > 0 %}
    <div class="blog-tags">
      Tags:
      {% if site.link-tags %}
      {% for tag in post.tags %}
      <a href="{{ site.baseurl }}/blog/tags#{{ tag }}">{{ tag }}</a>
      {% endfor %}
      {% else %}
        {{ post.tags | join: ", " }}
      {% endif %}
    </div>
{% endif %}
~~~~
This will make your tags now work perfectly. Click a tag and you will be taken to a page to view all the other posts with the same tag. Perfect! 

Cheers,
