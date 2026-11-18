---
layout: page
title: "2019年 講義一覧"
description: >
  Missing Semester(MIT IAP 2019)の講義ノートと動画。
permalink: /2019/
phony: true
---

{% assign weekdays = "日,月,火,水,木,金,土" | split: "," %}
{% assign lecture_date = site['2019'] | group_by: 'date' | sort: 'name' %}
{% for date in lecture_date %}
  {% if date.name == "" %}{% continue %}{% endif %}
  {% assign weekday = date.name | date: '%w' | plus: 0 %}
  <h2>{{ date.name | date: '%-m/%-d' }}({{ weekdays[weekday] }})</h2>
  {% assign lectures = date.items | sort: 'order' %}
  <ul>
  {% for lecture in lectures %}
    {% if lecture.phony != true %}
      <li>
        <a href="{{ lecture.url | relative_url }}">{{ lecture.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
  </ul>
{% endfor %}

# MITの外へ

この授業がMITの外の人々にも役立つことを願って、私たちはこの授業をMITの外にも公開しています。投稿や議論は以下で見ることができます。

- [Hacker News](https://news.ycombinator.com/item?id=19078281)
- [Lobsters](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on)
- [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/an42uu/mit_hacker_tools_a_lecture_series_on_programmer/)
- [r/programming](https://www.reddit.com/r/programming/comments/an3xki/mit_hacker_tools_a_lecture_series_on_programmer/)
- [Twitter](https://twitter.com/Jonhoo/status/1091896192332693504)
- [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv)

# 謝辞

この授業は[SIPB IAP 2019](https://sipb.mit.edu/iap/2019/)の一環として開講され、[SIPB](https://sipb.mit.edu/)と[MIT EECS](https://www.eecs.mit.edu/)の共催で行われました。
