---
layout: lecture
title: "バージョン管理とGit"
description: >
  Gitのデータモデルを学び、バージョン管理と共同作業のためのGitの使い方を身につけます。
thumbnail: /static/assets/thumbnails/2026/lec5.png
date: 2026-01-16
ready: true
video:
  aspect: 56.25
  id: 9K8lB61dl3Y
---

バージョン管理システム(VCS)は、ソースコード(あるいはその他のファイルやフォルダの集まり)への変更を追跡するためのツールです。その名前が示すとおり、これらのツールは変更の履歴を保持するのに役立ちます。さらに、共同作業も容易にしてくれます。論理的には、VCSはフォルダとその中身への変更を一連の「_スナップショット_」として追跡します。各スナップショットは、最上位ディレクトリ内のファイルやフォルダの状態全体を丸ごと収めたものです。VCSはまた、各スナップショットを誰が作成したか、各スナップショットに添えられたメッセージなどのメタデータも保持します。

バージョン管理はなぜ便利なのでしょうか? 一人で作業しているときでも、プロジェクトの過去のスナップショットを見たり、特定の変更を行った理由の記録を残したり、並行する開発ブランチで作業したりと、さまざまなことができます。他の人と作業するときには、他の人が何を変更したかを確認したり、並行開発で生じた競合を解決したりするうえで欠かせないツールです。

現代のVCSでは、次のような質問にも簡単に(しかも多くの場合、自動的に)答えられます。

- このモジュールを書いたのは誰か?
- この特定のファイルのこの特定の行は、いつ編集されたのか? 誰によって? なぜ編集されたのか?
- 直近1000リビジョンの間で、特定のユニットテストはいつ、なぜ動かなくなったのか?

