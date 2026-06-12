---
title: "20.4 MCPプロジェクト管理 — コラボレーションツール・ドキュメントをLLMにつなぐ"
part: 20
chapter: 4
status: v3
version: v3
written: 2026-05-24
author: 이민수
ip_check: done
---

# 20.4 MCPプロジェクト管理 — コラボレーションツール・ドキュメントをLLMにつなぐ

火曜日の出社直後、9時12分。私はコラボレーションツールのボードを開く前に、Claude Codeのウィンドウに1行打ち込みます。

```
今週の未完了P0タスクを、締め切りが近い順に表示して
```

3秒ほど間があって、答えが表示されました。コラボレーションツールを直接開いてはいません。ダッシュボードのタブを漁ることも、担当者にメッセンジャーを送ることもしていません。それなのに、締め切りを1日過ぎたタスクが1件、いちばん上に挙がっていました。私はそこで初めてコラボレーションツールを開き、そのカード1枚だけを確認しました。

この3秒がどう作られたのか — それがこの章のすべてです。重要なのは、ツールを替えたわけではないという点です。プロジェクトAのチームは今もコラボレーションツール（本プロジェクトではClickUp — タスクとスケジュールを管理するSaaSで、JIRA・Redmine・Linearも同じ位置づけ）を使っています。コラボレーションツールが何であれ、この章の流れはツール名を置き換えるだけでそのまま当てはまります。マスターデータも変わらずSVNにあり、決定カードも変わらずポータルにあります。変わったのはただ一つ、LLMがそれらのツールを**自分の手で開いて見られる**ようになったことです。その接続の標準がMCP（Model Context Protocol）です。

たとえるなら、受付デスクに新人をもう一人座らせる話ではありません。すでにある資料室の鍵を、LLMにも開けられるようにすることに近いです。資料室はそのままです。鍵をもう1本削り出しただけです。

---

## 20.4.1 MCPが正確に何をつなぐのか

MCPは、LLMが外部のツール・データにアクセスするための標準プロトコルです。「標準」という言葉が肝心です。コラボレーションツール用、ドキュメント用、git用と別々にアダプターを削り出すのではなく、JSON-RPCという一つの約束の上で各ツールが自分を「サーバー」として公開し、LLMは「クライアント」としてそのサーバーに話しかけます。

構造は3つのピースからなります。

<svg viewBox="0 0 760 220" xmlns="http://www.w3.org/2000/svg" font-family="sans-serif" font-size="13">
  <rect x="20" y="70" width="180" height="80" rx="8" fill="#e8f0fe" stroke="#3367d6" stroke-width="1.5"/>
  <text x="110" y="100" text-anchor="middle" font-weight="bold">MCPクライアント</text>
  <text x="110" y="122" text-anchor="middle">LLM / ユーザー</text>
  <text x="110" y="140" text-anchor="middle" fill="#555">(Claude Code)</text>

  <rect x="300" y="70" width="160" height="80" rx="8" fill="#fef7e0" stroke="#f9a825" stroke-width="1.5"/>
  <text x="380" y="105" text-anchor="middle" font-weight="bold">プロトコル</text>
  <text x="380" y="128" text-anchor="middle">JSON-RPC</text>

  <rect x="560" y="30" width="180" height="55" rx="8" fill="#e6f4ea" stroke="#1e8e3e" stroke-width="1.5"/>
  <text x="650" y="55" text-anchor="middle" font-weight="bold">MCPサーバー — コラボツール</text>
  <text x="650" y="74" text-anchor="middle" fill="#555">タスク検索・照会</text>

  <rect x="560" y="100" width="180" height="55" rx="8" fill="#e6f4ea" stroke="#1e8e3e" stroke-width="1.5"/>
  <text x="650" y="125" text-anchor="middle" font-weight="bold">MCPサーバー — ドキュメント</text>
  <text x="650" y="144" text-anchor="middle" fill="#555">決定カード・GDD照会</text>

  <line x1="200" y1="110" x2="300" y2="110" stroke="#666" stroke-width="1.5" marker-end="url(#a)"/>
  <line x1="460" y1="100" x2="558" y2="60" stroke="#666" stroke-width="1.5" marker-end="url(#a)"/>
  <line x1="460" y1="120" x2="558" y2="128" stroke="#666" stroke-width="1.5" marker-end="url(#a)"/>
  <defs><marker id="a" markerWidth="9" markerHeight="9" refX="7" refY="3" orient="auto"><path d="M0,0 L7,3 L0,6 z" fill="#666"/></marker></defs>
