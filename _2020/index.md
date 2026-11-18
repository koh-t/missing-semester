---
layout: page
title: "2020年 講義一覧"
description: >
  Missing Semester(MIT IAP 2020)の講義ノートと動画。
permalink: /2020/
phony: true
---

<ul class="double-spaced">
  {% assign lectures = site['2020'] | sort: 'date' %}
  {% for lecture in lectures %}
    {% if lecture.phony != true %}
      <li>
        <strong>{{ lecture.date | date: '%-m/%-d' }}</strong>:
        {% if lecture.ready %}
          <a href="{{ lecture.url | relative_url }}">{{ lecture.title }}</a>
        {% elsif lecture.noclass %}
          {{ lecture.title }} [休講]
        {% else %}
          {{ lecture.title }} [近日公開]
        {% endif %}
        {% if lecture.details %}
          <br>
          ({{ lecture.details }})
        {% endif %}
      </li>
    {% endif %}
  {% endfor %}
</ul>

講義の録画は[YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J)で公開されています。

# MITの外へ

この授業がMITの外の人々にも役立つことを願って、私たちはこの授業をMITの外にも公開しています。投稿や議論は以下で見ることができます。

 - [Hacker News](https://news.ycombinator.com/item?id=22226380)
 - [Lobsters](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit)
 - [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/)
 - [r/programming](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/)
 - [Twitter](https://twitter.com/jonhoo/status/1224383452591509507)
 - [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J)

{% comment %}
その他のURL:

- https://news.ycombinator.com/item?id=27154577
- https://news.ycombinator.com/item?id=34934216
- https://www.reddit.com/r/learnprogramming/comments/nca1v3/mit_the_missing_semester_of_your_cs_education/
- https://www.reddit.com/r/compsci/comments/eyywv8/the_missing_semester_of_your_cs_education_from_mit/
- https://www.reddit.com/r/programming/comments/io7nq3/the_missing_semester_of_your_cs_education_mit/
- https://twitter.com/MIT_CSAIL/status/1349766980413263873
- https://twitter.com/MIT_CSAIL/status/1481676163491659780
- https://twitter.com/MIT_CSAIL/status/1581313961093484545
{% endcomment %}

# 謝辞

講義動画の収録を可能にしてくださったElaine Melloさん、Jim Cainさん、[MIT Open Learning](https://openlearning.mit.edu/)に感謝します。A/V機材を提供してくださったAnthony Zolnikさんと[MIT AeroAstro](https://aeroastro.mit.edu/)に感謝します。この授業を支援してくださったBrandi Adamsさんと[MIT EECS](https://www.eecs.mit.edu/)に感謝します。