他にもVCSは存在しますが、**Git** がバージョン管理のデファクトスタンダードです。この[XKCDのコミック](https://xkcd.com/1597/)は、Gitの評判をよく表しています。

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

Gitのインターフェースは「漏れのある抽象化(leaky abstraction)」であるため、Gitをトップダウンで(インターフェース、つまりコマンドラインインターフェースから)学ぼうとすると、大きな混乱を招きがちです。いくつかのコマンドを丸暗記して魔法の呪文のように扱い、何か問題が起きるたびに上のコミックのようなアプローチを取ることにもなりかねません。

Gitのインターフェースはお世辞にも美しいとは言えませんが、その根底にある設計と考え方は美しいものです。醜いインターフェースは「_暗記_」するしかありませんが、美しい設計は「_理解_」できます。そのため、この講義ではGitをボトムアップで説明します。まずデータモデルから始め、その後にコマンドラインインターフェースを扱います。データモデルを理解すれば、各コマンドが根底にあるデータモデルをどのように操作するのかという観点から、コマンドをよりよく理解できるようになります。

# Gitのデータモデル

Gitの独創性は、よく練られたデータモデルにあります。このデータモデルこそが、履歴の保持、ブランチのサポート、共同作業といったバージョン管理のあらゆる優れた機能を可能にしています。

## スナップショット

Gitは、ある最上位ディレクトリ内のファイルとフォルダの集まりの履歴を、一連のスナップショットとしてモデル化します。Gitの用語では、ファイルは「blob」と呼ばれ、単なるバイト列です。ディレクトリは「tree」と呼ばれ、名前をblobまたはtreeに対応づけます(こうしてディレクトリは他のディレクトリを含むことができます)。スナップショットとは、追跡対象となっている最上位のtreeのことです。たとえば、次のようなtreeがあるとします。

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

最上位のtreeには2つの要素が含まれています。1つはtree「foo」(それ自体が1つの要素、blob「bar.txt」を含む)で、もう1つはblob「baz.txt」です。

## 履歴のモデル化: スナップショット同士の関係づけ

バージョン管理システムはスナップショット同士をどのように関係づけるべきでしょうか? 単純なモデルの1つは、線形の履歴を持つことでしょう。この場合、履歴は時系列順に並んだスナップショットのリストになります。しかし、さまざまな理由から、Gitはこのような単純なモデルを採用していません。

Gitでは、履歴はスナップショットの有向非巡回グラフ(DAG)です。難しそうな数学用語に聞こえるかもしれませんが、身構える必要はありません。これが意味するのは、Gitの各スナップショットが「親」の集合、つまりそのスナップショットに先行するスナップショットたちを参照している、ということだけです。(線形の履歴のように)単一の親ではなく親の「集合」なのは、スナップショットが複数の親から派生する場合があるためです。たとえば、並行する2つの開発ブランチを統合(マージ)した場合がそうです。

Gitはこれらのスナップショットを「コミット」と呼びます。コミット履歴を可視化すると、次のようになるでしょう。

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

上のアスキーアートでは、`o` が個々のコミット(スナップショット)に対応します。矢印は各コミットの親を指しています(「後に来る」ではなく「前に来る」という関係です)。3番目のコミットの後、履歴は2つの独立したブランチに分岐しています。これはたとえば、2つの別々の機能が互いに独立して並行開発されている状況に対応します。将来、これらのブランチはマージされて、両方の機能を取り込んだ新しいスナップショットが作られるかもしれません。その結果、新しく作られたマージコミットを太字で示すと、履歴は次のようになります。

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</code>
</pre>

Gitのコミットは不変(イミュータブル)です。ただし、これは間違いを訂正できないという意味ではありません。コミット履歴への「編集」は、実際にはまったく新しいコミットを作成することであり、参照(後述)が新しいコミットを指すように更新される、というだけのことです。

## 疑似コードで表したデータモデル

Gitのデータモデルを疑似コードで書き下してみると、理解の助けになるかもしれません。

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

とてもクリーンでシンプルな履歴のモデルです。

## オブジェクトとコンテンツアドレッシング

「オブジェクト」とは、blob、tree、またはコミットのことです。

```
type object = blob | tree | commit
```

Gitのデータストアでは、すべてのオブジェクトはその[SHA-1ハッシュ](https://en.wikipedia.org/wiki/SHA-1)によってコンテンツアドレッシングされます。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

blob、tree、コミットは、この点で統一的に扱われます。すべてオブジェクトなのです。オブジェクトが他のオブジェクトを参照するとき、ディスク上の表現において実際にそれらを<em>含んでいる</em>わけではなく、ハッシュによる参照を持っています。

たとえば、[上の](#snapshots)例のディレクトリ構造に対応するtree(`git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` で可視化したもの)は次のようになります。

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

tree自体は、その中身である `baz.txt`(blob)と `foo`(tree)へのポインタを含んでいます。baz.txtに対応するハッシュが指す内容を `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85` で見てみると、次のようになります。

```
git is wonderful
```

## 参照

さて、これですべてのスナップショットをSHA-1ハッシュで識別できるようになりました。しかし、これは不便です。人間は40桁の16進数文字列を覚えるのが得意ではないからです。

この問題に対するGitの解決策が、SHA-1ハッシュに付ける人間が読める名前、すなわち「参照(references)」です。参照はコミットへのポインタです。不変であるオブジェクトとは異なり、参照は可変です(新しいコミットを指すように更新できます)。たとえば、`master` 参照は通常、開発のメインブランチにおける最新のコミットを指します。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

これにより、Gitは長い16進数文字列の代わりに、「master」のような人間が読める名前を使って履歴内の特定のスナップショットを参照できます。

1つ細かい点として、履歴の中で「今自分がどこにいるのか」という概念が欲しくなることがよくあります。そうすれば、新しいスナップショットを取るときに、それが何に対する相対なのか(コミットの `parents` フィールドをどう設定するか)が分かるからです。Gitでは、この「今いる場所」は「HEAD」と呼ばれる特別な参照です。

## リポジトリ

これでようやく、Gitの<em>リポジトリ</em>とは(おおまかに)何かを定義できます。それは、データ `objects` と `references` のことです。

ディスク上において、Gitが保存しているのはオブジェクトと参照だけです。Gitのデータモデルはこれがすべてです。すべての `git` コマンドは、オブジェクトを追加したり、参照を追加・更新したりすることによる、コミットDAGへの何らかの操作に対応しています。

コマンドを入力するときはいつでも、そのコマンドが根底にあるグラフデータ構造にどのような操作を加えているのかを考えてみてください。逆に、コミットDAGに対して特定の種類の変更を加えたい場合、たとえば「コミットされていない変更を破棄して、'master' 参照がコミット `5d83f9e` を指すようにしたい」という場合には、それを実現するコマンドがおそらく存在します(この例では `git checkout master; git reset
--hard 5d83f9e`)。

# ステージングエリア

これはデータモデルとは直交する別の概念ですが、コミットを作成するためのインターフェースの一部です。

上で説明したようなスナップショットの仕組みを実装する方法として、作業ディレクトリの<em>現在の状態</em>に基づいて新しいスナップショットを作る「スナップショット作成」コマンドを用意する、という方法が考えられます。実際にそのように動作するバージョン管理ツールもありますが、Gitは違います。私たちはクリーンなスナップショットが欲しいのであり、現在の状態からスナップショットを作ることが常に理想的とは限りません。たとえば、2つの別々の機能を実装し終えていて、それらを2つの別々のコミットに分けたい場合を考えてみてください。最初のコミットで1つ目の機能を導入し、次のコミットで2つ目の機能を導入したいわけです。あるいは、コード中のあちこちにデバッグ用のprint文を追加した状態でバグ修正も行っていて、print文はすべて破棄しつつバグ修正だけをコミットしたい場合もあるでしょう。

Gitは「ステージングエリア」と呼ばれる仕組みを通じて、次のスナップショットにどの変更を含めるかを指定できるようにすることで、こうしたシナリオに対応しています。

# Gitのコマンドラインインターフェース

情報の重複を避けるため、この講義ノートでは以下のコマンドを詳しくは説明しません。詳細は、強くおすすめする [Pro Git](https://git-scm.com/book/en/v2) を参照するか、講義ビデオをご覧ください。

## 基本

- `git help <command>`: gitコマンドのヘルプを表示する
- `git init`: 新しいgitリポジトリを作成する。データは `.git` ディレクトリに保存される
- `git status`: 現在の状況を教えてくれる
- `git add <filename>`: ファイルをステージングエリアに追加する
- `git commit`: 新しいコミットを作成する
    - [良いコミットメッセージ](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)を書きましょう!
    - [良いコミットメッセージ](https://chris.beams.io/posts/git-commit/)を書くべき理由はまだまだあります!
- `git log`: 履歴を平坦化されたログとして表示する
- `git log --all --graph --decorate`: 履歴をDAGとして可視化する
- `git diff <filename>`: ステージングエリアに対する自分の変更を表示する
- `git diff <revision> <filename>`: スナップショット間でのファイルの差分を表示する
- `git checkout <revision>`: HEADを更新する(ブランチをチェックアウトした場合は現在のブランチも更新する)

## ブランチとマージ

- `git branch`: ブランチの一覧を表示する
- `git branch <name>`: ブランチを作成する
- `git switch <name>`: ブランチに切り替える
- `git checkout -b <name>`: ブランチを作成して、そのブランチに切り替える
    - `git branch <name>; git switch <name>` と同じ
- `git merge <revision>`: 現在のブランチにマージする
- `git mergetool`: マージ競合の解決を助ける便利なツールを使う
- `git rebase`: 一連のパッチを新しいベースの上にリベースする

## リモート

- `git remote`: リモートの一覧を表示する
- `git remote add <name> <url>`: リモートを追加する
- `git push <remote> <local branch>:<remote branch>`: オブジェクトをリモートに送り、リモートの参照を更新する
- `git branch --set-upstream-to=<remote>/<remote branch>`: ローカルブランチとリモートブランチの対応関係を設定する
- `git fetch`: リモートからオブジェクトや参照を取得する
- `git pull`: `git fetch; git merge` と同じ
- `git clone`: リモートからリポジトリをダウンロードする

## 取り消し

- `git commit --amend`: コミットの内容やメッセージを編集する
- `git reset <file>`: ファイルをステージングから外す
- `git restore`: 変更を破棄する

# 高度なGit

- `git config`: Gitは[高度にカスタマイズ可能](https://git-scm.com/docs/git-config)です
- `git clone --depth=1`: 全バージョン履歴を含まないシャロークローン
- `git add -p`: 対話的なステージング
- `git rebase -i`: 対話的なリベース
- `git blame`: 各行を最後に編集したのが誰かを表示する
- `git stash`: 作業ディレクトリへの変更を一時的に退避する
- `git bisect`: 履歴を二分探索する(例: リグレッションの原因を探す)
- `git revert`: 以前のコミットの効果を打ち消す新しいコミットを作成する
- `git worktree`: 複数のブランチを同時にチェックアウトする
- `.gitignore`: 意図的に追跡しないファイルを[指定](https://git-scm.com/docs/gitignore)して無視する

# その他

- **GUI**: Gitには多くの[GUIクライアント](https://git-scm.com/downloads/guis)があります。私たち自身は使っておらず、代わりにコマンドラインインターフェースを使っています。
- **シェルとの統合**: Gitのステータスをシェルのプロンプトに表示できると非常に便利です([zsh](https://github.com/olivierverdier/zsh-git-prompt)、[bash](https://github.com/magicmonty/bash-git-prompt))。[Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) のようなフレームワークには最初から含まれていることが多いです。
- **エディタとの統合**: 上と同様に、多くの機能を備えた便利な統合があります。[fugitive.vim](https://github.com/tpope/vim-fugitive) がVimにおける定番です。
- **ワークフロー**: この講義ではデータモデルといくつかの基本コマンドを教えましたが、大きなプロジェクトで作業するときにどのような慣習に従うべきかは教えていません(そして、それには[数多くの](https://nvie.com/posts/a-successful-git-branching-model/)[異なる](https://www.endoflineblog.com/gitflow-considered-harmful)[アプローチ](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)があります)。
- **GitHub**: GitはGitHubではありません。GitHubには、他のプロジェクトにコードを寄贈するための独自の方法があり、[プルリクエスト](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)と呼ばれています。
- **その他のGitプロバイダ**: GitHubだけが特別なわけではありません。[GitLab](https://about.gitlab.com/) や [BitBucket](https://bitbucket.org/) など、Gitリポジトリのホスティングサービスは数多くあります。

# 参考資料

- [Pro Git](https://git-scm.com/book/en/v2) は**強くおすすめする読み物**です。データモデルを理解した今なら、第1章から第5章までを読み通せば、Gitを使いこなすのに必要なことのほとんどが身につくはずです。後半の章には、興味深い高度な内容が書かれています。
- [Oh Shit, Git!?!](https://ohshitgit.com/) は、Gitでよくある失敗から復旧する方法についての短いガイドです。
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) は、Gitのデータモデルの短い解説です。この講義ノートよりも疑似コードは少なめで、しゃれた図が多く使われています。
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) は、興味のある人向けに、データモデルにとどまらないGitの実装の詳細を解説したものです。
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) は、Gitを教えてくれるブラウザベースのゲームです。

# 演習

1. Gitを使った経験がまったくない場合は、[Pro Git](https://git-scm.com/book/en/v2) の最初の数章を読むか、[Learn Git Branching](https://learngitbranching.js.org/) のようなチュートリアルをやってみてください。取り組みながら、Gitのコマンドをデータモデルと関連づけて考えてみましょう。
1. [この授業のWebサイトのリポジトリ](https://github.com/missing-semester/missing-semester)をクローンしてください。
    1. バージョン履歴をグラフとして可視化して探索してみましょう。
    1. `README.md` を最後に変更したのは誰ですか?(ヒント: `git log` に引数を付けて使いましょう。)
    1. `_config.yml` の `collections:` の行が最後に変更されたときのコミットメッセージは何でしたか?(ヒント: `git blame` と `git show` を使いましょう。)
1. Gitを学ぶ際によくある失敗の1つが、Gitで管理すべきではない大きなファイルをコミットしてしまったり、機密情報を追加してしまったりすることです。リポジトリにファイルを追加していくつかコミットを作った後、そのファイルを(最新のコミットからだけでなく)<em>履歴</em>から削除してみてください。[こちら](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)が参考になるでしょう。
1. GitHubから適当なリポジトリをクローンし、既存のファイルの1つを変更してください。`git stash` を実行すると何が起こりますか? `git log
   --all --oneline` を実行すると何が表示されますか? `git stash pop` を実行して、`git stash` で行ったことを元に戻してください。これはどのような場面で役に立つでしょうか?
1. 多くのコマンドラインツールと同様に、Gitは `~/.gitconfig` という設定ファイル(dotfile)を提供しています。`git graph` を実行すると `git log --all --graph --decorate
   --oneline` の出力が得られるように、`~/.gitconfig` にエイリアスを作成してください。これは `~/.gitconfig` ファイルを直接[編集](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias)するか、`git config` コマンドを使ってエイリアスを追加することで実現できます。gitのエイリアスについての情報は[こちら](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)にあります。
1. `git config --global core.excludesfile ~/.gitignore_global` を実行すると、`~/.gitignore_global` にグローバルな無視パターンを定義できます。これはGitが使用するグローバル無視ファイルの場所を設定するだけなので、そのパスにファイルを自分で作成する必要があります。`.DS_Store` のような、OS固有あるいはエディタ固有の一時ファイルを無視するように、グローバルなgitignoreファイルを設定してください。
1. [この授業のWebサイトのリポジトリ](https://github.com/missing-semester/missing-semester)をフォークし、タイポやその他の改善できる点を見つけて、GitHubでプルリクエストを送ってください([こちら](https://github.com/firstcontributions/first-contributions)が参考になるでしょう)。有用なPRだけを送ってください(スパムはやめてくださいね!)。改善点が見つからない場合は、この演習は飛ばしてかまいません。
1. 共同作業のシナリオをシミュレートして、マージ競合の解決を練習しましょう。
    1. `git init` で新しいリポジトリを作成し、数行の内容(たとえば簡単なレシピ)を書いた `recipe.txt` というファイルを作成します。
    1. それをコミットしてから、`git branch salty` と `git branch
       sweet` で2つのブランチを作成します。
    1. `salty` ブランチで、ある行を変更して(たとえば「1 cup sugar」を「1
       cup salt」に変えて)コミットします。
    1. `sweet` ブランチで、同じ行を別の内容に変更して(たとえば「1
       cup sugar」を「2 cups sugar」に変えて)コミットします。
    1. 次に `master` に切り替えて、`git merge salty`、続いて `git merge
       sweet` を試してみてください。何が起こるでしょうか? `recipe.txt` の中身を見てみましょう。`<<<<<<<`、`=======`、`>>>>>>>` のマーカーは何を意味しているのでしょうか?
    1. 残したい内容だけが残るようにファイルを編集し、競合マーカーを削除して、`git add` と `git commit`(または `git merge --continue`)でマージを完了させることで、競合を解決してください。あるいは、`git mergetool` を使って、グラフィカルまたはターミナルベースのマージツールで競合を解決してみてください。
    1. `git log --graph --oneline` を使って、いま作成したマージ履歴を可視化してみましょう。