</svg>

サーバーは「自分にできること」のリストを公開します。コラボレーションツールのサーバーなら、`search_tasks`、`get_task`、`update_task`といった関数群です。クライアント（LLM）はユーザーの自然言語を受け取り、その中から合う関数を選んで呼び出し、返ってきたJSONをふたたび自然言語に展開して返します。私が「P0タスクを見せて」と言ったとき、LLMがやったことは`search_tasks(priority=P0, status=open)`を呼び出した、それだけです。

ここで一つはっきりさせておきましょう。本書はMCPサーバーを自作する方法は扱いません。ClickUp・JIRA・GitHubのように、公式・コミュニティのMCPサーバーがすでにあるツールがほとんどで、プランナーがやるべきことは「サーバーをインストールしてつなぐこと」であって、「プロトコルを実装すること」ではないからです。プロトコル自体はAnthropicが2024年11月に公開標準として発表し、仕様はmodelcontextprotocol.ioで公開されています。

---

## 20.4.2 ワークド・トランスクリプト — 「今週のP0を見せて」が実際にたどる道

言葉だけで「自動でできます」と言うと、魔法のように聞こえます。実際の1回を最後まで追ってみましょう。以下は、プロジェクトAでコラボレーションツールのMCPをつないだ直後、私が最初に回してみたセッションをほぼそのまま書き写したものです。LLMの生の出力を要約せず、ぎこちない部分まで残しています。

**① 私が打ったプロンプト（全文）**

```
ClickUpで今週締め切りの未完了P0タスクを、締め切りが近い順に整理して。
各項目に担当者と締め切りまでの残り日数を付けて、締め切りを過ぎたものはいちばん上に上げて。
```

**② Claudeの生の出力（そのまま）**

```
ClickUpワークスペースで、P0・未完了・今週締め切りの条件で照会しました。
3件ヒットしました。（締め切りを過ぎたものをいちばん上に上げています。）

1. TASK_017  グローバルクールダウン（GCD）コード変更
   担当: teammate_a / 締め切り: 昨日（−1日、超過）
2. TASK_023  voice_lint v2の適用
   担当: teammate_b / 締め切り: 明日（+1日）
3. TASK_031  戦闘ログスキーマ更新
   担当: teammate_c / 締め切り: 金曜日（+3日）

参考: TASK_009（バランステーブルのレビュー）もP0に見えますが、
statusが'in review'のため、「未完了」に入れるべきか判断に迷いました。
含めていません — 必要であれば基準を教えてください。
```

ここで、LLMが自ら立ち止まった箇所に注目してください。`TASK_009`は'in review'状態で、それが「未完了」かどうかをLLMは断定せず、人に問い返しました。MCPはデータを正確に取ってきますが、「in reviewを未完了に数えるか」はチームの定義に属する問題です。これをLLMが勝手に判断していたら、それこそ事故です。

**③ 私が行った検証・拒否**

私はコラボレーションツールを開き、TASK_017のカード1枚だけを確認しました。締め切りは本当に過ぎていました。ところがTASK_009は、うちのチームの基準では'in review'も未完了に含めます。LLMの分類がうちのルールと違っていました。そこで拒否し、基準を出し直しました。

**④ 再リクエスト**

```
うちのチームは'in review'も未完了として数える。その基準でもう一度整理して。
今後も'in review' = 未完了とみなして。
```

その後の出力では、TASK_009が2番の位置に入ってきました。最後の文（「今後も〜みなして」）はこのセッションにだけ適用されます。毎回同じルールを繰り返し伝えるのが嫌なら、この定義を`team_memory`の`shared`スロットにatomとして登録しておけば、次のセッションからLLMが自動的に適用します（§20.1・§20.2参照）。

この1往復が示すことは明確です。MCPは**情報を正確に取ってくるツール**であって、**判断を代行するツールではありません。**データは自動、定義は人が与える。その境界をあいまいにした瞬間、自動化は事故に変わります。

---

## 20.4.3 五つの活用パターン

コラボレーションツールの照会だけでは、「検索が少し楽になった」止まりです。MCPがコラボレーションのシステムになるのは、複数のツールを一つの流れに束ねたときです。プロジェクトAで実際に回している五つのパターンを、流れとして見ていきます。

