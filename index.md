---
title: Home
layout: home
---

你好呀。欢迎来到我的技术博客。我是一名有很多兴趣的软件工程师。我之前的职业生涯一直从事存储软件开发。大部分工作经验是关于文件系统、NAS系统、键值存储、索引数据结构等。因为业余时间喜欢用STM32、ESP32、Atmega32u4等MCU做一些小项目。非常钦佩Arduino和QMK Firmware、CircuitPython等项目的开放分享精神，一直想和大家分享技术的乐趣。我以前没有时间写博客，不过最近比较空闲，就写写自己玩过的比较有意思的项目。

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