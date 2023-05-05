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

.image-wrapper {
    width: 250px;
    position: relative;
    display: inline-block;
    margin-bottom: 30px;
}

.image-border {
    position: absolute;
    z-index: -1;
    bottom: -10px;
    right: -10px;
    width: 100%;
    height: 100%;
    border-right: 2px solid #2F7C5B;
    border-bottom: 2px solid #B56A4A;
    box-sizing: border-box;
}

.image-wrapper img {
    display: block;
    max-width: 100%;
    height: auto;
}
</style>
<script src="https://kit.fontawesome.com/c83e37f840.js" crossorigin="anonymous"></script>

<div class="my-5"></div>

# Hello!

<div class="image-wrapper">
    <img src="/assets/about/selfie.jpg" alt="selfie" />
    <div class="image-border"></div>
</div>

Welcome to my little hideout on the internet! My name is Alexander Yemane, and here you will find stuff about me, my passions, my studies, and my compendium of curiosities, which is mostly curated towards my cybersecurity educational journey.

I'm a postgraduate student pursuing a Diploma in Industrial Network Cybersecurity at [BCIT](https://www.bcit.ca/). Outside of school, I'm involved in the mentorship program @ [ISACA Vancouver](https://engage.isaca.org/vancouverchapter/home), the CTF team @ [UBC](https://maplebacon.org/), and various meetups organized by the [Mainland Advanced Research Society (MARS)](https://fourthplanet.ca/). If you're interested to know more about my background, feel free to check out my [CV](cv).

Note, I have a habit of tinkering, so the aesthetic and form of this site may change a lot since the last time you've checked in. You've been warned :) 

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