<svg viewBox="0 0 800 400" xmlns="http://www.w3.org/2000/svg" font-family="sans-serif" font-size="11">
  <defs><marker id="mcpar" markerWidth="9" markerHeight="9" refX="7" refY="3" orient="auto"><path d="M0,0 L7,3 L0,6 z" fill="#888"/></marker></defs>
  <text x="400" y="20" text-anchor="middle" font-size="14" font-weight="bold">MCP五つの活用パターン — 各パターンは独立した一つの流れ</text>
  <text x="400" y="38" text-anchor="middle" font-size="10" fill="#666">左の色ブロックがパターン、右の白いボックスがそのパターンがたどるステップ（左→右）</text>

  <!-- パターン1 自動レポート -->
  <rect x="8" y="52" width="118" height="42" rx="6" fill="#e3f2fd" stroke="#1976d2" stroke-width="1.8"/>
  <text x="67" y="70" text-anchor="middle" font-weight="bold" fill="#1565c0">パターン1</text>
  <text x="67" y="85" text-anchor="middle" font-size="10" fill="#1565c0">自動レポート</text>
  <rect x="138" y="53" width="118" height="40" rx="4" fill="#fff" stroke="#1976d2"/>
  <text x="197" y="69" text-anchor="middle" font-size="10">毎日09時</text>
  <text x="197" y="83" text-anchor="middle" font-size="10">スケジューラートリガー</text>
  <line x1="256" y1="73" x2="266" y2="73" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="268" y="53" width="118" height="40" rx="4" fill="#fff" stroke="#1976d2"/>
  <text x="327" y="69" text-anchor="middle" font-size="10">コラボツール・git・</text>
  <text x="327" y="83" text-anchor="middle" font-size="10">ダッシュボード照会</text>
  <line x1="386" y1="73" x2="396" y2="73" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="398" y="53" width="118" height="40" rx="4" fill="#fff" stroke="#1976d2"/>
  <text x="457" y="76" text-anchor="middle" font-size="10">LLMレポート合成</text>
  <line x1="516" y1="73" x2="526" y2="73" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="528" y="53" width="118" height="40" rx="4" fill="#fff" stroke="#1976d2"/>
  <text x="587" y="76" text-anchor="middle" font-size="10">ポータル/メッセンジャー送信</text>

  <!-- パターン2 決定→タスク -->
  <rect x="8" y="120" width="118" height="42" rx="6" fill="#e8f5e9" stroke="#388e3c" stroke-width="1.8"/>
  <text x="67" y="138" text-anchor="middle" font-weight="bold" fill="#2e7d32">パターン2</text>
  <text x="67" y="153" text-anchor="middle" font-size="10" fill="#2e7d32">決定 → タスク</text>
  <rect x="138" y="121" width="118" height="40" rx="4" fill="#fff" stroke="#388e3c"/>
  <text x="197" y="137" text-anchor="middle" font-size="10">決定カード登録</text>
  <text x="197" y="151" text-anchor="middle" font-size="9">proposal P####</text>
  <line x1="256" y1="141" x2="266" y2="141" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="268" y="121" width="118" height="40" rx="4" fill="#fff" stroke="#388e3c"/>
  <text x="327" y="137" text-anchor="middle" font-size="10">コラボツールに</text>
  <text x="327" y="151" text-anchor="middle" font-size="10">タスク生成</text>
  <line x1="386" y1="141" x2="396" y2="141" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="398" y="121" width="118" height="40" rx="4" fill="#fff" stroke="#388e3c"/>
  <text x="457" y="137" text-anchor="middle" font-size="10">担当者・締め切り</text>
  <text x="457" y="151" text-anchor="middle" font-size="10">自動設定</text>

  <!-- パターン3 タスク→カード -->
  <rect x="8" y="188" width="118" height="42" rx="6" fill="#fff8e1" stroke="#f9a825" stroke-width="1.8"/>
  <text x="67" y="206" text-anchor="middle" font-weight="bold" fill="#e65100">パターン3</text>
  <text x="67" y="221" text-anchor="middle" font-size="9" fill="#e65100">タスク→カード（逆方向）</text>
  <rect x="138" y="189" width="118" height="40" rx="4" fill="#fff" stroke="#f9a825"/>
  <text x="197" y="205" text-anchor="middle" font-size="10">コラボツール</text>
  <text x="197" y="219" text-anchor="middle" font-size="10">タスク完了</text>
  <line x1="256" y1="209" x2="266" y2="209" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="268" y="189" width="118" height="40" rx="4" fill="#fff" stroke="#f9a825"/>
  <text x="327" y="205" text-anchor="middle" font-size="10">決定カード</text>
  <text x="327" y="219" text-anchor="middle" font-size="9">execution_log更新</text>

  <!-- パターン4 進捗率分析 -->
  <rect x="8" y="256" width="118" height="42" rx="6" fill="#f3e5f5" stroke="#7b1fa2" stroke-width="1.8"/>
  <text x="67" y="274" text-anchor="middle" font-weight="bold" fill="#6a1b9a">パターン4</text>
  <text x="67" y="289" text-anchor="middle" font-size="10" fill="#6a1b9a">進捗率分析</text>
  <rect x="138" y="257" width="118" height="40" rx="4" fill="#fff" stroke="#7b1fa2"/>
  <text x="197" y="273" text-anchor="middle" font-size="10">四半期全体の</text>
  <text x="197" y="287" text-anchor="middle" font-size="10">タスク照会</text>
  <line x1="256" y1="277" x2="266" y2="277" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="268" y="257" width="118" height="40" rx="4" fill="#fff" stroke="#7b1fa2"/>
  <text x="327" y="273" text-anchor="middle" font-size="10">LLMによる遅延</text>
  <text x="327" y="287" text-anchor="middle" font-size="10">パターン分析</text>
  <line x1="386" y1="277" x2="396" y2="277" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="398" y="257" width="118" height="40" rx="4" fill="#fff" stroke="#7b1fa2"/>
  <text x="457" y="280" text-anchor="middle" font-size="10">四半期振り返りへ入力</text>

  <!-- パターン5 1:1事前資料 -->
  <rect x="8" y="324" width="118" height="42" rx="6" fill="#fff3e0" stroke="#e64a19" stroke-width="1.8"/>
  <text x="67" y="342" text-anchor="middle" font-weight="bold" fill="#d84315">パターン5</text>
  <text x="67" y="357" text-anchor="middle" font-size="10" fill="#d84315">1:1事前資料</text>
  <rect x="138" y="325" width="118" height="40" rx="4" fill="#fff" stroke="#e64a19"/>
  <text x="197" y="348" text-anchor="middle" font-size="10">1:1ミーティング5分前</text>
  <line x1="256" y1="345" x2="266" y2="345" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="268" y="325" width="118" height="40" rx="4" fill="#fff" stroke="#e64a19"/>
  <text x="327" y="341" text-anchor="middle" font-size="9">メンバーのタスク +</text>
  <text x="327" y="355" text-anchor="middle" font-size="9">team_memory・活動</text>
  <line x1="386" y1="345" x2="396" y2="345" stroke="#888" stroke-width="1.3" marker-end="url(#mcpar)"/>
  <rect x="398" y="325" width="118" height="40" rx="4" fill="#fff" stroke="#e64a19"/>
  <text x="457" y="348" text-anchor="middle" font-size="10">事前要約を自動生成</text>
