---
layout: lecture
title: "開発環境とツール"
description: >
  IDE、Vim、言語サーバー、AIを活用した開発ツールについて学びます。
thumbnail: /static/assets/thumbnails/2026/lec3.png
date: 2026-01-14
ready: true
video:
  aspect: 56.25
  id: QnM1nVzrkx8
---

_開発環境_ とは、ソフトウェアを開発するためのツール一式のことです。開発環境の中心にあるのはテキスト編集の機能で、そこにシンタックスハイライト、型チェック、コードフォーマット、オートコンプリートといった付随機能が加わります。[VS Code][vs-code]のような _統合開発環境_(IDE)は、これらすべての機能を1つのアプリケーションにまとめたものです。ターミナルベースの開発ワークフローでは、[tmux](https://github.com/tmux/tmux)(ターミナルマルチプレクサ)、[Vim](https://www.vim.org/)(テキストエディタ)、[Zsh](https://www.zsh.org/)(シェル)のようなツールに加えて、[Ruff](https://docs.astral.sh/ruff/)(Pythonのリンタ兼コードフォーマッタ)や[Mypy](https://mypy-lang.org/)(Pythonの型チェッカー)のような言語固有のコマンドラインツールを組み合わせて使います。

IDEとターミナルベースのワークフローには、それぞれ長所と短所があります。たとえば、グラフィカルなIDEは習得しやすく、今日のIDEはAIオートコンプリートのようなAI統合機能が最初からよく整っている傾向があります。一方、ターミナルベースのワークフローは軽量で、GUIがない環境やソフトウェアをインストールできない環境では唯一の選択肢になることもあります。両方の基本的な使い方に慣れたうえで、少なくともどちらか一方を使いこなせるようになることをお勧めします。まだお気に入りのIDEがないなら、[VS Code][vs-code]から始めるとよいでしょう。

この講義では、次の内容を扱います。

- [テキスト編集とVim](#text-editing-and-vim)
- [コードインテリジェンスと言語サーバー](#code-intelligence-and-language-servers)
- [AIを活用した開発](#ai-powered-development)
- [拡張機能とその他のIDE機能](#extensions-and-other-ide-functionality)

[vs-code]: https://code.visualstudio.com/

# テキスト編集とVim {#text-editing-and-vim}

プログラミング中の時間の大半は、長い文章を一気に書いたりファイルを頭から順に読んだりすることではなく、コードの中を移動し、コードの断片を読み、コードに編集を加えることに費やされます。[Vim] は、このようなタスクの分布に最適化されたテキストエディタです。

**Vimの哲学。** Vimの根底には美しいアイデアがあります。そのインターフェース自体が、テキストの移動と編集のために設計されたプログラミング言語なのです。キー入力(覚えやすい名前が付いています)はコマンドであり、これらのコマンドは合成可能です。Vimはマウスの使用を避けます。遅すぎるからです。Vimは矢印キーの使用さえ避けます。手の移動が大きすぎるからです。その結果として生まれるのは、まるでブレイン・コンピュータ・インターフェースのように感じられ、思考の速度に追従するエディタです。

**他のソフトウェアでのVimサポート。** Vimの核にあるアイデアの恩恵を受けるのに、[Vim] そのものを使う必要はありません。テキスト編集を伴う多くのプログラムが、組み込み機能またはプラグインとして「Vimモード」をサポートしています。たとえば、VS Codeには[VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim)プラグインがあり、ZshにはVimエミュレーションの[組み込みサポート](https://zsh.sourceforge.io/Guide/zshguide04.html)があり、Claude CodeにさえVimエディタモードの[組み込みサポート](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode)があります。あなたが使うテキスト編集を伴うツールなら、何らかの形でVimモードをサポートしている可能性が高いでしょう。

## モーダル編集

Vimは _モーダルエディタ_ です。タスクの種類ごとに異なる操作モードを持っています。

- **ノーマル(Normal)**: ファイル内の移動と編集のため
- **挿入(Insert)**: テキストの入力のため
- **置換(Replace)**: テキストの置き換えのため
- **ビジュアル(Visual)**(通常・行・ブロック): テキストのブロックの選択のため
- **コマンドライン(Command-line)**: コマンドの実行のため

キー入力の意味は操作モードによって異なります。たとえば、文字 `x` はInsertモードでは単に文字「x」を入力しますが、Normalモードではカーソル位置の文字を削除し、Visualモードでは選択範囲を削除します。

デフォルト設定のVimは、現在のモードを左下に表示します。起動時のデフォルトのモードはNormalモードです。ほとんどの時間はNormalモードとInsertモードの行き来に費やすことになるでしょう。

モードの切り替えは、どのモードからでも `<ESC>`(エスケープキー)を押せばNormalモードに戻れます。Normalモードからは、`i` でInsertモード、`R` でReplaceモード、`v` でVisualモード、`V` でVisual Lineモード、`<C-v>`(Ctrl-V、`^V` と書かれることもあります)でVisual Blockモード、`:` でCommand-lineモードに入ります。

Vimを使っていると `<ESC>` キーを多用することになります。Caps LockをEscapeに割り当て直す([macOSでの手順](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS))か、簡単なキーシーケンスによる `<ESC>` の[代替マッピング](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings)を作ることを検討しましょう。

## 基本: テキストの入力

Normalモードから `i` を押すとInsertモードに入ります。この状態では、`<ESC>` を押してNormalモードに戻るまで、Vimは他の一般的なテキストエディタと同じように振る舞います。これと上で説明した基本事項さえ知っていれば、Vimでファイルの編集を始められます(ただし、ずっとInsertモードで編集しているようでは、あまり効率的とは言えません)。

## Vimのインターフェースはプログラミング言語である

Vimのインターフェースはプログラミング言語です。キー入力(覚えやすい名前が付いています)はコマンドであり、これらのコマンドは _合成_ できます。これにより、効率的な移動と編集が可能になります。コマンドが体に染み付いてしまえばなおさらです。キーボード配列を覚えるとタイピングが格段に速くなるのと同じです。

### 移動

ほとんどの時間はNormalモードで過ごし、移動コマンドを使ってファイル内を動き回るべきです。Vimの移動コマンドは「名詞」とも呼ばれます。テキストのまとまりを指し示すからです。

- 基本の移動: `hjkl`(左、下、上、右)
- 単語: `w`(次の単語)、`b`(単語の先頭)、`e`(単語の末尾)
- 行: `0`(行頭)、`^`(行の最初の非空白文字)、`$`(行末)
- 画面: `H`(画面の上端)、`M`(画面の中央)、`L`(画面の下端)
- スクロール: `Ctrl-u`(上)、`Ctrl-d`(下)
- ファイル: `gg`(ファイルの先頭)、`G`(ファイルの末尾)
- 行番号: `:{number}<CR>` または `{number}G`({number} 行目へ)
    - `<CR>` はキャリッジリターン / Enterキーを指します
- その他: `%`(括弧やブレースなどの対応する相手へ)
- 検索(行内): `f{character}`、`t{character}`、`F{character}`、`T{character}`
    - 現在の行で {character} を前方/後方にfind/toで探します
    - `,` / `;` でマッチ間を移動します
- 検索: `/{regex}`、`n` / `N` でマッチ間を移動します

### 選択

ビジュアルモード:

- Visual: `v`
- Visual Line: `V`
- Visual Block: `Ctrl-v`

移動キーを使って選択範囲を作れます。

### 編集

これまでマウスでやっていたことはすべて、移動コマンドと合成できる編集コマンドを使ってキーボードで行うようになります。ここからが、Vimのインターフェースがプログラミング言語のように見えてくるところです。Vimの編集コマンドは「動詞」とも呼ばれます。動詞は名詞に作用するからです。

- `i` でInsertモードに入る
    - ただしテキストの操作や削除には、バックスペース以上のものを使いたいところです
- `o` / `O` で下 / 上に行を挿入
- `d{motion}` で {motion} を削除
    - 例: `dw` は単語を削除、`d$` は行末まで削除、`d0` は行頭まで削除
- `c{motion}` で {motion} を変更
    - 例: `cw` は単語を変更
    - `d{motion}` のあとに `i` を続けるのと同様
- `x` で文字を削除(`dl` と同等)
- `s` で文字を置き換え(`cl` と同等)
- Visualモード + 操作
    - テキストを選択して、`d` で削除、`c` で変更
- `u` で取り消し(undo)、`<C-r>` でやり直し(redo)
- `y` でコピー / 「ヤンク」(`d` など一部の他のコマンドもコピーを行います)
- `p` で貼り付け
- 学ぶことはまだたくさんあります: たとえば `~` は文字の大文字・小文字を反転し、`J` は行を連結します

### カウント

名詞と動詞にはカウントを組み合わせることができ、指定した操作を指定回数実行できます。

- `3w` で3単語先に進む
- `5j` で5行下に移動
- `7dw` で7単語を削除

### 修飾子

修飾子を使って名詞の意味を変えられます。修飾子には、「inner」または「inside」(内側)を意味する `i` や、「around」(周りごと)を意味する `a` があります。

- `ci(` で現在の丸括弧のペアの内側の内容を変更
- `ci[` で現在の角括弧のペアの内側の内容を変更
- `da'` でシングルクォート文字列を、囲んでいるシングルクォートごと削除

## すべてを組み合わせる

以下は壊れた[fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz)の実装です。

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20)
```

Normalモードから始めて、次の一連のコマンドで問題を修正していきます。

- mainが呼ばれていない
    - `G` でファイルの末尾にジャンプ
    - `o` で下に新しい行を開く(**o**pen)
    - `if __name__ == "__main__": main()` と入力
        - エディタにPythonの言語サポートがあれば、Insertモードで自動インデントしてくれるかもしれません
    - `<ESC>` でNormalモードに戻る
- 1ではなく0から始まっている
    - `/` に続けて `range` と `<CR>` で「range」を検索
    - `ww` で2単語(**w**ord)先に進む(`2w` でも構いませんが、実際には小さいカウントではカウント機能を使うよりキーを繰り返し押すことが多いです)
    - `i` で **i**nsertモードに切り替えて `1,` を追加
    - `<ESC>` でNormalモードに戻る
    - `e` で次の単語の末尾(**e**nd)にジャンプ
    - `a` でテキストの追記(**a**ppend)を始めて `+ 1` を追加
    - `<ESC>` でNormalモードに戻る
- 5の倍数で「fizz」と出力される
    - `:6<CR>` で6行目へ移動
    - `ci"` で「**"**」の内側(**i**nside)を変更(**c**hange)し、`"buzz"` に書き換える
    - `<ESC>` でNormalモードに戻る

## Vimを学ぶ

Vimを学ぶ最良の方法は、基本(ここまでで扱った内容)を学んだら、あとは使うすべてのソフトウェアでVimモードを有効にして、実践の中で使い始めることです。マウスや矢印キーを使いたくなる誘惑に耐えましょう。エディタによっては矢印キーのバインドを解除して、良い習慣を強制的に身につけることもできます。

### 追加リソース

- この講座の以前の年度の[Vimの講義]({{ site.baseurl }}/2020/editors/) --- そこではVimをより深く扱っています
- `vimtutor` はVimに付属するチュートリアルです --- Vimがインストールされていれば、シェルから `vimtutor` を実行できるはずです
- [Vim Adventures](https://vim-adventures.com/)はVimを学ぶためのゲームです
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/)には様々なVimのtipsがあります
- [VimGolf](https://www.vimgolf.com/)は[code golf](https://en.wikipedia.org/wiki/Code_golf)の一種で、プログラミング言語がVimのUIになったものです
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/)(書籍)

[Vim]: https://www.vim.org/

# コードインテリジェンスと言語サーバー {#code-intelligence-and-language-servers}

IDEは一般に、コードの意味的な理解を必要とする言語固有のサポートを提供します。これは、[Language Server Protocol](https://microsoft.github.io/language-server-protocol/)を実装した _言語サーバー_ に接続するIDE拡張機能を通じて実現されます。たとえば、[VS CodeのPython拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-python.python)は[Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)に依存しており、[VS CodeのGo拡張機能](https://marketplace.visualstudio.com/items?itemName=golang.go)は公式の[gopls](https://go.dev/gopls/)に依存しています。使っている言語の拡張機能と言語サーバーをインストールすることで、次のような言語固有の機能をIDEで使えるようになります。

- **コード補完。** より優れたオートコンプリートとサジェスト。たとえば `object.` と入力するとオブジェクトのフィールドやメソッドが見られます。
- **インラインドキュメント。** ホバー時やサジェスト時にドキュメントを表示します。
- **定義へのジャンプ。** 使用箇所から定義へジャンプします。たとえばフィールド参照 `object.field` からそのフィールドの定義へ移動できます。
- **参照の検索。** 上記の逆で、特定のフィールドや型などの項目が参照されているすべての箇所を見つけます。
- **インポートの支援。** インポートの整理、未使用のインポートの削除、不足しているインポートの指摘。
- **コード品質。** これらのツールは単体でも使えますが、この機能も言語サーバーによって提供されることがよくあります。コードフォーマットは入力中に自動インデントと自動整形を行い、型チェッカーとリンタは入力中にコードのエラーを見つけます。この種の機能については[コード品質の講義]({{ site.baseurl }}/2026/code-quality/)でより深く扱います。

## 言語サーバーの設定

言語によっては、拡張機能と言語サーバーをインストールするだけで準備完了です。他の言語では、言語サーバーの恩恵を最大限に受けるために、IDEにあなたの環境を教える必要があります。たとえば、VS Codeに[Python環境](https://code.visualstudio.com/docs/python/environments)の場所を指定すると、言語サーバーがインストール済みのパッケージを認識できるようになります。環境については[コードのパッケージングと配布の講義]({{ site.baseurl }}/2026/shipping-code/)でより深く扱います。

言語によっては、言語サーバーに設定できる項目があるかもしれません。たとえば、VS CodeのPythonサポートでは、Pythonのオプションの型アノテーションを使っていないプロジェクトに対して、静的型チェックを無効にできます。

# AIを活用した開発 {#ai-powered-development}

2021年半ばにOpenAIの[Codexモデル](https://openai.com/index/openai-codex/)を用いた[GitHub Copilot][github-copilot]が登場して以来、[LLM](https://en.wikipedia.org/wiki/Large_language_model)はソフトウェアエンジニアリングで広く採用されるようになりました。現在、主に3つの形態が使われています。オートコンプリート、インラインチャット、そしてコーディングエージェントです。

[github-copilot]: https://github.com/features/copilot/ai-code-editor

## オートコンプリート

AIによるオートコンプリートは、IDEの従来のオートコンプリートと同じ形態で、入力中のカーソル位置に補完を提案します。「ただ動く」受け身の機能として使われることもあります。それに加えて、AIオートコンプリートは一般にコードコメントを使って[プロンプト](https://en.wikipedia.org/wiki/Prompt_engineering)を与えられます。

例として、この講義ノートの内容をダウンロードしてすべてのリンクを抽出するスクリプトを書いてみましょう。まずこう書き始めます。

```python
import requests

def download_contents(url: str) -> str:
```

モデルは関数の本体をオートコンプリートします。

```python
    response = requests.get(url)
    return response.text
```

コメントを使って補完をさらに誘導できます。たとえば、すべてのMarkdownリンクを抽出する関数を書き始めたものの、あまり説明的でない名前を付けたとします。

```python
def extract(contents: str) -> list[str]:
```

モデルは次のような補完をします。

```python
    lines = contents.splitlines()
    return [line for line in lines if line.strip()]
```

コードコメントを通じて補完を誘導できます。

```python
def extract(content: str) -> list[str]:
    # extract all Markdown links from the content
```

今度は、モデルはより良い補完をしてくれます。

```python
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)
```

ここで、このAIコーディングツールの欠点の一つが見えてきます。補完はカーソル位置にしか提供できないのです。この場合、`import re` は関数の内側ではなく、モジュールのトップレベルに置く方が良い作法でしょう。

上の例では、コメントによってコード補完を誘導できることを示すために、わざと名前の悪い関数を使いました。実際には、`extract_links` のようにもっと説明的な名前の関数を書き、docstringも書くべきです(そうすれば、モデルはそれに基づいて上と同様の補完を生成するはずです)。

デモンストレーションのために、スクリプトを完成させましょう。

```python
print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

## インラインチャット

インラインチャットでは、行やブロックを選択して、AIモデルに編集の提案を直接プロンプトで指示できます。この対話モードでは、モデルは既存のコードに変更を加えられます(カーソルの先のコードを補完するだけのオートコンプリートとは異なる点です)。

上の例の続きとして、サードパーティの `requests` ライブラリを使わないことにしたとしましょう。該当する3行のコードを選択し、インラインチャットを呼び出して、たとえばこう伝えます。

```
use built-in libraries instead
```

モデルはこう提案します。

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')
```

## コーディングエージェント

コーディングエージェントについては、[エージェンティックコーディング]({{ site.baseurl }}/2026/agentic-coding/)の講義で詳しく扱います。

## おすすめのソフトウェア

人気のあるAI IDEには、[GitHub Copilot][github-copilot]拡張機能を入れた[VS Code][vs-code]や、[Cursor](https://cursor.com/)があります。GitHub Copilotは現在、[学生には無料](https://github.com/education/students)で提供されており、教員や人気オープンソースプロジェクトのメンテナも対象です。この分野は急速に進化しています。主要な製品の多くは、おおむね同等の機能を備えています。

# 拡張機能とその他のIDE機能 {#extensions-and-other-ide-functionality}

IDEは強力なツールですが、_拡張機能_ によってさらに強力になります。1回の講義でこれらの機能をすべて扱うことはできませんが、ここではいくつかの人気の拡張機能へのポインタを紹介します。この領域はぜひ自分で探索してみてください。人気のIDE拡張機能のリストはオンラインに数多くあります。たとえば、Vimプラグインなら[Vim Awesome](https://vimawesome.com/)、VS Codeなら[人気順に並べたVS Code拡張機能](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Installs)があります。

- [開発コンテナ(Development containers)](https://containers.dev/): 人気のIDEでサポートされており(例: [VS Codeでのサポート](https://code.visualstudio.com/docs/devcontainers/containers))、dev containerを使うとコンテナ内で開発ツールを実行できます。ポータビリティや分離のために役立ちます。[コードのパッケージングと配布の講義]({{ site.baseurl }}/2026/shipping-code/)でコンテナをより深く扱います。
- リモート開発: SSHを使ってリモートマシン上で開発します(例: [VS CodeのRemote SSHプラグイン](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh))。たとえば、クラウド上の強力なGPUマシンでコードを開発・実行したい場合に便利です。
- 共同編集: Google Docsのように同じファイルを同時に編集します(例: [VS CodeのLive Shareプラグイン](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare))。

# 演習

1. エディタやシェルなど、使っているソフトウェアのうちVimモードをサポートするものすべてでVimモードを有効にし、これから1か月間、すべてのテキスト編集をVimモードで行ってみましょう。何かが非効率に感じられたときや、「もっと良い方法があるはずだ」と思ったときは、ググってみてください。たいてい、もっと良い方法があります。
1. [VimGolf](https://www.vimgolf.com/)のチャレンジを1つクリアしてみましょう。
1. 取り組んでいるプロジェクトで、IDE拡張機能と言語サーバーを設定しましょう。ライブラリ依存関係に対する定義へのジャンプなど、期待されるすべての機能が正しく動くことを確認してください。この演習に使えるコードがない場合は、GitHubのオープンソースプロジェクト([こちら](https://github.com/spf13/cobra)など)を使うとよいでしょう。
1. IDE拡張機能のリストを眺めて、便利そうなものを1つインストールしてみましょう。
