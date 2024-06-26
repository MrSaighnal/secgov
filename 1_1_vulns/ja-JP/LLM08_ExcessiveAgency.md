## LLM08: 過剰な代理行為

### 解説

LLMベースのシステムでは、開発者により、他のシステムとのインターフェイスや、プロンプトに応答してアクションを実行する能力など、ある程度の処理代行能力が与えられていることが良くあります。どの機能を呼び出すかの判断は、入力プロンプトまたは LLM の出力に基づいて動的に決定する LLM 「エージェント」に委譲されることがあります。

過剰な代理行為 は、LLM からの予期しない/あいまいな出力に応答して有害なアクションを実行できる脆弱性 のことです（LLM が誤動作する原因が何であるかに関係ありません。ハルシネーション(幻覚) / コンファビュレーション(作話)、直接的/間接的なプロンプトの 注入、悪意のあるプラグイン、悪意はないが不十分な設計のプロンプト、または単なるパフォーマンスの低いモデル）。過剰な代理行為 の根本的な原因は、通常、「過剰なファンクション」、「過剰なパーミッション」、「過剰な自律性」の いずれかまたはその組みあわせです。

過剰な代理行為 は、機密性、完全性、可用性の各領域にわたって広範な影響をもたらす可能性があります。また、LLM ベースのアプリがどのシステムと相互作用できるかにも依存します。

### リスクの事例

1. 過剰なファンクション： あるLLMエージェントでは、システムの運用に必要のない機能を含むプラグインにアクセスできました。例えば、開発者はLLMエージェントにリポジトリからドキュメントを読み込む機能を与える必要がありますが、彼らが使用するサードパーティのプラグインには、ドキュメントを修正・削除する機能も含まれていました。あるいは、あるプラグインが開発段階で試用され、よりよい代替品に取って代わられたとしても、元のプラグインはLLMエージェントが利用できるままになっていることもあります。
2. 過剰なファンクション： あるLLMプラグインは、アプリケーションの操作に必要なコマンド以外の入力命令を適切にフィルタリングすることができませんでした。例えば、ある特定のシェルコマンドを実行するプラグインは、それ以外のシェルコマンドの実行を適切に防ぐことができませんでした。
3. 過剰なパーミッション： あるLLMプラグインでは、アプリケーションの意図された操作に必要でないパーミッションを持っていました。たとえば、データの読み取りを目的としたプラグインが、SELECT パーミッションだけでなく、UPDATE、INSERT、DELETE パーミッションも持つ ID を使用してデータベースサーバーに接続していました。
4. 過剰なパーミッション：  あるLLM プラグインは、ユーザーの代わりに操作を実行するように設計されていますが、一般的な高特権 ID を使用してダウンストリームシステムにアクセスしていました。例えば、現在のユーザーのドキュメントストアを読むプラグインが、すべてのユーザーのファイルにアクセスできる特権アカウントでドキュメントリポジトリに接続していました。
5. 過剰な自律性： LLM ベースのアプリケーションやプラグインが、影響度の高いアクションを個別に検証・承認できないことがありました。例えば、ユーザーの文書を削除できるようにするプラグインは、ユーザーからの確認なしに削除を実行していました。

### 予防策と被害の軽減策

「過剰な代理行為」を防止するには、次のような対応が必要です：