</svg>

**パターン1 — 自動レポート。** 毎朝9時、スケジューラーがトリガーを投げると、LLMがコラボレーションツールのタスク状態・gitコミット・ダッシュボード指標を一括で照会し、日次レポートを書きます。送信先はポータルまたはチームの社内メッセンジャーです。ここでの「ポータル」は、§20.3で扱った社内ポータルWebのことです。`server.py`（FastAPI）が常時稼働しており、Claudeが作成した`View_*.html`がその上で動くので、レポートもポータルの1ページとして自然に載ります。

**パターン2 — 決定 → タスク自動生成。** 決定カード（`proposal P####`）を登録すると、カードのimplementation・verification項目がそのままコラボレーションツールのタスクになります。担当者と締め切りまで自動で入力されます。会議で「では、こうすることに」と決めたことが、人手を介さずボードのカードとして落ちてきます。

**パターン3 — タスク → 決定カードの逆参照。** パターン2の逆方向です。コラボレーションツールのタスクが完了すると、MCPが元の決定カードの`execution_log`を更新します。「この決定は実際に実行されたのか」が自動で追跡されます。決定と実行が双方向に結ばれます（図の点線）。

**パターン4 — 進捗率分析。** 四半期末、その四半期のすべてのタスクを一度に引いてきて、遅延パターンを分析します。「どの種類のタスクが繰り返し先送りされるのか」が振り返りの入力になります。

