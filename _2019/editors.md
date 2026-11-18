---
layout: lecture
title: "エディタ"
presenter: Anish
date: 2019-01-22
order: 1
video:
  aspect: 62.5
  id: 1vLcusYSrI4
---

# エディタの重要性

プログラマーである私たちは、ほとんどの時間をプレーンテキストファイルの編集に費やします。自分のニーズに合ったエディタを学ぶことに時間を投資する価値があります。

新しいエディタはどうやって学べばよいのでしょうか? 一時的に生産性が落ちるとしても、しばらくのあいだ無理にでもそのエディタを使い続けるのです。すぐに元は取れます(基本を身につけるには2週間もあれば十分です)。

これからVimを教えますが、他のエディタも試してみることをおすすめします。エディタ選びはとても個人的な選択であり、人々は[強いこだわり](https://en.wikipedia.org/wiki/Editor_war)を持っています。

強力なエディタの使い方を50分で教えることはできないので、基本を教え、より高度な機能をいくつか紹介し、このツールを習得するための参考資料を提供することに焦点を当てます。Vimを題材にして教えますが、ほとんどの考え方は、あなたが使う他のどんな強力なエディタにも応用できます(もし応用できないなら、おそらくそのエディタは使うべきではありません!)。

![エディタの学習曲線](/2019/files/editor-learning-curves.jpg)

<!-- 出典: https://blogs.msdn.microsoft.com/steverowe/2004/11/17/code-editor-learning-curves/ -->

このエディタの学習曲線のグラフは作り話です。強力なエディタの基本を学ぶのはとても簡単です(使いこなせるようになるには何年もかかるかもしれませんが)。

現在人気のあるエディタはどれでしょうか? こちらの[Stack Overflowの調査](https://insights.stackoverflow.com/survey/2018/#development-environments-and-tools)を見てください(Stack Overflowのユーザーがプログラマー全体を代表しているとは限らないため、多少の偏りがあるかもしれません)。

## コマンドラインエディタ

最終的にGUIエディタに落ち着くとしても、リモートマシン上のファイルを手軽に編集できるように、コマンドラインエディタを学んでおく価値があります。

# Nano

Nanoはシンプルなコマンドラインエディタです。

- 矢印キーで移動
- その他のショートカット(保存、終了)はすべて画面下部に表示される

# Vim

Vi/Vimは強力なテキストエディタです。たいていどこにでもインストールされているコマンドラインプログラムなので、リモートマシン上のファイルを編集するのに便利です。

Vimには、GVimや[MacVim](https://macvim-dev.github.io/macvim/)のようなグラフィカル版もあります。これらは、24ビットカラー、メニュー、ポップアップといった追加機能を提供します。

## Vimの哲学

- プログラミングでは、ほとんどの時間を書くことではなく読むこと/編集することに費やす
    - Vimは **モーダル** エディタ: テキストの挿入とテキストの操作とで異なるモードを持つ
- Vimはプログラム可能(Vimscriptのほか、Pythonのような他の言語でも)
- Vimのインターフェースそのものがプログラミング言語のようなもの
    - キー入力(覚えやすい名前が付いている)がコマンド
    - コマンドは合成できる
- マウスは使わない: 遅すぎる
- エディタは思考の速さで動くべき

## Vim入門

### モード

Vimは現在のモードを左下に表示します。

- Normalモード: ファイル内の移動と編集のため
    - ほとんどの時間をここで過ごす
- Insertモード: テキストの挿入のため
- Visual(通常・行・ブロック)モード: テキストのブロックの選択のため

モードの切り替えは、どのモードからでも `<ESC>` を押せばNormalモードに戻れます。Normalモードからは、`i` でInsertモード、`v` でVisualモード、`V` でVisual Lineモード、`<C-v>` でVisual Blockモードに入ります。

Vimを使っていると `<ESC>` キーを頻繁に使います。Caps LockをEscapeにリマップすることを検討してください。

### 基本

Vimのexコマンドは、Normalモードで `:{command}` と入力して実行します。

- `:q` 終了(ウィンドウを閉じる)
- `:w` 保存
- `:wq` 保存して終了
- `:e {name of file}` 編集するファイルを開く
- `:ls` 開いているバッファを表示
- `:help {topic}` ヘルプを開く
    - `:help :w` はexコマンド `:w` のヘルプを開く
    - `:help w` は移動コマンド `w` のヘルプを開く

### 移動

Vimの本質は効率的な移動にあります。ファイル内の移動はNormalモードで行います。

- 悪い癖がつかないように矢印キーを無効にする
```vim
nnoremap <Left> :echoe "Use h"<CR>
nnoremap <Right> :echoe "Use l"<CR>
nnoremap <Up> :echoe "Use k"<CR>
nnoremap <Down> :echoe "Use j"<CR>
```
- 基本の移動: `hjkl`(左、下、上、右)
- 単語: `w`(次の単語)、`b`(単語の先頭)、`e`(単語の末尾)
- 行: `0`(行頭)、`^`(行の最初の非空白文字)、`$`(行末)
- 画面: `H`(画面の上端)、`M`(画面の中央)、`L`(画面の下端)
- ファイル: `gg`(ファイルの先頭)、`G`(ファイルの末尾)
- 行番号: `:{number}<CR>` または `{number}G`({number} 行目へ)
- その他: `%`(対応する相手へ)
- 検索(行内): `f{character}`、`t{character}`、`F{character}`、`T{character}`
    - 現在の行で {character} を前方/後方にfind/toで探す
- N回の繰り返し: `{number}{movement}`、たとえば `10j` は10行下に移動する
- 検索: `/{regex}`、`n` / `N` でマッチ間を移動

### 選択

ビジュアルモード:

- Visual
- Visual Line
- Visual Block

移動キーを使って選択範囲を作れます。

### テキストの操作

これまでマウスでやっていたことはすべて、キーボードで(そして強力で合成可能なコマンドで)行うようになります。

- `i` でInsertモードに入る
    - ただしテキストの操作や削除には、バックスペース以上のものを使いたい
- `o` / `O` で下 / 上に行を挿入
- `d{motion}` で {motion} を削除
    - 例: `dw` は単語を削除、`d$` は行末まで削除、`d0` は行頭まで削除
- `c{motion}` で {motion} を変更
    - 例: `cw` は単語を変更
    - `d{motion}` のあとに `i` を続けるのと同様
- `x` で文字を削除(`dl` と同等)
- `s` で文字を置き換え(`xi` と同等)
- Visualモード + 操作
    - テキストを選択して、`d` で削除、`c` で変更
- `u` で取り消し(undo)、`<C-r>` でやり直し(redo)
- 学ぶことはまだたくさんある: たとえば `~` は文字の大文字・小文字を反転する

### 参考資料

- `vimtutor` vimを教えてくれるコマンドラインプログラム
- [Vim Adventures](https://vim-adventures.com/) Vimを学ぶためのゲーム

## Vimのカスタマイズ

Vimは、`~/.vimrc` にあるプレーンテキストの設定ファイル(Vimscriptのコマンドを含みます)でカスタマイズします。有効にしておきたい基本的な設定は、おそらくたくさんあるでしょう。

GitHubにある他の人のドットファイルを見てヒントを得るとよいですが、他人の設定を丸ごとコピー&ペーストするのは避けましょう。読んで、理解して、必要なものだけを取り入れてください。

検討するとよいカスタマイズをいくつか挙げます。

- シンタックスハイライト: `syntax on`
- カラースキーム
- 行番号: `set nu` / `set rnu`
- あらゆるものをバックスペースで消せるようにする: `set backspace=indent,eol,start`

## 高度なVim

ここでは、このエディタの威力を示す例をいくつか紹介します。この種のことをすべて教えることはできませんが、使っていくうちに身についていきます。良い経験則があります。エディタを使っていて「もっと良い方法があるはずだ」と思ったら、おそらく実際にあります。オンラインで調べてみましょう。

### 検索と置換

`:s`(substitute)コマンド([ドキュメント](https://vim.fandom.com/wiki/Search_and_replace))。

- `%s/foo/bar/g`
    - ファイル全体でfooをbarに置換する
- `%s/\[.*\](\(.*\))/\1/g`
    - 名前付きのMarkdownリンクをただのURLに置換する

### 複数ウィンドウ

- `sp` / `vsp` でウィンドウを分割
- 同じバッファに対して複数のビューを持てる。

### マウスサポート

- `set mouse+=a`
    - クリック、スクロール、選択ができる

### マクロ

- `q{character}` でレジスタ `{character}` へのマクロの記録を開始
- `q` で記録を停止
- `@{character}` でマクロを再生
- マクロの実行はエラーが起きると停止する
- `{number}@{character}` でマクロを {number} 回実行
- マクロは再帰的にできる
    - まず `q{character}q` でマクロをクリアする
    - マクロを記録し、その中で `@{character}` を使ってマクロを再帰的に呼び出す(記録が完了するまでは何もしない)
- 例: XMLをJSONに変換する([ファイル](/2019/files/example-data.xml))
    - キー「name」/「email」を持つオブジェクトの配列
    - Pythonプログラムを使う?
    - sed / 正規表現を使う
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - ...
    - Vimのコマンド / マクロ
        - `Gdd`、`ggdd` で最初と最後の行を削除
        - 1つの要素を整形するマクロ(レジスタ `e`)
            - `<name>` のある行に移動
            - `qe^r"f>s": "<ESC>f<C"<ESC>q`
        - 1人分のpersonを整形するマクロ
            - `<person>` のある行に移動
            - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        - 1人分のpersonを整形して次のpersonに進むマクロ
            - `<person>` のある行に移動
            - `qq@pjq`
        - ファイルの末尾までマクロを実行
            - `999@q`
        - 最後の `,` を手作業で取り除き、区切り文字の `[` と `]` を追加する

## Vimの拡張

vimを拡張するためのプラグインは大量にあります。

まず、[vim-plug](https://github.com/junegunn/vim-plug)、[Vundle](https://github.com/VundleVim/Vundle.vim)、[pathogen.vim](https://github.com/tpope/vim-pathogen)のようなプラグインマネージャを導入しましょう。

検討するとよいプラグインをいくつか挙げます。

- [ctrlp.vim](https://github.com/kien/ctrlp.vim): ファジーファイルファインダー
- [vim-fugitive](https://github.com/tpope/vim-fugitive): gitとの統合
- [vim-surround](https://github.com/tpope/vim-surround): 「囲み」の操作
- [gundo.vim](https://github.com/sjl/gundo.vim): undoツリーの移動
- [nerdtree](https://github.com/scrooloose/nerdtree): ファイルエクスプローラ
- [syntastic](https://github.com/vim-syntastic/syntastic): 構文チェック
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): 魔法のようなモーション
- [vim-over](https://github.com/osyo-manga/vim-over): 置換のプレビュー

プラグインの一覧:

- [Vim Awesome](https://vimawesome.com/)

## 他のプログラムでのVimモード

人気のある多くのエディタ(たとえばvimやemacs)については、他の多くのツールがそのエディタのエミュレーションをサポートしています。

- シェル
    - bash: `set -o vi`
    - zsh: `bindkey -v`
    - `export EDITOR=vim`(`git` のようなプログラムが使う環境変数)
- `~/.inputrc`
    - `set editing-mode vi`

Web[ブラウザ](https://vim.fandom.com/wiki/Vim_key_bindings_for_web_browsers)向けのvimキーバインド拡張機能まであります。人気があるのは、Google Chrome向けの[Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en)やFirefox向けの[Tridactyl](https://github.com/tridactyl/tridactyl)です。


## 参考資料

- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2018/): さまざまなVimのtips
- [Neovim](https://neovim.io/)は、より活発に開発が行われている、モダンなvimの再実装です。
- [Vim Golf](https://www.vimgolf.com/): さまざまなVimの課題

{% comment %}
# 参考資料

TODO 他のエディタ向けの参考資料?
{% endcomment %}

# 演習

1. いくつかのエディタを試してみましょう。コマンドラインエディタ(たとえばVim)を少なくともひとつ、GUIエディタ(たとえばAtom)を少なくともひとつ試してください。`vimtutor`(または他のエディタでそれに相当するもの)のようなチュートリアルで学びましょう。新しいエディタの本当の感触をつかむために、普段の作業をしながら、数日間はそのエディタだけを使うと決めてください。

1. エディタをカスタマイズしましょう。オンラインでtipsやコツを調べ、他の人の設定にも目を通してみてください(多くの場合、よくドキュメント化されています)。

1. エディタのプラグインを試してみましょう。

1. 少なくとも数週間は強力なエディタを使い続けると決めてください。その頃には効果が実感できるようになっているはずです。いずれは、エディタを自分の思考と同じ速さで動かせるようになるでしょう。

1. リンタ(たとえばpython向けのpyflakes)をインストールしてエディタと連携させ、動作することを確かめましょう。
