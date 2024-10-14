---
layout: default
---

<h2 class="home-title">Hi! I'm Andras Kindler.</h2>

<p>I'm building <a href="https://brew.dev" target="_blank">Brew</a> and <a href="https://followergrow.com" target="_blank">Followergrow</a>. Previously, I was the cofounder of <a href="https://makery.co" target="_blank">Makery</a> (acquired in 2021), then VP Engineering at TIER Mobility.</p>

<p>
I originally started this site 10+ years ago, writing mostly about Android development, and closed it down for a few years because of inactivity. Recently I started writing again, with a wider range of topics about my professional journey.</p>

<p class="home-title">Posts:</p>

<ul class="post-list">
    {% for post in site.posts %}
        {% if post.deprecated == false %}
      	<li>
            <h2>
      		    <a href="{{ post.url }}">{{ post.title }}</a>
            </h2>
      		<p class="post-excerpt">{{ post.excerpt }} (<a href="{{ post.url }}">more</a>)</p>
		</li>
        {% endif %}
    {% endfor %}
</ul>

<p>You can find older posts in the <a href="/archives">archives</a>.</p>