**パターン5 — 1:1事前資料。** 1:1ミーティングの5分前、該当メンバーのコラボレーションツールのタスク・`team_memory`スロット・最近の活動を合成し、事前要約を作ります。1:1が「前回は何をしていましたっけ?」で5分を浪費する代わりに、本題から始まります。

五つのパターンの共通点は一つです。**人の手に残る繰り返し作業を減らせる場所でだけ、価値が回収されます。**かっこよく見せるために付けるパターンは、運用の負担を増やすだけです。

---

## 20.4.4 段階的導入 — 一度に全部つながない

MCPサーバー5個を初日に全部つなぐのは、いちばんよくある失敗です。順序があります。

| 段階 | 何を | 期間の感覚 |
|---|---|---|
| 1 | MCPサーバーを1個インストール（ClickUpまたはJIRA） | 1〜2日 |
| 2 | パターンを1個試行（自動レポート） | 約1週間 |
| 3 | 5パターンの運用 | 1〜2か月 |
| 4 | 自前のMCPサーバー開発（特殊ツールが必要な場合） | 1〜3か月 |

上記の期間はプロジェクトAの中規模（10〜50人）チームを基準にした著者の推定（未検証）であり、チームの規模・ツールへの習熟度によって変わります。確かなのは、**ほとんどのチームは1〜3段階で十分**だということです。4段階（自前サーバー開発）は、市中にMCPサーバーがない特殊な社内ツールをどうしてもつなぐ必要があるときだけ進みます。そこまで行かなくても、効果の大半は回収できます。

JIRAを別途取り上げる理由があります。ClickUpがチーム内部のボードだとすれば、JIRAはパブリッシャー・外注のような**外部組織と共有する**ツールであることが多いからです。JIRA MCPをつなげば、外注ボードの進捗率を毎週の会議前に自動で引いてきて、遅延が疑われるタスクをあらかじめ絞り込んでおけます。会議が「現況共有」ではなく「決定」から始まるようになります。ただし、外部と共有するツールであるほど、次節の権限・流出の落とし穴がより重くのしかかります。

---

## 20.4.5 四つの落とし穴 — MCPが諸刃の剣である理由

MCPは、LLMにツールを握らせる行為です。手に握ったものが刃物なら、切られることもあります。

**落とし穴1 — 権限事故。** LLMがwrite権限まで持っていると、意図しない変更が起きます。「整理して」の一言でタスクの状態を一括で書き換えてしまう、といった具合です。処方は明確です。**MCPサーバーはread-onlyで始めます。**writeはパターン2・3のように本当に必要な場所にだけ、それも実行前の確認ゲートを置いた上で開きます。先のトランスクリプトでLLMが'in review'の分類を人に問い返したのも同じ精神です — あいまいなら止まる、です。

**落とし穴2 — データ流出。** MCPで引いてきた社内データが外部のLLM APIに送信されます。バランス数値、未公開コンテンツ、売上指標がそのまま流れていきかねません。処方は、機密データに限って自前ホスティングのLLMを使うか、MCPサーバー側でplaceholderに置換してから外に出すことです（本書全体のIP保護原則と同じです）。

**落とし穴3 — 依存関係の爆増。** MCPサーバー5個をcriticalな経路にぶら下げておくと、1か所が落ちたときに朝のレポート全体が止まります。処方は、核心の1〜2個だけをcriticalに置き、残りは補助として分離することです。補助サーバーが落ちればその項目だけが空欄になり、レポート自体は出ます。

**落とし穴4 — APIコストの爆増。** MCP呼び出し1回が、LLMトークンと外部API呼び出しを同時に燃やします。自動レポートを5分ごとに回すと、コストが静かに膨らみます。処方は、呼び出し頻度にcapをかけ、頻繁に変わらない照会結果はキャッシュすることです。

四つの落とし穴を1行にまとめると、MCPの安全な置き場所は、**「read-onlyで始め、核心だけをcriticalに置き、機密データはフィルタリングし、呼び出しに上限をかける」**です。

---