1. LLMエージェントが呼び出せるプラグイン/ツールは、必要最小限の機能に限定します。例えば、LLMベースのシステムがURLの内容を取得する機能を必要としない場合、そのようなプラグイン機能をLLMエージェントに提供すべきではありません。
2. LLMプラグイン/ツールに実装する機能は、必要最小限のものに限定します。例えば、メールを要約するためにユーザのメールボックスにアクセスするプラグインは、メールを読む機能だけが必要かもしれません。その場合、プラグインはメッセージの削除や送信といった他の機能を含むべきではありません。
3. 可能な限り、オープンエンドな機能（シェルコマンドの実行、URLの取得など）は避け、より詳細な機能を持つプラグイン/ツールを使うべきです。例えば、LLMベースのアプリは、ファイルに出力を書き出す必要があるかもしれません。これをシェル関数を実行するプラグインを使って実装すると、望ましくないアクションの範囲が非常に大きくなります（他のシェルコマンドも実行される可能性があります）。より安全な代替案は、その特定の機能だけをサポートするファイル書き込みプラグインを構築することです。
4. 望ましくない動作の範囲を制限するために、LLMプラグイン/ツールが他のシステムに与えるパーミッションを必要最小限に制限します。例えば、顧客に購入を勧めるために商品データベースを使用する LLM エージェントは、「商品」テーブルへの読み取りアクセスだけが必要かもしれません。その場合、LLMエージェントは他のテーブルへのアクセスや、レコードの挿入、更新、削除を行えるべきではありません。これには、LLM プラグインがデータベースへの接続に使用する ID に対して、適切なデータベース権限を適用する必要があります。
5. ユーザーのアクセス権限とセキュリティスコープを追跡し、あるユーザーのために実行されたアクションが、その特定のユーザーのコンテキストで、必要最小限の権限でダウンストリームシステム上で実行されるようにします。例えば、LLMプラグインがユーザーのコード・レポジトリを読み込む場合、ユーザーには必要最小限のスコープについてOAuthによる認証を要求します。
6. 人の目でのチェック(human-in-the-loop control)を活用して、すべてのアクションを実行する前に人間が承認することを要求します。これは、（LLMアプリケーションの範囲外の）ダウンストリームシステムに実装しても、LLMプラグイン/ツール自体に実装してもよいです。例えば、ユーザーに代わってソーシャルメディアコンテンツを作成し投稿するLLMベースのアプリは、「投稿」操作を実装するプラグイン/ツール/API内に、ユーザー承認ルーチンを含めるべきです。
7. アクションが許可されるかどうかの決定をLLMに依存するのではなく、ダウンストリームシステムで認可を実装します。ツール/プラグインを実装するときは、プラグイン/ツールを介して下流システムになされるすべての要求がセキュリティポリシーに照らして検証されるように、完全な仲介の原則を実施します。

以下のオプションは、「過剰な代理行為」 を防止するものではありませんが、発生する損害のレベ ルを制限することができます：

1. LLM プラグイン/ツールおよびダウンストリーム・システムのアクティビティをログに記録して監視し、好ま しくないアクションが行われている場所を特定し、それに応じて対応します。
2. 一定の時間内に実行される望ましくないアクションの数を減らし、重大な損害が発生する前に、監視により望ましくないアクションを発見する可能性を高めるためにレート制限を実装します。

### 攻撃相手の手法

LLMベースのパーソナル・アシスタント・アプリは、受信メールの内容を要約するために、プラグインを介して個人のメールボックスへのアクセスを許可します。この機能を実現するために、メールプラグインではメッセージを読む機能が必要ですが、システム開発者が選択したプラグインにはメッセージを送信する機能も含まれていました。LLMは間接的なプロンプトインジェクション攻撃に対しても脆弱であったため、悪意を持って作成された受信メールがLLMを騙してメールプラグインに「send message」関数を呼び出させ、ユーザーのメールボックスからスパムを送信させることができました。

これを避けるには以下を行います。 
  (a)メールを読む機能だけを提供するプラグインを使うことで過剰な機能を排除する、
  (b)読み取り専用スコープを持つOAuthセッションを介してユーザーのメールサービスを認証することで過剰なパーミッションを排除する、および/または、
  (c)LLMプラグインによってドラフトされたすべてのメールをユーザーが手動で確認し、「送信」を押すことを要求することで過剰な自律性を排除する。あるいは、メール送信インターフェースにレート制限を実装することで、被害を軽減することもできます。

### 参考文献

1. [Embrace the Red: Confused Deputy Problem](https://embracethered.com/blog/posts/2023/chatgpt-cross-plugin-request-forgery-and-prompt-injection./): **Embrace The Red**
2. [NeMo-Guardrails: Interface guidelines](https://github.com/NVIDIA/NeMo-Guardrails/blob/main/docs/security/guidelines.md): **NVIDIA Github**
3. [LangChain: Human-approval for tools](https://python.langchain.com/docs/modules/agents/tools/how_to/human_approval): **Langchain Documentation**
4. [Simon Willison: Dual LLM Pattern](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/): **Simon Willison**
