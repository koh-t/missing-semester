---
layout: page
title: "2026年 講義一覧"
description: >
  Missing Semester(MIT IAP 2026)の講義ノートと動画。
permalink: /2026/
phony: true
---

<ul class="double-spaced">
  {% assign lectures = site['2026'] | sort: 'date' %}
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

講義の録画は[YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L)で公開されています。

# MITの外へ

この授業がMITの外の人々にも役立つことを願って、私たちはこの授業をMITの外にも公開しています。投稿や議論は以下で見ることができます。

- [Hacker News](https://news.ycombinator.com/item?id=47124171)
- [Lobsters](https://lobste.rs/s/q4ykw7/missing_semester_your_cs_education_2026)
- [r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/1r93yk6/the_missing_semester_of_your_cs_education_2026/)
- [X](https://x.com/anishathalye/status/2024521145777848588)
- [Bluesky](https://bsky.app/profile/jonhoo.eu/post/3mfa2bhyuj22i)
- [Mastodon](https://fosstodon.org/@jonhoo/116098318361854057)
- [LinkedIn](https://www.linkedin.com/posts/anishathalye_i-returned-to-mit-during-iap-january-term-activity-7430285026933522433-Ehr9)
- [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L)

# 謝辞

講義動画の収録を可能にしてくださったElaine Melloさんと[MIT Open Learning](https://openlearning.mit.edu/)に感謝します。[SIPB IAP 2026](https://sipb.mit.edu/iap/)の一環としてこの授業を支援してくださったLuis Turinoさん / [SIPB](https://sipb.mit.edu/)に感謝します。
