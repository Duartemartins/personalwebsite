---
layout: post
title: How to add HrefLang tags to a Jekyll blog
description: How to add HrefLang tags to a Jekyll blog
date: 2021-05-25 23:09:37 +0100
published: true
categories: liquid jekyll programming webdev seo
tags:
lang:
---

If you are using the standard Jekyll minima 3 theme hosted on Github pages this can be done fairly easily.

Create a file called custom-head.html under _includes and add the following snippet in:

{% highlight html %}
{% raw %}{% if page.lang %}{% endraw %}
<link rel="alternate" hreflang={% raw %}"{{ page.lang }}"{% endraw %} href="https://www.yourdomain.com/{% raw %}{{ page.url }}"{% endraw %} />
{% raw %}{% else %}{% endraw %}
<link rel="alternate" hreflang="x-default" href={% raw %}"{{ page.url }}"{% endraw %} />
{% raw %}{% endif %}{% endraw %}
{% endhighlight %}

Then for posts written in different languages, add a "lang:" attribute in the post front matter, and set that as the target language. Remember you can also set regional localisation too, for example pt-BR for Portuguese speakers in Brazil, but also for cases like es-JP for Spanish speakers in Japan. Google follows [ISO639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for languages and [ISO 3166-1 Alpha 2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) for countries.

