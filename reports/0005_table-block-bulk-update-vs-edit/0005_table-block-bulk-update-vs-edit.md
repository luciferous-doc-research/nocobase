# テーブルブロックの一括更新と一括編集の違い

## 知りたいこと

テーブルブロックの操作において、一括更新と一括編集の違い

## 目的

違いを知ることで使い分けをできるようにしたい。

## 調査サマリー

**一括更新（Bulk Update）と一括編集（Bulk Edit）の本質的な違いは、「更新する値をいつ決めるか」にある。**

- **一括更新（Bulk Update）**: 更新する値を**アクション設定時に管理者が事前定義**しておく。エンドユーザーはボタンをクリックするだけで、選択済み（または全件）のレコードに対して固定の値が一括適用される。設定項目は「Data to update（Selected/All）」と「Field assignment（更新するフィールドと値の割り当て）」のみとシンプル。
  - 出典: `./repository/docs/docs/en/interface-builder/actions/types/bulk-update.md` / `./repository/docs/docs/ja/interface-builder/actions/types/bulk-update.md`
- **一括編集（Bulk Edit）**: ボタンクリック後に**ポップアップでフォームが開き、エンドユーザーがその場でフィールドごとに値を入力**して送信する。フィールドごとに「更新しない（Do not update）」「指定した値に変更（Change to）」「クリア（Clear）」の3つの編集モードを選べるため、より柔軟な一括更新に向く。設定側もフォームブロックを別途組み立てる必要があり、一括更新よりも設定手順が一段階多い。
  - 出典: `./repository/docs/docs/en/interface-builder/actions/types/bulk-edit.md` / `./repository/docs/docs/ja/interface-builder/actions/types/bulk-edit.md`
- 両プラグインとも組み込み（`builtIn: true`）だがデフォルトでは無効（`defaultEnabled: false`）。
  - 出典: `./repository/docs/docs/en/plugins/@nocobase/plugin-action-bulk-update/index.md`、`./repository/docs/docs/en/plugins/@nocobase/plugin-action-bulk-edit/index.md`（日本語版も同ディレクトリ配下の `ja/` に存在し内容一致）
- **使い分けの目安**:
  - 「全選択レコードに同じ固定値を一斉に適用したい」（例: 選択した注文をまとめて『承認待ち』にする）→ 一括更新
  - 「レコードごとに多少異なる値を入力したい、またはフィールドごとに『変更しない/変更する/クリアする』を選びたい」→ 一括編集
  - 参考: これらとは別に、テーブルブロックには「Enable Quick Edit（クイック編集）」という第三の機能があり、テーブル内のセルを1件ずつインラインで直接編集する用途に使う（ステータスや担当者を1件ずつサッと変える場合など）。一括更新・一括編集の「複数件まとめて」とは性質が異なる。
    - 出典: `./repository/docs/docs/en/interface-builder/blocks/data-blocks/table.md`（59〜71行目）、`./repository/docs/docs/en/tutorials/v2/04-forms-and-details.md`（222行目）

**調査中に見つかった注意点（ドキュメント上の非対称性・不整合）**:
- Table / Grid Card / List いずれのデータブロックのドキュメントでも、アクション一覧に「Bulk Update」は明記されているが「Bulk Edit」は明記されていない（記載漏れの可能性）。
  - 出典: `./repository/docs/docs/en/interface-builder/blocks/data-blocks/table.md`、`grid-card.md`、`list.md`（各日本語版も同様）
- 英語版のサイドバー定義 `./repository/docs/docs/en/interface-builder/_meta.json` は "Bulk Update"/"Bulk Edit"/"Add Child" のラベルとリンク先が一項目分ずれている。日本語版 `./repository/docs/docs/ja/interface-builder/_meta.json` は正しく対応している。
- なお「Bulk update」という同名の語がワークフローの更新ノード（`./repository/docs/docs/en/workflow/nodes/update.md`）でも使われているが、これはコレクションイベント発火有無に関するパフォーマンス上のオプションであり、テーブルブロックの一括更新アクションとは別概念。

詳細な調査内容・全引用は `reports/0005_table-block-bulk-update-vs-edit/log.md` を参照。

---

## 完了サマリー

- **完了日時**: 2026-07-02T11:28:38+09:00
- **ログファイル**: `reports/0005_table-block-bulk-update-vs-edit/log.md`
- **備考**: Table/Grid Card/List ブロックのアクション一覧に Bulk Edit が明記されていない点、および英語版 `_meta.json` のラベル/リンク不整合は、ドキュメント調査だけでは原因を特定できなかった未解決の疑問点としてログに記録済み。
