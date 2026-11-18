---
layout: lecture
title: "OSのカスタマイズ"
presenter: Anish
date: 2019-01-29
order: 3
video:
  aspect: 62.5
  id: epSRVqQzeDo
special: true
---

オペレーティングシステムは、設定メニューで用意されているもの以外にも、さまざまな形でカスタマイズできます。

# キーボードのリマッピング

あなたのキーボードには、おそらくあまり使っていないキーがあるでしょう。役に立たないキーをそのままにしておく代わりに、それらをリマップして役に立つことをさせられます。

## 他のキーへのリマッピング

もっとも単純なのは、キーを他のキーにリマップすることです。たとえば、Caps Lockキーをあまり使わないなら、もっと役に立つものにリマップできます。たとえばVimユーザーであれば、Caps LockをEscapeにリマップしたくなるかもしれません。

macOSでは、システム環境設定のキーボード設定からいくつかのリマッピングが行えます。より複雑なマッピングには、専用のソフトウェアが必要です。

## 任意のコマンドへのリマッピング

キーは他のキーにリマップできるだけではありません。キー(またはキーの組み合わせ)を任意のコマンドにリマップできるツールもあります。たとえば、command-shift-tで新しいターミナルウィンドウが開くようにできます。

# OSの隠し設定のカスタマイズ

## macOS

macOSは、`defaults` コマンドを通じて多くの便利な設定を公開しています。たとえば、非表示にしたアプリケーションのDockアイコンを半透明にできます。

```shell
defaults write com.apple.dock showhidden -bool true
```

設定可能な項目をすべて網羅した単一のリストはありませんが、Mathias Bynensさんの[.macos](https://github.com/mathiasbynens/dotfiles/blob/master/.macos)のように、特定のカスタマイズをまとめたリストがオンラインで見つかります。

# ウィンドウ管理

## タイル型ウィンドウ管理

[タイル型ウィンドウ管理](https://en.wikipedia.org/wiki/Tiling_window_manager)はウィンドウ管理の手法のひとつで、ウィンドウを互いに重ならないフレームに配置します。Unix系のオペレーティングシステムを使っているなら、タイル型ウィンドウマネージャをインストールできます。WindowsやmacOSのようなものを使っている場合は、この挙動に近いことを実現するアプリケーションをインストールできます。

## 画面の管理

複数の画面をまたいでウィンドウを操作しやすくするためのキーボードショートカットを設定できます。

## レイアウト

画面上のウィンドウの配置に決まったやり方があるなら、そのレイアウトを手作業で「実行」する代わりにスクリプト化でき、レイアウトの再現がごく簡単になります。

# 参考資料

- [Hammerspoon](https://www.hammerspoon.org/) - macOSのデスクトップ自動化
- [Rectangle](https://rectangleapp.com/) - macOSのウィンドウマネージャ
- [Karabiner](https://karabiner-elements.pqrs.org/) - macOS向けの高機能なキーボードリマッピング
- [r/unixporn](https://www.reddit.com/r/unixporn/) - 人々の凝った設定のスクリーンショットとドキュメント

# 演習

1. Caps Lockキーを、もっとよく使うもの(EscapeやCtrl、Backspaceなど)にリマップする方法を調べてください。

1. 新しいターミナルウィンドウまたは新しいブラウザウィンドウを開く、独自のグローバルキーボードショートカットを作ってください。

{% comment %}

TODO

- Bitbar / Polybar
- クリップボードマネージャ(スタック/検索可能な履歴)

{% endcomment %}
