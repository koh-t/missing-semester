---
layout: lecture
title: "Webとブラウザ"
presenter: Jose
date: 2019-01-31
order: 1
video:
  aspect: 62.5
  id: XpZO3S8odec
special: true
---

ターミナルを別にすれば、Webブラウザはかなりの時間を費やすことになるツールです。したがって、効率的かつ生産的に使う方法を学んでおく価値があります。

## ショートカット

ブラウザの中をクリックして回るのは、多くの場合もっとも速い方法ではありません。よく使うショートカットに慣れておくと、長い目で見て大いに元が取れます。

- リンクを `Middle Button Click` すると、新しいタブで開きます
- `Ctrl+T` 新しいタブを開きます
- `Ctrl+Shift+T` 最近閉じたタブを開き直します
- `Ctrl+L` 検索バーの内容を選択します
- `Ctrl+F` Webページ内を検索します。これをよく使うなら、検索で正規表現をサポートする拡張機能が役に立つかもしれません。


## 検索演算子

GoogleやDuckDuckGoのようなWeb検索エンジンは、より凝ったWeb検索を可能にする検索演算子を提供しています。

- `"bar foo"` bar fooに完全一致するものだけに絞ります
- `foo site:bar.com` bar.comの中でfooを検索します
- `foo -bar ` barを含む語を検索から除外します
- `foobar filetype:pdf` その拡張子のファイルを検索します
- `(foo|bar)` fooまたはbarを含むものを検索します

