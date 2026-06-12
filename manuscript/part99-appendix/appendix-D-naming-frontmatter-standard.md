# 付録D. R&D文書の命名・Frontmatter標準

> 会社のプロジェクトAで使っているR&D文書の命名・frontmatter標準（`_NAMING_FRONTMATTER_STANDARD`）を一般化したバージョンです。

---

## D.1 命名標準

### D.1.1 atom

```
<category>_<topic>_<subtopic>.md

例:
combat_global_cooldown_constant.md
narrative_voice_profile_K_007.md
ui_button_primary_style.md
```

snake_caseで記述し、カテゴリーをprefixとして付けます。

### D.1.2 決定カード

```
D<YEAR>_Q<QUARTER>_<NUMBER>.md

例:
D2026_Q2_017.md
```

年・四半期・番号で構成します。

### D.1.3 議事録

```
<category>_<YYYY-MM-DD>[_<seq>].md

例:
95_BattleTF_2026-05-18.md
art_review_2026-05-18_1.md
art_review_2026-05-18_2.md
```

### D.1.4 仕様書

```
spec_<topic>.md

例:
spec_combat_global_cooldown.md
spec_guild_attendance.md
```

### D.1.5 レポート

```
report_<period>_<type>.md

例:
report_W21_alpha_gap.md
report_Q2_user_voice.md
```

---

## D.2 Frontmatter標準

### D.2.1 atom

```yaml
---
name: combat_global_cooldown_constant
description: 戦闘システムのグローバルクールダウン標準値の定義
type: atom
category: combat
status: active
priority: P0
related_atoms:
  - combat_skill_cooldown_rule
  - combat_healing_skill_cooldown_exception
created: 2026-05-18
last_modified: 2026-05-18
related:
  derives_from: [combat_design_principle]
  affects: [combat_skill_cooldown_rule, ui_skill_cooldown_indicator]
---
```

### D.2.2 決定カード

```yaml
---
decision_id: D2026_Q2_017
title: 戦闘のグローバルクールダウンを0.5秒に統一
type: system_change
status: active
created: 2026-05-18
created_by: チームメンバーA
approved_by: イ・ミンス
scope:
  - combat_system
affected_atoms: [...]
implementation:
  target_build: 2026-05-18
verification:
  layer_1: passed
  layer_2: passed
  layer_3: pending
---
```

### D.2.3 議事録

```yaml
---
type: meeting_note
category: battle
date: 2026-05-18
attendees: [チームメンバーA, チームメンバーB, イ・ミンス]
related_atoms: [...]
---
```

### D.2.4 仕様書

```yaml
---
title: ギルド出席機能の仕様
type: spec
priority: P1
target_milestone: MS2
---
```

---

## D.3 必須フィールドと任意フィールド

### D.3.1 必須フィールド

| 文書の種類 | 必須 |
|---|---|
| atom | name, description, type, category, status |
| 決定カード | decision_id, title, type, status, created, scope |
| 議事録 | type, category, date, attendees |
| 仕様書 | title, type, priority |

### D.3.2 任意フィールド（あれば望ましい）

| 文書の種類 | 任意 |
|---|---|
| atom | related, last_modified, priority |
| 決定カード | rationale, related_decisions, verification |
| 議事録 | related_atoms, sub_topic |
| 仕様書 | target_milestone, related_atoms |

---

## D.4 Lintによる自動チェック

```bash
# frontmatter_lint.py

for file in glob("**/*.md"):
    fm = parse_frontmatter(file)
    if not fm:
        warn(f"{file}: frontmatterなし")
    
    doc_type = infer_type_from_filename(file)
    required = REQUIRED_FIELDS[doc_type]
    
    for field in required:
        if field not in fm:
            warn(f"{file}: 必須フィールド{field}が欠落")
```

ビルド時に自動実行されます。違反があればalertを出します。

---

## D.5 命名衝突の防止

| 領域 | 防止策 |
|---|---|
| atom name | グローバルでunique |
| 決定ID | 四半期内でunique |
| 会議ID | 日付 + seq |
| ファイル名 | フォルダー内でunique |

命名が衝突したときは自動でブロックします。

---

## D.6 変更手順

### D.6.1 atom名の変更

```
1. 新しい名前のatomを作成
2. 既存atomのすべてのwikilinkを新しい名前に更新（自動）
3. 既存atomをdeprecated + redirectに
4. 1か月後に_archiveへ移動
```

性急な名前変更は、資料を損なうリスクがあります。

### D.6.2 frontmatter標準の変更

```
1. 変更理由の提案（decision手続き）
2. 既存全文書のマイグレーションスクリプト
3. ビルドlintの更新
4. チームへの通知
```

---

## D.7 読者向けの補足

本標準は著者の環境に合わせたものです。読者の皆さんは、ご自分の環境に合わせて調整してください。要点は次のとおりです。

| 要点 | 理由 |
|---|---|
| 命名の一貫性 | 検索・自動化 |
| Frontmatter標準 | ツールとの親和性 |
| 必須・任意の分離 | 作成負担↓ |
| Lintの自動化 | 標準の強制 |
| 変更手順 | 資料の保護 |
