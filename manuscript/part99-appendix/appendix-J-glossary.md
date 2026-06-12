---
title: "付録J. 略語・用語集"
appendix: J
status: v3
written: 2026-06-06
author: 이민수
version: v3
---

# 付録J. 略語・用語集

本文に登場する略語と、本書固有の用語を一か所にまとめました。本文では各略語が初めて登場する箇所で一度だけ展開して書いていますが、順番どおりに読まない場合や、途中で忘れてしまった場合は、ここですぐに調べられます。一つの略語が文脈によって異なる意味を持つ場合は、両方を記載しました。

この用語集は次の順序でまとめています。チーム規模の等級 → ゲーム企画ドキュメント → ゲームドメイン → データ・運営 → AI・ツール → UI・アクセシビリティ標準 → ファイル・フォーマット。探している略語の性質を先に思い浮かべると、どのグループにあるかが絞り込めます。たとえば`DPS`・`TTK`は「ゲームドメイン」、`KPI`・`DAU`は「データ・運営」、`atom`・`JIT`は「AI・ツール」のグループです。

表記ルールは三つです。①一般的な略語は、正式名称と日本語の意味を併記しました。②`atom`・`Wrapper`のように本書だけで使う固有の用語は、正式名称の欄に「（本書固有の用語）」と表示しました。③`PK`（戦争文脈のPlayer Kill ↔ データ文脈のPrimary Key）のように一つの略語が二つの意味を持つ場合、本文では初出時にどちらの意味かを併記し、この表には両方の意味を載せました。

## チーム規模の等級

本書では、チームの人数を特定の数字に固定せず、次の三つの等級で表記します。同じ手法でも、チーム規模によって導入の深さが変わるためです。

| 等級 | 人数の目安 | 説明 |
|---|---|---|
| 小規模 | 〜10人 | 1人・趣味の開発者から一桁規模のチームまで。たいてい1〜2段階の導入で十分 |
| 中規模 | 10〜50人 | 本書の運用事例が生まれた著者のチームが属する区間。標準化・整合性自動化の累積効果がはっきりしてくる規模 |
| 大規模 | 100+ | 複数のパート・複数のチーム。専用インフラと専任の運用が正当化される規模 |

本文で「中規模（10〜50人）チーム」のように等級と人数の範囲を併記している箇所は、この表を基準とします。1人・個人開発のように人数そのものが意味を持つ箇所では、等級の代わりに正確な数をそのまま使います。

## ゲーム企画ドキュメント

| 略語 | 正式名称 | 意味 |
|---|---|---|
| GDD | Game Design Document | ゲームデザインドキュメント。システム・数値・動作を確定した詳細仕様書 |
| CDD | Concept Design Document | コンセプトデザインドキュメント。GDD以前の段階の初期企画書（方向性・コンセプト） |
| TF | TaskForce | 短期目標のために一時的に編成した専任チーム（例：戦闘TF） |
| DD | Design Director | デザインディレクター。ゲームの設計方針を統括するリード役 |
| RnD | Research and Development | 研究・開発。プロトタイプ・新手法を探索する段階・組織（例：プロシージャル生成RnD） |

## ゲームドメイン

| 略語 | 正式名称 | 意味 |
|---|---|---|
| NPC | Non-Player Character | プレイヤーが操作しないキャラクター |
| HUD | Heads-Up Display | ゲーム画面に重ねて表示する状態情報（体力・ミニマップなど） |
| DPS | Damage Per Second | 1秒あたりのダメージ量 |
| GCD | Global Cooldown | グローバルクールダウン。スキルを一つ使うと、すべてのスキルが短時間まとめてロックされる共用の待機時間 |
| TTK | Time To Kill | 対象を倒すのにかかる時間 |
| PK | Player Kill | （戦争・PvP文脈）プレイヤー間の戦闘・キル |
| BT | BehaviorTree | ビヘイビアツリー。NPCのAIの行動分岐をツリーで定義した構造 |
| FSM | Finite State Machine | 有限ステートマシン。状態と遷移で行動を定義するモデル |
| PCG | Procedural Content Generation | プロシージャルコンテンツ生成。ルール・アルゴリズムでコンテンツを自動生成 |
| VFX | Visual Effects | ビジュアルエフェクト |
| SFX | Sound Effects | 効果音 |
| VA | Voice Actor | 声優 |
| RPG / MMORPG | (Massively Multiplayer Online) Role-Playing Game | ロールプレイングゲーム / 大規模多人数同時参加型オンラインRPG |
| P2W / P2E | Pay To Win / Play To Earn | 課金で強くなる構造 / プレイで収益を得る構造 |
| RMT | Real Money Trading | ゲーム内財貨の現金取引 |