主要な検索エンジンについては、より網羅的なリストが公開されています。[Google](https://ahrefs.com/blog/google-advanced-search-operators/)と[DuckDuckGo](https://duck.co/help/results/syntax)のものを参照してください


## 検索バー

検索バーも強力なツールです。ほとんどのブラウザは、Webサイトから検索エンジンを推測して保存してくれます。キーワードの引数は編集できます

- Google Chromeでは、[chrome://settings/searchEngines](chrome://settings/searchEngines)にあります
- Firefoxでは、[about:preferences#search](about:preferences#search)にあります

たとえば、`y SOME SEARCH TERMS` と入力するとYouTubeを直接検索するように設定できます。

さらに、ドメインを所有しているなら、レジストラを使ってサブドメインの転送を設定できます。たとえば私は、`https://ht.josejg.com` をこの授業のWebサイトに割り当てています。こうしておけば、`ht.` と入力するだけで検索バーが自動補完してくれます。この設定のもうひとつの良い点は、ブックマークと違って、どのブラウザでも機能することです。

## プライバシー拡張機能

最近では、Webの閲覧は広告のせいでかなり煩わしく、トラッカーのせいでプライバシーを侵害するものになりがちです。さらに、優れた広告ブロッカーは、ほとんどの広告コンテンツをブロックするだけでなく、怪しいWebサイトや悪意のあるWebサイトもブロックしてくれます。そうしたサイトは一般的なブラックリストに含まれているからです。また、実行されるリクエストの数が減ることで、ページの読み込み時間が短くなることもあります。いくつかおすすめを挙げます。

- **uBlock origin**([Chrome](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm)、[Firefox](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)): あらかじめ定義されたルールに基づいて広告とトラッカーをブロックします。設定で有効になっているブラックリストも確認してみるとよいでしょう。地域や閲覧の習慣に応じて、さらに多くのリストを有効にできるからです。[Web上のあちこち](https://github.com/gorhill/uBlock/wiki/Filter-lists-from-around-the-web)で公開されているフィルタをインストールすることもできます

- **[Privacy Badger](https://privacybadger.org/)**: トラッカーを自動的に検出してブロックします。たとえば、あなたがWebサイトからWebサイトへと移動するとき、広告会社はあなたがどのサイトを訪れたかを追跡し、あなたのプロファイルを作り上げています

- **[HTTPS everywhere](https://www.eff.org/https-everywhere)** は、HTTPS版が利用できる場合にWebサイトのHTTPS版へ自動的にリダイレクトしてくれる素晴らしい拡張機能です。

この種のアドオンは、[こちら](https://www.privacytools.io/privacy-browser-addons/)でさらに見つけられます

## スタイルのカスタマイズ

Webブラウザは、_あなたのマシン_ で動いているソフトウェアのひとつにすぎません。したがって、何を表示すべきか、どう振る舞うべきかについて最終的な決定権を持つのは、たいていあなたです。その一例がカスタムスタイルです。ブラウザは、Cascading Style Sheets(しばしばCSSと略されます)を使って、Webページのスタイルをどう描画するかを決定します。

Webサイトを検証(inspect)すればそのソースコードにアクセスでき、内容やスタイルを一時的に変更できます(これは、Webページのスクリーンショットを決して信用してはならない理由でもあります)。

あるWebページのスタイル設定を上書きするようブラウザに恒久的に指示したい場合は、拡張機能を使う必要があります。私たちのおすすめは **[Stylus](https://github.com/openstyles/stylus)**([Firefox](https://addons.mozilla.org/en-US/firefox/addon/styl-us/)、[Chrome](https://chrome.google.com/webstore/detail/stylus/clngdbkpkpeebahjckkjfobafhncgmne?hl=en))です。


たとえば、この授業のWebサイト向けに次のようなスタイルを書けます


```css

body {
    background-color: #2d2d2d;
    color: #eee;
    font-family: Fira Code;
    font-size: 16pt;
}

a:link {
    text-decoration: none;
    color: #0a0;
}
```

さらにStylusは、他のユーザーが書いて[userstyles.org](https://userstyles.org/)で公開しているスタイルを探すこともできます。たとえば、よく使われるWebサイトのほとんどには、ダークテーマのスタイルシートがひとつ以上あります。ちなみに、Stylishは使うべきではありません。ユーザーのデータを漏洩させていたことが明らかになったからです。詳しくは[こちら](https://arstechnica.com/information-technology/2018/07/stylish-extension-with-2m-downloads-banished-for-tracking-every-site-visit/)


## 機能のカスタマイズ

スタイルを変更できるのと同じように、独自のJavaScriptを書き、[Tampermonkey](https://tampermonkey.net/)のようなWebブラウザ拡張機能を使ってそれを読み込ませることで、Webサイトの挙動を変更することもできます

たとえば次のスクリプトは、JキーとKキーを使ったvim風のナビゲーションを可能にします。

```js
// ==UserScript==
// @name         VIM HT
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  Vim JK for our website
// @author       You
// @match        https://hacker-tools.github.io/*
// @grant        none
// ==/UserScript==


(function() {
    'use strict';

    window.onkeyup = function(e) {
        var key = e.keyCode ? e.keyCode : e.which;

        if (key == 74) { // J is key 74
            window.scrollBy(0,500);;
        }else if (key == 75) { // K is key 75
            window.scrollBy(0,-500);;
        }
    }
})();
```

[OpenUserJS](https://openuserjs.org/)や[Greasy Fork](https://greasyfork.org/en)のようなスクリプトのリポジトリもあります。ただし注意してください。他人のユーザースクリプトをインストールするのは非常に危険な場合があります。クレジットカード番号を盗むなど、ほとんど何でもできてしまうからです。自分で全体を読み、何をするものかを理解し、怪しいことを何もしていないと絶対に確信できる場合を除いて、スクリプトは決してインストールしないでください。自分で読めない、ミニファイまたは難読化されたコードを含むスクリプトは、決してインストールしてはいけません!

## Web API

Webサービスがアプリケーションインターフェース、いわゆるWeb APIを提供することがますます一般的になっており、Webリクエストを送ることでサービスとやり取りできます。
このトピックのより詳しい入門は[こちら](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)にあります。[公開されているAPIは数多くあります](https://github.com/toddmotto/public-apis)。Web APIは、実にさまざまな理由で役に立ちます。

- **情報の取得**。Web APIを使うと、地図、天気、自分のパブリックIPアドレスといった情報をかなり簡単に取得できます。たとえば `curl ipinfo.io` は、あなたのパブリックIP、地域、位置などの詳細を含むJSONオブジェクトを返します。適切にパースすれば、これらのツールはコマンドラインツールとも統合できます。次のbash関数は、Googleの自動補完APIと通信して、最初の10件のマッチを返します。

```bash
function c() {
    url='https://www.google.com/complete/search?client=hp&hl=en&xhr=t'
    # NB: user-agent must be specified to get back UTF-8 data!
    curl -H 'user-agent: Mozilla/5.0' -sSG --data-urlencode "q=$*" "$url" |
        jq -r ".[1][][0]" |
        sed 's,</\?b>,,g'
}
```

- **操作の実行**。Web APIのエンドポイントは、アクションを起動するためにも使えます。これらには通常、サービスを通じて取得できる何らかの認証トークンが必要です。たとえば
`curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' "https://hooks.slack.com/services/$SLACK_TOKEN"` を実行すると、チャンネルに `Hello, World!` というメッセージが送信されます。

- **パイプ処理**。Web APIを持つサービスの中にはかなり人気のあるものがあるため、よくあるWeb API同士の「つなぎ合わせ」はすでに実装されており、サーバー込みで提供されています。[If This Then That](https://ifttt.com/)や[Zapier](https://zapier.com/)のようなサービスがこれにあたります


## Webの自動化

Web APIだけでは足りないこともあります。読み取りだけで済むなら、`pup` のようなHTMLパーサを使うか、ライブラリを使えます。たとえばPythonにはBeautifulSoupがあります。しかし、対話的な操作やJavaScriptの実行が必要な場合、これらの方法では力不足です。

- **WebDriver**。たとえば次のスクリプトは、WebサイトのURLを入力する操作をシミュレートすることで、指定したURLをWayback Machineに保存します。

```python
from selenium.webdriver import Firefox
from selenium.webdriver.common.keys import Keys


def snapshot_wayback(driver, url):

    driver.get("https://web.archive.org/")
    elem = driver.find_element_by_class_name('web-save-url-input')
    elem.clear()
    elem.send_keys(url)
    elem.send_keys(Keys.RETURN)
    driver.close()


driver = Firefox()
url = 'https://hacker-tools.github.io'
snapshot_wayback(driver, url)
```


## 演習

1. Webブラウザでよく使う検索エンジンのキーワードを編集してください
1. 上で挙げた拡張機能をインストールしてください。uBlock OriginやPrivacy Badgerを特定のWebサイトで無効にする方法を調べてみましょう。どのような違いが見られますか?YouTubeのように広告の多いWebサイトで試してみてください。
1. Stylusをインストールし、掲載したCSSを使って授業のWebサイト用のカスタムスタイルを書いてください。プログラミングでよく使われる文字をいくつか挙げます: `=   ==   ===   >=   =>   ++   /=   ~=`。フォントをFira Codeに変更すると、これらはどうなるでしょうか?もっと知りたい場合は、プログラミングフォントの合字(リガチャ)について検索してみてください。
1. 自分の住む都市・地域の天気を取得できるWeb APIを見つけてください。
1. [Selenium](https://www.selenium.dev/documentation/)のようなWebDriverソフトウェアを使って、ブラウザでよく行っている繰り返しの手作業を自動化してください。