## 20.4.6 効果 — 時間はどこで回収されるのか

プロジェクトAでMCP運用の前後に体感した変化です。以下の時間数値は著者の経験的推定（未検証）であり、絶対値ではなく**方向と比率**として読んでください。

| 項目 | MCPなし | MCP運用 | 方向 |
|---|---|---|---|
| 日次レポートの情報集約 | 手動30〜60分 | 自動 約5分 | 大幅短縮 |
| ツール間の情報同期 | 人の手作業 | 自動 | 手作業の除去 |
| 1:1の事前準備 | 10〜15分 | 自動要約 約3分 | 短縮 |
| 外注進捗率の把握 | 会議でのみ | リアルタイム照会 | 常時化 |
| 決定 ↔ タスクの連結 | 手作業 | 双方向自動 | 漏れ防止 |

もっとも大きな回収は「情報を集める時間」から生まれます。決定・判断は依然として人の仕事です。MCPが減らしてくれるのはその手前、散らばったツールを漁って1か所に集める単純労働です。この章の冒頭の3秒が、まさにその場所です。

---

## 20.4.7 第20部を閉じるにあたって

第20部では、チームコラボレーションのシステムを四つの層で積み上げました。

| 章 | 要点 |
|---|---|
| 20.1 | atom運用 — カテゴリー分類、四半期整理 |
| 20.2 | チームメンバーのメモリー — `team_memory`の5人スロット（leeminsoo・チームメンバーA/b/c・shared）、共有 vs 個人の分離 |
| 20.3 | ポータルWeb — `server.py`（FastAPI）・`build_index.py`・nginx・nssmで常時稼働、`View_*.html`が動作 |
| 20.4 | MCP — 5パターン・段階的導入・権限優先 |

四つの章は別々に動くツールではなく、一つの体です。ポータル（20.3）はレポートが載る場所であり、atom・メモリー（20.1・20.2）はLLMが「in review = 未完了」のようなチームの定義を記憶する場所であり、MCP（20.4）はそのすべてをコラボレーションツール・ドキュメントとつなぐ配線です。どれか一つだけを切り離せば、残りの価値も半分に減ります。

次の部ではガバナンスと運用を扱います。ツールをここまでつないだときについてくる安全性・コスト・著作権・倫理の問題を、この章の落とし穴をチーム単位のルールに引き上げる形で整理します。

---

### 本章のポイント

- MCPは、ツールを替えずにLLMへ既存ツールの鍵を削り出して渡す標準です。
- データはMCPが取ってきて、定義・判断は人が与えます — その境界が安全線です。
- read-onlyで始めて核心だけをcriticalに置けば、自動化は事故に転じません。

---

## 20.4.8 やってみよう — ClickUp MCP初接続

**setup**
1. ClickUp MCPサーバー（公式またはコミュニティ）をインストールし、ClickUp APIトークンを発行しましょう。
2. Claude Codeの設定にMCPサーバーを登録します。ただし**read-onlyスコープのみで**始めます。
3. ワークスペースIDを確認し、照会範囲を自分のボードに制限します。

**prompt**
```
ClickUpで今週締め切りの未完了P0タスクを、締め切りが近い順に整理して。
担当者と締め切りまでの残り日数を付けて、締め切りを過ぎたものはいちばん上に。
```

**verify**
1. 出力されたタスクのうち1件をClickUpで直接開き、締め切り・担当者が合っているか突き合わせましょう。
2. LLMがあいまいな項目を自ら問い返したかを確認します — 問い返さずに断定していたら、分類基準を明示してやり直させます。
3. よく使う定義（'in review=未完了'など）は`team_memory`の`shared`にatomとして登録し、次のセッションから自動適用されるようにします。

### 一人ミニ版

チームもコラボレーションツールもない一人開発者なら、MCPの最初の対象は**GitHubとローカルドキュメント**です。GitHub MCPをread-onlyでつなぎ、「今週クローズされていないIssueを古い順に」のような照会から始めて、決定カードの代わりにローカルの`decisions/`フォルダーのMarkdownをドキュメントMCPで照会しましょう。自動レポート（パターン1）はそのまま適用できます — 送信先をチームのメッセンジャーから自分のメモファイルに変えるだけです。権限・コストの落とし穴は一人でも同じようにかかるので、read-onlyでの始動と呼び出しcapは最初から守るのがよいです。
