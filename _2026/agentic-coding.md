---
layout: lecture
title: "エージェンティックコーディング"
description: >
  AIコーディングエージェントをソフトウェア開発のタスクに効果的に活用する方法を学びます。
thumbnail: /static/assets/thumbnails/2026/lec7.png
date: 2026-01-21
ready: true
video:
  aspect: 56.25
  id: sTdz6PZoAnw
---

コーディングエージェントとは、ファイルの読み書き、Web検索、シェルコマンドの実行といったツールへのアクセスを持つ、対話型のAIモデルです。IDEの中に組み込まれているものもあれば、スタンドアロンのコマンドラインツールやGUIツールとして提供されているものもあります。コーディングエージェントは高い自律性を持つ強力なツールであり、非常に幅広いユースケースを可能にします。

この講義は、[開発環境とツール]({{ site.baseurl }}/2026/development-environment/)の講義で扱ったAIを活用した開発の内容を発展させたものです。簡単なデモとして、[AIを活用した開発]({{ site.baseurl }}/2026/development-environment/#ai-powered-development)のセクションの例の続きから始めましょう。

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

コーディングエージェントに次のようなタスクをプロンプトとして与えてみましょう。

```
Turn this into a proper command-line program, with argparse for argument parsing. Add type annotations, and make sure the program passes type checking.
```

エージェントはまずファイルを読んで内容を理解し、次にいくつかの編集を行い、最後に型チェッカーを実行して型アノテーションが正しいことを確認します。型チェックに失敗するようなミスをした場合は、おそらく自分で修正を繰り返すでしょう(もっとも、これは単純なタスクなので、そうなる可能性は低いですが)。コーディングエージェントは有害になりうるツールへのアクセスを持つため、デフォルトでは、エージェントハーネスはツール呼び出しの前にユーザーに確認を求めます。

> コーディングエージェントがミスをした場合 --- たとえば、`mypy` バイナリが `$PATH` 上で直接利用できるのにエージェントが `python -m mypy` を呼び出そうとした場合など --- テキストでフィードバックを与えて軌道修正を促すことができます。

コーディングエージェントはマルチターンの対話に対応しているので、エージェントとのやり取りを重ねながら作業を進めていくことができます。エージェントが間違った方向に進んでいる場合には、途中で割り込むことも可能です。役に立つメンタルモデルのひとつは、インターンを指導するマネージャーというものです。インターンは細々とした作業をこなしてくれますが、指導が必要であり、ときには間違ったことをして修正が必要になります。

> より分かりやすいデモとして、続けてエージェントに、生成されたスクリプトを実行するよう頼んでみてください。出力を観察し、変更を依頼してみましょう(たとえば、絶対URLだけを含めるように頼んでみてください)。

# AIモデルとエージェントの仕組み {#how-ai-models-and-agents-work}

現代の[大規模言語モデル(LLM)](https://en.wikipedia.org/wiki/Large_language_model)やエージェントハーネスなどのインフラの内部動作を完全に説明することは、このコースの範囲を超えています。しかし、いくつかの重要なアイデアを高いレベルで理解しておくことは、この最先端技術を効果的に「使う」上でも、その限界を理解する上でも役立ちます。

LLMは、プロンプト文字列(入力)が与えられたときの補完文字列(出力)の確率分布をモデル化したものと見なせます。LLMの推論(たとえば対話型チャットアプリにクエリを入力したときに起こること)は、この確率分布から「サンプリング」を行います。LLMには固定の「コンテキストウィンドウ」があり、これは入力と出力の文字列の最大長です。

{% comment %}
> 数式で書くと、LLMはプロンプト $x$ を条件とする補完 $y$ の確率分布 $\pi_\theta$ をモデル化しており、私たちはこの分布からサンプリングします: $\hat{y} \sim \pi_\theta(\cdot \mid x)$。
{% endcomment %}

対話型チャットやコーディングエージェントといったAIツールは、このプリミティブの上に構築されています。マルチターンの対話のために、チャットアプリやエージェントはターンマーカーを使い、新しいユーザープロンプトが来るたびに会話履歴全体をプロンプト文字列として与え、ユーザープロンプトごとに1回LLMの推論を実行します。ツール呼び出しを行うエージェントでは、ハーネスがLLMの特定の出力をツール呼び出しの要求として解釈し、ツール呼び出しの結果をプロンプト文字列の一部としてモデルに返します(つまり、ツールの呼び出し/応答があるたびにLLMの推論が再び実行されます)。ツール呼び出しエージェントの中核となる概念は、[200行のコードで実装できます](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)。

## プライバシー

ほとんどのAIコーディングツールは、標準の設定では大量のデータをクラウドに送信します。ハーネスはローカルで動作しLLMの推論だけがクラウドで実行される場合もあれば、より多くの部分がクラウドで実行される場合もあります(その場合、たとえばサービス提供者は、リポジトリ全体のコピーと、あなたがAIツールと行うすべてのやり取りを事実上入手できることになります)。

オープンソースのAIコーディングツールや、かなり優秀なオープンソースのLLMも存在します(とはいえ、プロプライエタリなモデルほどの性能はありません)。しかし現時点では、ほとんどのユーザーにとって、最先端のオープンLLMをローカルで動かすことはハードウェアの制約から現実的ではないでしょう。

# ユースケース

コーディングエージェントは、非常に幅広いタスクで役立ちます。いくつか例を挙げます。

- **新機能の実装。** 上の例のように、コーディングエージェントに機能の実装を依頼できます。良い仕様を与えることは、現時点では科学というよりも技芸に近いものです。エージェントが意図どおりに動く(少なくとも正しい方向に進み始めて、そこから反復改善できる)程度に十分具体的な入力を与えたい一方で、詳細すぎて自分の作業量が増えてしまうのも避けたいところです。テスト駆動開発は特に効果的です。テストを書き(またはコーディングエージェントにテスト作成を手伝ってもらい)、そのテストが望むものを捉えているか自分で確認したうえで、コーディングエージェントに機能の実装を依頼します。モデルは絶えず進化しているので、モデルに何ができるかについての直観は常にアップデートしていく必要があります。
    > このTufteスタイルのサイドノートは、Claude Codeを使って[実装](https://github.com/missing-semester/missing-semester/pull/345)しました。
{%- comment %}
ここはデモ不要。講義の導入が、新機能を追加する小さなデモになっているため。
{% endcomment %}
- **エラーの修正。** コンパイラ、リンタ、型チェッカー、テストからエラーが出ている場合、エージェントに修正を依頼できます。たとえば「fix the issues with mypy」のようなプロンプトを与えます。コーディングモデルはフィードバックループに入れると特に効果を発揮するので、モデル自身が失敗しているチェックを直接実行できるように環境を整えましょう。そうすればモデルは自律的に反復できます。それが難しい場合は、手動でフィードバックを与えることもできます。
    > missing-semesterリポジトリのコミット[f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd)で、Claude Codeに「Review the agentic coding lecture for typos and grammatical issues」というプロンプトを与え、続けて見つかった問題の修正を依頼しました。その修正は[f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637)でコミットされています。
{%- comment %}
コーディングエージェントが https://github.com/anishathalye/dotbot/commit/cef40c902ef0f52f484153413142b5154bbc5e99 のバグを修正する様子をデモする。

バグを示すための失敗するテストを書き、それからエージェントに修正を依頼する。demo-bugfixブランチに準備済み。

失敗するテストは次のコマンドで実行できる:

    hatch test tests/test_cli.py::test_issue_357

コーディングエージェントには次のプロンプトを与える:

    There is a bug I wrote a failing test for, you can repro it with `hatch test tests/test_cli.py::test_issue_357`. Fix the bug.

変更をエージェントにコミットさせる。
{% endcomment %}
- **リファクタリング。** コーディングエージェントは、さまざまな形のリファクタリングに利用できます。メソッドのリネームのような単純なタスク(この種のリファクタリングは[コードインテリジェンス]({{ site.baseurl }}/2026/development-environment/#code-intelligence-and-language-servers)でも可能です)から、機能を別モジュールに切り出すような複雑なタスクまで対応できます。
    > エージェンティックコーディングを独立した講義に[分割](https://github.com/missing-semester/missing-semester/pull/344)する作業には、Claude Codeを使いました。
{%- comment %}
Missing Semesterでの利用例を見せ、エージェントがいくつかミスをしたことにも触れる。
{% endcomment %}
- **コードレビュー。** コーディングエージェントにコードのレビューを依頼できます。「review my latest changes that are not yet committed」のような簡単な指示を与えるだけで構いません。プルリクエストをレビューしたい場合、コーディングエージェントがWebフェッチに対応していたり、[GitHub CLI](https://cli.github.com/)のようなコマンドラインツールがインストールされていれば、「Review the pull request {link}」と頼むだけで、あとはエージェントが処理してくれるかもしれません。
{%- comment %}
Porcupineリポジトリで、エージェントに次のプロンプトを与える:

    Review this PR: https://github.com/anishathalye/porcupine/pull/39
{% endcomment %}
- **コード理解。** コードベースについてコーディングエージェントに質問できます。これは特にオンボーディングの際に役立ちます。
{%- comment %}
missing-semesterリポジトリで試すプロンプトの例:

    How do I run this site locally?

    How are the social preview cards implemented?
{% endcomment %}
- **シェルとして。** 特定のツールを使ってタスクを解決するようコーディングエージェントに依頼できます。つまり、自然言語でシェルコマンドを呼び出せるということです。たとえば「use the find command to find all files older than 30 days」や「use mogrify to resize all the jpgs to 50% of their original size」のように指示できます。
{%- comment %}
Dotbotリポジトリで、エージェントに次のプロンプトを与える:

    Use the ag command to find all Python renaming imports
{% endcomment %}
- **バイブコーディング。** エージェントは十分に強力なので、自分では一行もコードを書かずにアプリケーションを実装できることもあります。
    > 講師のひとりがバイブコーディングで作った実際のプロジェクトの[例がこちらです](https://github.com/cleanlab/office-presence-dashboard)。
{%- comment %}
missing-semesterリポジトリで、エージェントに次のプロンプトを与える:

    Make this site look retro.
{% endcomment %}

# 高度なエージェント

ここでは、コーディングエージェントのより高度な使用パターンと機能について簡単に紹介します。

- **再利用可能なプロンプト。** 再利用可能なプロンプトやテンプレートを作成できます。たとえば、特定のやり方でコードレビューを行うための詳細なプロンプトを書き、それを再利用可能なプロンプトとして保存できます。
    > エージェント関連のツールは急速に進化しています。ツールによっては、独立した機能としての再利用可能なプロンプトは非推奨になっています。たとえばCodexとClaude Codeでは、[スキル](https://code.claude.com/docs/en/skills)に[統合](https://developers.openai.com/codex/custom-prompts)されています。
- **並列エージェント。** コーディングエージェントは時間がかかることがあります。プロンプトを与えると、数十分にわたって問題に取り組むこともあります。複数のエージェントを同時に走らせることができ、同じタスクに取り組ませることも(LLMは確率的なので、同じことを複数回実行して最良の解を選ぶのが有効な場合があります)、別々のタスクに取り組ませることも(たとえば、重複しない2つの機能を同時に実装するなど)できます。異なるエージェントの変更が互いに干渉しないようにするには、[gitのworktree](https://git-scm.com/docs/git-worktree)が使えます。これは[バージョン管理]({{ site.baseurl }}/2026/version-control/)の講義で扱っています。
- **MCP。** MCPとは「Model Context Protocol」の略で、コーディングエージェントをツールと接続するためのオープンなプロトコルです。たとえば、この[Notion MCPサーバー](https://github.com/makenotion/notion-mcp-server)を使うと、エージェントがNotionのドキュメントを読み書きできるようになり、「{Notionドキュメント}にリンクされた仕様を読んで、実装計画をNotionの新しいページとして下書きし、プロトタイプを実装する」といったユースケースが可能になります。MCPを探すには、[Pulse](https://www.pulsemcp.com/servers)や[Glama](https://glama.ai/mcp/servers)のようなディレクトリが利用できます。
- **コンテキスト管理。** [上で述べたように](#how-ai-models-and-agents-work)、コーディングエージェントの基盤となるLLMには限られた「コンテキストウィンドウ」があります。コーディングエージェントを効果的に使うには、コンテキストをうまく活用する必要があります。エージェントが必要な情報にアクセスできるようにする一方で、不要なコンテキストは避けるべきです。コンテキストウィンドウのあふれを防ぐためだけでなく、モデルの性能低下を避けるためでもあります(コンテキストウィンドウがあふれなくても、コンテキストサイズが大きくなるにつれて性能は低下する傾向があります)。エージェントハーネスは自動的にコンテキストを供給し、ある程度は管理もしてくれますが、多くの制御はユーザーに委ねられています。
    - **コンテキストウィンドウのクリア。** 最も基本的な制御として、コーディングエージェントはコンテキストウィンドウのクリア(新しい会話の開始)に対応しています。無関係な問い合わせをするときはクリアすべきです。
    - **会話の巻き戻し。** コーディングエージェントの中には、会話履歴のステップを取り消せるものがあります。エージェントを別の方向に誘導するフォローアップメッセージを送るよりも、「取り消し」のほうが理にかなっている状況では、こちらのほうが効果的にコンテキストを管理できます。
{%- comment %}
簡単なデモをその場で作る。
{% endcomment %}
    - **コンパクション。** 長さに制限のない会話を可能にするため、コーディングエージェントはコンテキストの「コンパクション」に対応しています。会話履歴が長くなりすぎると、自動的にLLMを呼び出して会話の前半部分を要約し、会話履歴をその要約で置き換えます。ユーザーが任意のタイミングでコンパクションを実行できるエージェントもあります。
{%- comment %}
Claude Codeで `/compact` を見せ、要約の全文を表示する。
{% endcomment %}
    - **llms.txt。** `/llms.txt` ファイルは、LLMが推論時に利用するためのドキュメントの置き場所として提案されている[標準](https://llmstxt.org/)です。製品(例: [cursor.com/llms.txt](https://cursor.com/llms.txt))、ソフトウェアライブラリ(例: [ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt))、API(例: [apify.com/llms.txt](https://apify.com/llms.txt))が、開発に便利な `llms.txt` ファイルを提供していることがあります。こうしたドキュメントはトークンあたりの情報密度が高いため、コーディングエージェントにHTMLページを取得して読ませるよりもコンテキスト効率に優れています。外部ドキュメントは、利用しようとしている依存ライブラリについてコーディングエージェントが組み込みの知識を持っていない場合(たとえば、LLMの知識カットオフ以降に公開されたものである場合)に便利です。
{%- comment %}
空のリポジトリ(デスクトップなど他から独立した場所に作り、`git init` を実行しておく)で、次の2つを並べて比較する:

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers.

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers. See https://semlib.anish.io/llms.txt. Follow links to Markdown versions of any pages linked in llms.txt files.

エージェントがなぜデフォルトでこうしないのかは不明。この最後の一文は、おそらくCLAUDE.mdファイルに書いておくことになる。
{% endcomment %}
    - **AGENTS.md。** ほとんどのコーディングエージェントは、コーディングエージェント向けのREADMEとして[AGENTS.md](https://agents.md/)またはそれに類するもの(たとえばClaude Codeは `CLAUDE.md` を探します)に対応しています。エージェントの起動時に、`AGENTS.md` の内容全体がコンテキストにあらかじめ読み込まれます。これを使って、セッションをまたいで共通するアドバイスをエージェントに与えることができます(たとえば、コード変更後は必ず型チェッカーを実行するよう指示する、ユニットテストの実行方法を説明する、エージェントが参照できるサードパーティのドキュメントへのリンクを載せる、など)。このファイルを自動生成できるコーディングエージェントもあります(たとえばClaude Codeの `/init` コマンド)。実際の `AGENTS.md` の例は[こちら](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md)をご覧ください。
{%- comment %}
Dotbotの例。CLAUDE.mdが@DEVELOPMENT.mdを取り込んでおり、Pythonコードを変更したあとは必ず型チェッカーとコードフォーマッタを実行するよう指示している。

プロンプトの例(masterから作業する):

    Remove the "--version" command-line flag.

デモ用に、すぐに終わるものを選んでいる。
{% endcomment %}
    - **スキル。** `AGENTS.md` の内容は、常にその全体がエージェントのコンテキストウィンドウに読み込まれます。「スキル」は、コンテキストの肥大化を避けるために一段階の間接参照を加えたものです。エージェントにスキルの一覧とその説明を提供しておき、エージェントは必要に応じてスキルを「開く」(コンテキストウィンドウに読み込む)ことができます。
    - **サブエージェント。** コーディングエージェントの中には、タスク特化のワークフローのためのエージェントであるサブエージェントを定義できるものがあります。トップレベルのコーディングエージェントは、特定のタスクを完了させるためにサブエージェントを呼び出せます。これにより、トップレベルエージェントとサブエージェントの両方がコンテキストをより効果的に管理できます。トップレベルエージェントのコンテキストはサブエージェントが見るすべての情報で肥大化せずに済み、サブエージェントは自分のタスクに必要なコンテキストだけを受け取れます。一例として、Webリサーチをサブエージェントとして実装しているコーディングエージェントがあります。トップレベルエージェントがサブエージェントにクエリを投げると、サブエージェントはWeb検索を実行し、個々のWebページを取得・分析し、クエリへの回答をトップレベルエージェントに返します。こうすることで、トップレベルエージェントのコンテキストは取得したすべてのWebページの全文で肥大化せず、サブエージェントのコンテキストにはトップレベルエージェントの残りの会話履歴が含まれずに済みます。

プロンプトを書く必要がある高度な機能(スキルやサブエージェントなど)の多くでは、LLMを使って最初の一歩を踏み出すことができます。これを組み込みでサポートしているコーディングエージェントもあります。たとえばClaude Codeは、短いプロンプトからサブエージェントを生成できます(`/agents` を実行して新しいエージェントを作成します)。次のプロンプトでサブエージェントを作ってみてください。

```
A Python code checking agent that uses `mypy` and `ruff` to type-check, lint, and format *check* any files that have been modified from the last git commit.
```

その後、「use the code checker subagent」のようなメッセージで、トップレベルエージェントから明示的にサブエージェントを呼び出せます。適切な場面で、たとえばPythonファイルを変更したあとに、トップレベルエージェントが自動的にサブエージェントを呼び出すようにすることもできるかもしれません。

# 注意すべきこと

AIツールは間違いを犯すことがあります。AIツールはLLMの上に構築されており、LLMは単なる確率的な次トークン予測モデルにすぎません。人間と同じ意味で「知的」なわけではありません。AIの出力は、正しさとセキュリティ上のバグの観点からレビューしましょう。コードの検証は、自分でコードを書くよりも難しい場合があります。重要なコードは手で書くことを検討してください。AIは深みにはまり込んだり、あなたを言いくるめようとしたりすることがあります。デバッグの泥沼にはご注意を。AIを松葉杖のように使わないでください。過度な依存や理解の浅さには警戒が必要です。AIがまだこなせないプログラミングタスクは依然として山ほどあります。計算論的思考(computational thinking)には今も価値があります。

# おすすめのソフトウェア

多くのIDEやAIコーディング拡張機能にはコーディングエージェントが含まれています([開発環境の講義]({{ site.baseurl }}/2026/development-environment/)のおすすめを参照してください)。その他の人気のあるコーディングエージェントとしては、Anthropicの[Claude Code](https://www.claude.com/product/claude-code)、OpenAIの[Codex](https://openai.com/codex/)、そして[opencode](https://github.com/anomalyco/opencode)のようなオープンソースのエージェントがあります。

# 演習

1. 同じプログラミングタスクを4回行って、手作業でのコーディング、AIオートコンプリート、インラインチャット、エージェントのそれぞれの体験を比較してみましょう。題材として最適なのは、すでに取り組んでいるプロジェクトの小さめの機能です。他のアイデアを探しているなら、GitHub上のオープンソースプロジェクトの「good first issue」的なタスクや、[Advent of Code](https://adventofcode.com/)、[LeetCode](https://leetcode.com/)の問題に取り組むのもよいでしょう。
1. AIコーディングエージェントを使って、なじみのないコードベースを探索してみましょう。これは、実際に関心のあるプロジェクトをデバッグしたり新機能を追加したりしたいという文脈で行うのが一番です。思い当たるものがなければ、AIエージェントを使って[opencode](https://github.com/anomalyco/opencode)エージェントのセキュリティ関連機能がどのように動作しているかを理解してみてください。
1. 小さなアプリをゼロからバイブコーディングしてみましょう。一行たりとも自分の手でコードを書いてはいけません。
1. 使っているコーディングエージェントで、`AGENTS.md`(または `CLAUDE.md` のような、そのエージェントに対応するもの)、スキル(例: [Claude Codeのスキル](https://code.claude.com/docs/en/skills)や[Codexのスキル](https://developers.openai.com/codex/skills/))、サブエージェント(例: [Claude Codeのサブエージェント](https://code.claude.com/docs/en/sub-agents))を作成してテストしてみましょう。どのような場面でどれを使いたいかを考えてみてください。なお、使っているコーディングエージェントがこれらの機能の一部に対応していない場合もあります。その場合はスキップしてもよいですし、対応している別のコーディングエージェントを試してもよいでしょう。
1. [コード品質の講義]({{ site.baseurl }}/2026/code-quality/)にあるMarkdownの箇条書き記号の正規表現の演習と同じ目標を、コーディングエージェントを使って達成してみましょう。エージェントはファイルを直接編集してタスクを完了させるでしょうか?このようなタスクをエージェントがファイルの直接編集で完了させることの欠点や限界は何でしょうか?ファイルの直接編集ではない方法でタスクを完了させるには、エージェントにどうプロンプトを与えればよいかを考えてみてください。ヒント: [最初の講義]({{ site.baseurl }}/2026/course-shell/)で紹介したコマンドラインツールのひとつを使うようエージェントに依頼してみましょう。
1. ほとんどのコーディングエージェントは、いわゆる「yoloモード」に対応しています(たとえばClaude Codeでは `--dangerously-skip-permissions`)。このモードをそのまま使うのは安全ではありませんが、仮想マシンやコンテナのような隔離された環境でコーディングエージェントを動かし、自律的な動作を有効にするのであれば許容できるかもしれません。このセットアップをあなたのマシンで動かしてみましょう。[Claude Codeのdevcontainer](https://code.claude.com/docs/en/devcontainer)や[Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/agents/claude-code/)のようなドキュメントが役に立つかもしれません。セットアップの方法はひとつではありません。
