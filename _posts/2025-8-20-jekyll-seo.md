---
layout: post
title:  How to Do Jekyll SEO in 2025
date:   2025-08-20
categories: 
slug: jekyll-seo
excerpt: A collection of techniques to improve SEO and discoverability of a Jekyll site.
og_image: /assets/images/og/og-seo-jekyll.png
deprecated: false
main_page: true
---

I want to get better at marketing, to get more views on my content and products, and boost discoverability. I'm not an SEO expert; I want to learn and improve. I am using my personal website as a test subject to try out best practices and experiment. 

By default, a Jekyll site is quite barebone in terms of SEO, which makes it perfect for this experiment. There are excellent plugins available for most tasks, and it's also easy to write custom code for more control.

This post is a living document of what I've tried so far, and the site itself is also [open source](https://github.com/andraskindler/andraskindler.github.io).

## Sitemap in Jekyll
A sitemap is an xml listing all the pages of a website, helping search engines to crawl and index it. Usually available directly on the root route, and submitted to search engines (eg. Google Search Console). Mine is available [here](https://andraskindler.com/sitemap.xml).

This site is using the [Jekyll Sitemap Generator Plugin](https://github.com/jekyll/jekyll-sitemap), an open source plugin maintained by the Jekyll team. This is a pretty simple file to generate manually as well.

## Speed and Performance
Website performance is a core factor in ranking. Jekyll serves plain static files, so runtime performance is excellent. But there's always room for improvement. I check [PageSpeed Insights](https://pagespeed.web.dev/) and Core Web Vitals (via Google Search Console) from time to time to get actionable insights.

<figure style="text-align: center;">
  <img src="/assets/images/seo-jekyll-psi.png" alt="PageSpeed Insights of andraskindler.com (desktop version)">
  <figcaption><i>PageSpeed Insights of andraskindler.com (desktop version)</i></figcaption>
</figure>

Performance is also an obvious indirect factor. A slow and laggy site means bad user experience, often resulting in reduced conversion rate, users dropping off, lower retention.

## Urls in Jekyll
It's important to define SEO- and reader-friendly urls. This is straightforward to do in Jekyll, and the [documentation](https://jekyllrb.com/docs/permalinks/) does an excellent job at explaining it. This site uses the following setup:

* Globally defined permalink (`permalink: '/blog/:slug/'`) in the config file;
* Use the `slug` parameter in the front matter in each essay to define the "last part" of the url.

## Meta tags in Jekyll
Generally, the `title`, `description`, `canonical`, and `robots` tags are the most important for SEO. The latter is still missing for my site.

A popular plugin is the [Jekyll SEO tag](https://github.com/jekyll/jekyll-seo-tag), which takes care of generating meta tags for search engines, and og tags social networks. Alternatively, it is pretty straigthforward to implement it for more control. That's what I've ended up doing, the code is [available here](https://github.com/andraskindler/andraskindler.github.io/blob/main/_includes/seo.html).

## OG tags and Twitter tags in Jekyll
These control how the link preview card (also known as social card) looks when shared on Facebook, LinkedIn, X, etc. The Jekyll SEO tag can also take care of this.

## Making search results convert better
I want to improve the conversion of the search results as well. So far, this is what I'm doing:

* Favicon: this can appear next to the title (not working yet, I'm using an emoji as a favicon);
* Title: the `title` parameter of the page;
* Description: it is usually the content of the `meta-description` tag.

## What's next?
A list of things I want to try out in the future.
* Improve link management (also utilise `nofollow`, `noopener` params);
* Robots.txt;
* `robots` tag;
* Automatic generation of share images;
* GEO (short for Generative Engine Optimisation);
* SEO analysers;
* Experiment with keyword research tools;
* Try different analytics tools (currently using [Simple Analytics](https://www.simpleanalytics.com/));
* Add a 'Resources' section for useful stuff I read.

## Wrapping up
The technical part of SEO and discoverability is not rocket science, at least so far. I want to continue learning SEO, adding new features, writing code manually without using plugins, and monitoring search performance. If there's anything I'm missing, feedback is highly appreciated.