## タスク: Wikiの現在の状態を表示する

以下の情報を収集して、ダッシュボード形式で表示してください。

### 収集する情報

1. `index.md` から Ingest進捗（N / 139話）を読み取る
2. `wiki/` 以下の各ディレクトリのファイル数をカウントする
3. `log.md` から直近5件の操作を読み取る
4. `timeline.md` のイベント数をカウントする

### 表示形式

```
📊 進撃の巨人 Wiki Status
═══════════════════════

進捗:  ch{最新} / ch139  ({パーセント}%)

ページ数:
  Characters : N
  Locations  : N
  Factions   : N
  Events     : N
  Themes     : N
  Items      : N
  Synthesis  : N
  ─────────────
  合計       : N

Timeline イベント数: N

直近の操作:
  [日付] 操作内容
  ...

次のIngest: ch{次の話数}
```

$ARGUMENTS が指定された場合は、その追加情報も表示する:
- `detail` → 各カテゴリのページ一覧も表示
- `graph` → リンク数が多い上位5ページ（ハブページ）を表示
