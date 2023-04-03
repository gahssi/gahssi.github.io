---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Homepage
# show_footer: false
---

<style>
.decorative-bg {
    z-index: -1;
    position: absolute;
    width: 100%;
}
</style>
<script src="https://kit.fontawesome.com/c83e37f840.js" crossorigin="anonymous"></script>

<div class="my-5"></div>

# Hello!

My name is Alexander, and welcome to my corner on the internet. Here you will find stuff about me, my studies, some resources I've collected to help break into cybersecurity, and my interests & hobbies.

I'm a postgraduate student pursuing a Diploma in Industrial Network Cybersecurity at [BCIT](https://www.bcit.ca/). Outside of school I'm involved in the [ISACA Vancouver](https://engage.isaca.org/vancouverchapter/home) mentorship program and the [UBC CTF team](https://maplebacon.org/).


I hope to freshen up my frontend development skills and play around with web design as I continue to flesh out my site. Speaking of skills, check out my [CV](cv) for more details.

## Contacts

<p>
{% for item in site.data.contacts.default %}
{% unless item.show_homepage %}
{% continue %}
{% endunless %}
{% if item.icon %}
<!-- <i class="{{ item.icon }}"></i> -->
<a class="btn" style="min-width: 3em; margin-right: .5em" href="{{ item.link }}"><i class="{{ item.icon }}"></i></a>
{% else %}
<i>{{ item.title }}</i>
{% endif %}
<!-- <a href="{{ item.link }}">{{ item.text }}</a>&nbsp; -->


{% endfor %}
</p>