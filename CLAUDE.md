# 進撃の巨人 Wiki — Schema

LLMエージェントがWikiを構築・維持するためのルールブック。すべての操作はこれに従う。

## ディレクトリ構造

```
attack-on-titan/
├── CLAUDE.md, index.md, log.md, timeline.md
├── wiki/{characters,locations,factions,events,themes,items,synthesis}/
├── summaries/chNNN.md       # 各話要約（Ingest単位）
└── raw/{chapters,interviews,analysis}/   # 生ソース（読み取り専用）
```

## ガードレール（厳守）

1. `raw/` 内のファイルは**絶対に変更しない**
2. `attack-on-titan/` 外のファイルは**絶対に変更しない**
3. Wiki内のテキストはすべて**日本語**
4. ファイル名は日本語可（`[[リンク]]` を自然にするため）
5. 既存ページは**追記・修正**で対応（全面書き換えしない）

## 共通フロントマター規約

すべてのPage Typeで使う共通フィールド:

- `type`: `character` / `location` / `faction` / `event` / `theme` / `item` / `summary`
- `name`: ページの正式名称
- `aliases`: 別名・二つ名（配列）
- `first_appearance`: 初登場話 (`ch001` 形式)
- `spoiler_scope`: このページの内容を読んでも安全な読了範囲（最もネタバレ度の高い情報の初出話数に合わせる。Themeは原則 `complete`）
- リンクは `[[characters/エレン・イェーガー]]` 形式（パス込み）

## Page Types

### Character

```yaml
---
type: character
name: ""
aliases: []
affiliation: []           # 所属組織（最新）
status: alive             # alive / deceased / unknown
first_appearance: ch001
last_appearance: null
spoiler_scope: ch001
related_characters:
  - {name: "", relation: ""}   # 親子, 幼馴染, 上官, 宿敵, etc.
key_events: []            # [[events/xxx]]
themes: []                # [[themes/xxx]]
---
```

本文: **概要**（1-2文） / **経歴**（話数順、各段落冒頭に `(ch0XX)` で出典明記） / **人物像**（性格・動機・信念） / **関係性**（フロントマターより詳しく） / **考察**（synthesis由来ならその旨記載）

### Location

```yaml
---
type: location
name: ""
aliases: []
region: ""                # 壁内, マーレ, etc.
first_appearance: ch001
spoiler_scope: ch001
related_locations: []
key_events: []
---
```

本文: **概要** / **地理・構造** / **歴史**（時系列） / **関連**（位置関係・政治的関係）

### Faction

```yaml
---
type: faction
name: ""
aliases: []
leader: ""                # [[characters/xxx]]
members: []
status: active            # active / dissolved / reformed
first_appearance: ch001
spoiler_scope: ch001
goals: ""                 # 1文
---
```

本文: **概要** / **構成員**（役職付き） / **活動歴**（時系列） / **他勢力との関係**

### Event

```yaml
---
type: event
name: ""
in_story_date: ""         # 845年, etc.
chapters: []              # [ch005, ch006, ch007]
location: ""              # [[locations/xxx]]
participants: []          # [[characters/xxx]]
spoiler_scope: ch005
outcome: ""               # 1文
---
```

本文: **概要**（1-2文） / **背景** / **経過**（時系列） / **結果と影響**（直後＋長期） / **関連する伏線**

### Theme

```yaml
---
type: theme
name: ""
related_characters: []
related_events: []
spoiler_scope: complete   # 原則として全巻読了前提
---
```

本文: **概要**（定義） / **作品での表現** / **キャラクターとの関連** / **考察**

### Item

```yaml
---
type: item
name: ""
aliases: []
category: ""              # 兵器, 巨人の力, 技術, 制度, etc.
first_appearance: ch001
spoiler_scope: ch001
---
```

### Summary

```yaml
---
type: summary
chapter: 1
title: ""                 # サブタイトル
arc: ""                   # 所属アーク
spoiler_scope: ch001
new_characters: []
new_locations: []
key_events: []
---
```

本文: **あらすじ**（3-5文） / **重要な出来事**（箇条書き） / **新情報**（初出設定・事実） / **伏線**（張られた／回収された）

## 操作ワークフロー

### Ingest（1話を取り込む）

1. `raw/chapters/` の該当話を読む
2. ユーザーと対話（あらすじ確認 → 感想・疑問・印象的なシーンを聞く。コメントは `synthesis/` 昇格候補）
3. `summaries/chNNN.md` を作成
4. **新規ページ作成**: 初登場のキャラ・場所・勢力・アイテム
5. **既存ページ更新**: 経歴に追記、関係性更新
6. **Eventページ**: 重要事件があれば作成
7. **timeline.md** に時系列順で挿入
8. 影響を受けたページの `spoiler_scope` を更新
9. `index.md` に新規ページ追加
10. `log.md` に追記

Wikiページの作成・更新は対話の**後**に行う。

### Query（質問への回答）

1. `index.md` で関連ページ特定 → 該当ページを読む
2. 出典への `[[リンク]]` 込みで回答
3. 有用な回答は `synthesis/` への昇格を提案
4. ユーザーが「chNNNまで」と指定したら、`spoiler_scope` がそれ以下のページのみ参照

### Lint（定期メンテ）

チェック項目:
- **Orphan**: index.mdに載るが他ページからリンクされていない
- **リンク切れ**: `[[存在しないページ]]`
- **spoiler_scope不整合**: 参照先より小さい（ネタバレ漏れ）
- **空ページ**: 本文なし
- **Timeline抜け**: Eventページがあるのにtimeline.md未記載
- **片方向リンク**: A→Bがあるが B→A がない
- **未作成ページ候補**: 本文中で複数回言及されているが独立ページなし

## ファイル形式

### index.md

カテゴリ別（Characters / Locations / Factions / Events / Themes / Items / Summaries）に1行ずつ:
`- [[characters/エレン・イェーガー]] — 主人公。調査兵団→イェーガー派 (ch001-)`

### log.md（append-only）

```markdown
## [YYYY-MM-DD] ingest | chNNN「サブタイトル」
- 新規作成: characters/xxx, locations/yyy
- Index更新, Timeline更新
```

### timeline.md

作品内の年代見出し（## 845年）の下に時系列順:
`- [[events/シガンシナ陥落]] — 超大型巨人出現 (ch001-002)`

## Wikilink規約

- 形式: `[[characters/エレン・イェーガー]]`（パス込み）
- 表示名変更: `[[characters/エレン・イェーガー|エレン]]`
- 存在しないページへのリンクは張らず、本文中に `(※要ページ作成: xxx)` と注記
- Ingest時にリンク先の存在を確認してから張る
