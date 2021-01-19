---
layout: default
title: about
---

<!-- <img class="user-avatar" src="{{ site.owner.avatar }}"> -->

![bubble, bubble](/assets/images/cauldron.jpg)

I'm a writer and software developer based in the greater DC area. Stay in touch: [hi@andykuny.com](mailto:hi@andykuny.com)

## colophon

This site was built with Jekyll and the [White Paper theme](https://github.com/vinitkumar/white-paper).

<div class="pagination">
  {% if site.owner.email %}
  <a href="mailto:{{ site.owner.email }}" class="social-media-icons"
    ><i class="fa fa-2x fa-envelope-square" aria-hidden="true"></i
  ></a>
  {% endif %} {% if site.owner.github %}
  <a href="{{ site.owner.github }}" class="social-media-icons"
    ><i class="fa fa-2x fa-github-square" aria-hidden="true"></i
  ></a>
  {% endif %} {% if site.owner.linkedin %}
  <a href="{{ site.owner.linkedin }}" class="social-media-icons"
    ><i class="fa fa-2x fa-linkedin-square" aria-hidden="true"></i
  ></a>
  {% endif %} {% if site.owner.twitter %}
  <a
    href="https://twitter.com/{{ site.owner.twitter }}"
    class="social-media-icons"
    ><i class="fa fa-2x fa-twitter-square" aria-hidden="true"></i
  ></a>
  {% endif %}
</div>
