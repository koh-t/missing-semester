---
layout: page
title: 過去の開講
description: >
  Missing Semesterの過去のすべての開講一覧です。
---

{% comment %} デフォルトの "posts" コレクションを取り除くためにpopする {% endcomment %}
{% assign sorted_collections = site.collections | sort: 'label' | pop | reverse %}
<ul>
{% for collection in sorted_collections %}
    <li><a href="{{ site.baseurl }}/{{ collection.label }}/">{{ collection.label }}</a></li>
{% endfor %}
</ul>

各年度の講義はそれぞれ完全に自己完結しています。最新版の資料から始めることをおすすめします。扱うトピックは年度によって異なるため、過去の版の講義ノートと動画も引き続き公開しています。
