---
title: Home
layout: home
---

Hello there. Welcome to my tech blog. I am a software engineer who has many interests. I have been in storage industry for quite some time. Most of my experiences are about on File Systems, Network Attached Storage, KV Store and Index Data Strutures. During my spare time I like to play with MCUs like STM32, ESP32 and Atmega32u4, etc. I really admire the open and share spirit of projects like Arduino and QMK Firmware, CircuitPython, and want to share with you the pleasure of techonlogies. I used to have few time to blog. Recently I have quite some free time, I will share the interesting projects those I have played with.

<div class="home">
<div><u>博客文章:</u></div>
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