## データ・運営

| 略語 | 正式名称 | 意味 |
|---|---|---|
| KPI | Key Performance Indicator | 重要業績評価指標 |
| DAU | Daily Active Users | 1日あたりのアクティブユーザー数 |
| FK | Foreign Key | 外部キー。他のシートの主キーを参照するカラム |
| PK | Primary Key | （データ文脈）主キー。行を一意に識別するカラム |
| ROI | Return on Investment | 投資対効果（回収） |
| MECE | Mutually Exclusive, Collectively Exhaustive | 相互排他・全体網羅。漏れなく、ダブりなく分ける分類原則 |
| STT | Speech-to-Text | 音声をテキストに変換 |
| VBA | Visual Basic for Applications | Excelに組み込まれたマクロ言語 |
| SVN | Subversion | ファイルのバージョン管理システム |
| telemetry | （計測データ） | ゲームのビルド・実行から自動収集するプレイログ・指標（入力・戦闘・離脱など）。読み方は「テレメトリー」 |

## AI・ツール

| 略語 | 正式名称 | 意味 |
|---|---|---|
| AI | Artificial Intelligence | 人工知能 |
| LLM | Large Language Model | 大規模言語モデル（ChatGPT・Claudeなどの基盤） |
| JIT | Just-In-Time | 必要な瞬間にだけ差し込む方式（本書では入力に合わせた記憶の自動注入） |
| MCP | Model Context Protocol | AIツールを外部サービスと連携させる標準 |
| API | Application Programming Interface | プログラム間の呼び出し規約 |
| UE | Unreal Engine | アンリアルエンジン |
| atom | （本書固有の用語） | 1決定 = 1ファイルとして固定化した、決定・ルールのカード |
| Wrapper / Cascade / Junction | （本書固有の用語） | よく使うツールのエントリーポイント / 複数のチェックを一度にまとめたツール / 本体につなぐシンボリックリンク |
| rg | ripgrep | 高速なテキスト検索コマンド（grepを代替するCLIツール）。コード・ドキュメントの全件検索に使用 |
| ClickUp | （タスク・イシュートラッカー） | 作業・スケジュールを管理するクラウド型コラボレーションツール。JIRA・Redmine・Linearも同じカテゴリー。MCPで連携してAIが照会・更新 |

## UI・アクセシビリティ標準

| 略語 | 正式名称 | 意味 |
|---|---|---|
| UI / UX | User Interface / User Experience | ユーザーインターフェース / ユーザー体験 |
| WCAG | Web Content Accessibility Guidelines | ウェブアクセシビリティガイドライン（コントラスト比・タッチターゲットサイズなどの合格ライン） |
| HIG | (Apple) Human Interface Guidelines | Appleのインターフェースガイドライン |
| SC | Success Criterion | WCAGの個別の達成基準番号（例：SC 1.4.3） |
| pt / dp / px | point / density-independent pixel / pixel | 画面サイズの単位 |

## ファイル・フォーマット

| 略語 | 正式名称 | 意味 |
|---|---|---|
| YAML | YAML Ain't Markup Language | 人が読みやすい設定・データ記述形式 |
| JSON | JavaScript Object Notation | データ交換用の記述形式 |
| HTML / SVG | HyperText Markup Language / Scalable Vector Graphics | ウェブ文書 / ベクターグラフィックス形式 |
| GLB | GL Transmission Format (Binary) | 3Dモデルのバイナリーファイル形式 |

---

*PKのように文脈によって意味が分かれる略語は、本文の初出時にどちらの意味かを併記しました。迷ったときはこの表に戻ってきてください。*
