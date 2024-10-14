---
layout: default
---

<h2 class="home-title">Archives</h2>

<ul class="post-list">
    {% for post in site.posts %}
        {% if post.deprecated == true %}
      	<li>
            <h2>
      		    <a href="{{ post.url }}">{{ post.title }}</a>
            </h2>
      		<p class="post-excerpt">{{ post.excerpt }} (<a href="{{ post.url }}">more</a>)</p>
		</li>  
        {% endif %}
    {% endfor %}
</ul>