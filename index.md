---
title: Home
layout: home
---

Hello there. Welcome to my tech blog. I am a software engineer who has many interests. Here are the thoughts on project experiences and interesting technologies.

<div class="home">

  <ul class="post-list">
    {% for post in site.posts %}
      <li>
        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
        <div><a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></div>
        <p hidden>
        {% for tag in post.tags %}
        {{ tag }}
        {% endfor %}
        </p>
      </li>
    {% endfor %}
  </ul>

</div>

---

<div class="home">

<p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>

</div>