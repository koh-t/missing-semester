---
layout: page
title: The Missing Semester of Your CS Education
description: >
  コンピュータサイエンティストやプログラマとしての生産性を高める強力なツールを使いこなしましょう。
# subtitle: IAP 2026
subtitle: "2026"
nositetitle: true
---

大学の授業では、オペレーティングシステムから機械学習まで、CSの高度なトピックについて幅広く学びます。しかし、非常に重要でありながらほとんど扱われず、学生が独学で身につけるしかないテーマがひとつあります。それは「ツールを使いこなす力」です。この授業では、コマンドラインの使いこなし方、強力なテキストエディタの使い方、バージョン管理システムの高度な機能など、さまざまなことを教えます!

学生は在学中に何百時間も(そしてキャリアを通じて何千時間も)これらのツールを使うことになります。だからこそ、その体験をできるだけ滑らかで摩擦のないものにする価値があります。これらのツールをマスターすれば、ツールを思いどおりに動かす方法を調べる時間が減るだけでなく、以前は途方もなく複雑に思えた問題も解決できるようになります。

近年では、AIを活用・強化したツールやワークフローの登場によって、ソフトウェアエンジニアリングの多くの側面も変化の途上にあります。その欠点を理解したうえで適切に使えば、これらはCSの実務者に大きな恩恵をもたらすことが多く、実用的な知識を身につけておく価値があります。AIは分野横断的な基盤技術であるため、独立したAIの講義は設けず、最新の応用可能なAIツールとテクニックの使い方を各講義の中に直接組み込んでいます。

[この授業を開講する理由](/about/)もあわせてお読みください。

{% comment %}
# Registration

Sign up for the IAP 2026 class by filling out this [registration form](https://forms.gle/j2wMzi7qeiZmzEWy9).
{% endcomment %}

# シラバス

{% comment %}
**Lecture**: [35-225](https://whereis.mit.edu/?go=35), 1:30--2:30pm (_exception_: 3--4pm on Friday 1/16)<br>
**Discussion**: [OSSU Discord](https://ossu.dev/#community) (use `#missing-semester-forum` like you would use Piazza, and `#missing-semester` to chat with the class/instructors)
{% endcomment %}

<ul>
{% assign lectures = site['2026'] | sort: 'date' %}
{% for lecture in lectures %}
    {% if lecture.phony != true %}
        <li>
        <strong>{{ lecture.date | date: '%-m/%-d/%y' }}</strong>:
        {% if lecture.ready %}
            <a href="{{ lecture.url }}">{{ lecture.title }}</a>
        {% else %}
            {{ lecture.title }} {% if lecture.noclass %}[休講]{% endif %}
        {% endif %}
        </li>
    {% endif %}
{% endfor %}
</ul>

## 過去の年度の特別トピック

扱うトピックは年度によって異なります。これまでに扱ってきたすべてのトピックに興味がある方のために、過去の年度で扱ったものの2026年には扱わなかったトピックを以下に挙げます。

{% comment %} pop to remove default "posts" collection {% endcomment %}
{% assign sorted_collections = site.collections | sort: 'label' | pop | reverse %}
<ul>
{% for collection in sorted_collections %}
    {% assign grouped_lectures = site[collection.label] | group_by: 'date' | sort: 'name' %}
    {% for group in grouped_lectures %}
        {% assign sorted_lectures = group.items | sort: 'order' %}
        {% for lecture in sorted_lectures %}
            {% if lecture.special == true %}
                <li>
                    <strong>{{ lecture.date | date: '%-m/%-d/%y' }}</strong>:
                    <a href="{{ lecture.url }}">{{ lecture.title }}</a>
                </li>
            {% endif %}
        {% endfor %}
    {% endfor %}
{% endfor %}
</ul>

{% comment %}
Lecture videos will be made available to MIT students immediately after lecture (via Panopto). The system has a limitation that only those with an MIT Kerberos can access the raw lecture videos. We are working on editing lecture videos and uploading them to YouTube. A couple have been uploaded already; we expect the rest to be uploaded by mid-February.

If you can't wait until January 2026, you can also take a look at the lectures
from the [previous offering of the course](/2020/), which covers many of the
same topics.
{% endcomment %}

# 一般情報

**スタッフ**: この授業は [Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/)、[Jose](https://josejg.com/) が共同で教えています。<br>
**質問**: [missing-semester@mit.edu](mailto:missing-semester@mit.edu) までメールでお問い合わせください。<br>
**ディスカッション**: [OSSU Discord](https://ossu.dev/#community)(`#missing-semester-forum` はPiazzaのように質問用に、`#missing-semester` は受講者・講師との雑談用にお使いください)。

# MITの外へ

この授業から他の方々にも役立ててもらえるよう、MITの外にも公開しています。投稿やディスカッションは以下で見つかります。

 - Hacker News ([2026](https://news.ycombinator.com/item?id=47124171), [2020](https://news.ycombinator.com/item?id=22226380), [2019](https://news.ycombinator.com/item?id=19078281))
 - Lobsters ([2026](https://lobste.rs/s/q4ykw7/missing_semester_your_cs_education_2026), [2020](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit), [2019](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on))
 - r/learnprogramming ([2026](https://www.reddit.com/r/learnprogramming/comments/1r93yk6/the_missing_semester_of_your_cs_education_2026/), [2020](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/), [2019](https://www.reddit.com/r/learnprogramming/comments/an42uu/mit_hacker_tools_a_lecture_series_on_programmer/))
 - r/programming ([2020](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/), [2019](https://www.reddit.com/r/programming/comments/an3xki/mit_hacker_tools_a_lecture_series_on_programmer/))
 - X ([2026](https://x.com/anishathalye/status/2024521145777848588), [2020](https://twitter.com/jonhoo/status/1224383452591509507), [2019](https://x.com/jonhoo/status/1090323977766137858))
 - Bluesky ([2026](https://bsky.app/profile/jonhoo.eu/post/3mfa2bhyuj22i))
 - Mastodon ([2026](https://fosstodon.org/@jonhoo/116098318361854057))
 - LinkedIn ([2026](https://www.linkedin.com/posts/anishathalye_i-returned-to-mit-during-iap-january-term-activity-7430285026933522433-Ehr9))
 - YouTube ([2026](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L), [2020](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J), [2019](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv))

# 翻訳

{% comment %} keep these in alphabetical order {% endcomment %}

- [アラビア語](https://missing-semester-ar.github.io/)
- [ベンガル語](https://missing-semester-bn.github.io/)
- [中国語(簡体字)](https://missing-semester-cn.github.io/)
- [中国語(繁体字・台湾)](https://missing-semester-tw.github.io/)
- [ドイツ語](https://missing-semester-de.github.io/)
- [イタリア語](https://missing-semester-it.github.io/)
- [日本語](https://missing-semester-jp.github.io/)
- [カンナダ語](https://missing-semester-kn.github.io/)
- [韓国語](https://missing-semester-kr.github.io/)
- [モンゴル語](https://missing-semester-mn.github.io)
- [ペルシア語](https://missing-semester-fa.github.io/)
- [ポルトガル語](https://missing-semester-pt.github.io/)
- [ロシア語](https://missing-semester-rus.github.io/)
- [セルビア語](https://netboxify.com/missing-semester/)
- [スペイン語](https://missing-semester-esp.github.io/)
- [スウェーデン語](https://den-saknade-terminen.l10n.se/)
- [タイ語](https://missing-semester-th.github.io/)
- [トルコ語](https://missing-semester-tr.github.io/)
- [ベトナム語](https://missing-semester-vn.github.io/)

注: これらはコミュニティによる翻訳への外部リンクです。内容の検証は行っていません。

この授業の講義ノートを翻訳しましたか? [プルリクエスト](https://github.com/missing-semester/missing-semester/pulls)を送っていただければ、リストに追加します!

## 謝辞

{% comment %}
2026 acks; previous years' acks are on their respective pages
{% endcomment %}

講義動画の収録を可能にしてくださったElaine Melloさんと[MIT Open Learning](https://openlearning.mit.edu/)に感謝します。[SIPB IAP 2026](https://sipb.mit.edu/iap/)の一環としてこの授業を支援してくださったLuis Turinoさん / [SIPB](https://sipb.mit.edu/)に感謝します。

---

<div class="small center">
<p><a href="https://github.com/missing-semester/missing-semester">ソースコード</a>.</p>
<p>CC BY-NC-SAでライセンスされています。</p>
<p>コントリビューションと翻訳のガイドラインは<a href="/license/">こちら</a>をご覧ください。</p>
</div>
