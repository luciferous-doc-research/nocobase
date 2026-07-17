# 0005_table-block-bulk-update-vs-edit 調査ログ

## 基本情報

- **タスクファイル**: reports/0005_table-block-bulk-update-vs-edit/0005_table-block-bulk-update-vs-edit.md
- **開始日時**: 2026-07-02T11:20:00+09:00
- **完了日時**: 2026-07-02T11:28:38+09:00

## タスク概要

テーブルブロックの操作において、「一括更新（Bulk Update）」と「一括編集（Bulk Edit）」の違いを知りたい。

目的: 違いを知ることで使い分けをできるようにしたい。

## 調査ファイル一覧

- `./repository/docs/docs/en/interface-builder/actions/types/bulk-update.md` / `./repository/docs/docs/ja/interface-builder/actions/types/bulk-update.md`
- `./repository/docs/docs/en/interface-builder/actions/types/bulk-edit.md` / `./repository/docs/docs/ja/interface-builder/actions/types/bulk-edit.md`
- `./repository/docs/docs/en/plugins/@nocobase/plugin-action-bulk-update/index.md` / `./repository/docs/docs/ja/plugins/@nocobase/plugin-action-bulk-update/index.md`
- `./repository/docs/docs/en/plugins/@nocobase/plugin-action-bulk-edit/index.md` / `./repository/docs/docs/ja/plugins/@nocobase/plugin-action-bulk-edit/index.md`
- `./repository/docs/docs/en/interface-builder/blocks/data-blocks/table.md` / `./repository/docs/docs/ja/interface-builder/blocks/data-blocks/table.md`
- `./repository/docs/docs/en/interface-builder/blocks/data-blocks/grid-card.md` / `./repository/docs/docs/ja/interface-builder/blocks/data-blocks/grid-card.md`
- `./repository/docs/docs/en/interface-builder/blocks/data-blocks/list.md` / `./repository/docs/docs/ja/interface-builder/blocks/data-blocks/list.md`
- `./repository/docs/docs/en/interface-builder/actions/action-settings/linkage-rule.md` / `./repository/docs/docs/ja/interface-builder/actions/action-settings/linkage-rule.md`
- `./repository/docs/docs/en/interface-builder/_meta.json` / `./repository/docs/docs/ja/interface-builder/_meta.json`
- `./repository/docs/docs/en/workflow/nodes/update.md` / `./repository/docs/docs/ja/workflow/nodes/update.md`（関連するが別機能: ワークフローの更新ノード）
- `./repository/docs/docs/en/runjs/context/form.md`（関連するが別機能: RunJS でのフォームフィールドの一括セット）
- `./repository/docs/docs/en/multi-app/multi-space/index.md`（一括編集への言及あり、間接的）
- `./repository/docs/docs/en/api/server/migration.md`、`./repository/docs/docs/en/plugin-development/server/migration.md`（別機能: サーバサイドマイグレーションでの batch update、無関係）
- `./repository/docs/docs/en/tutorials/v2/04-forms-and-details.md`（関連するが別機能: Quick Edit の説明）

## 調査結果

### `./repository/docs/docs/en/interface-builder/actions/types/bulk-update.md`（一括更新）

frontmatter で `pkg: "@nocobase/plugin-action-bulk-update"` と明示されている。

- **Introduction**: "The bulk update action is used when you need to apply the same update to a group of records. Before performing a bulk update, the user needs to pre-define the field assignment logic for the update. This logic will be applied to all selected records when the user clicks the update button."
  - つまり「複数レコードに同じ更新を適用する」ためのアクションであり、**更新するフィールドの値（割り当てロジック）はアクション設定時に事前定義**しておく。ユーザーはボタンをクリックするだけで、その場でフォームに値を入力する操作はない。
- **Action Configuration** の設定項目:
  - **Data to update**: `Selected/All`、デフォルトは `Selected`（選択済みレコードのみ対象。「すべて」も選べる）
  - **Field assignment**: 一括更新するフィールドを設定する。「設定されたフィールドのみが更新される（Only the set fields will be updated）」。例として「注文テーブルで、選択したデータのステータスを一括で『承認待ち』に変更する」設定が挙げられている。
  - その他のボタン付随設定として [Edit button](/interface-builder/actions/action-settings/edit-button)（ボタンのタイトル・タイプ・アイコン編集）、[Linkage rule](/interface-builder/actions/action-settings/linkage-rule)（動的な表示/非表示）、[Double check](/interface-builder/actions/action-settings/double-check)（確認ダイアログ）が言及されている。

日本語版 (`./repository/docs/docs/ja/interface-builder/actions/types/bulk-update.md`) も同一内容（「一括更新」「概要」「アクション設定」「更新対象データ」「フィールドの割り当て」の各見出し）で英語版と一致することを確認。「注文テーブルで一括更新アクションを設定し、選択されたデータを『承認待ち』に一括更新します」という記述も英語版と対応。

### `./repository/docs/docs/en/interface-builder/actions/types/bulk-edit.md`（一括編集）

frontmatter で `pkg: "@nocobase/plugin-action-bulk-edit"` と明示されている。

- **Introduction**: "Bulk Edit is suitable for scenarios requiring flexible batch updates of data. After clicking the Bulk Edit button, you can configure the bulk edit form in a pop-up window and set different update strategies for each field."
  - 一括更新と異なり、ボタンクリック後に**ポップアップウィンドウでフォームが開き**、フィールドごとに異なる更新戦略を設定できる。「柔軟な一括更新が必要なシーン」向けと明記。
- **User Guide → Bulk Edit Form Configuration**（設定側の手順）:
  1. Bulk Edit ボタンを追加する。
  2. 一括編集のスコープ（Selected / All、デフォルト Selected）を設定する。
  3. 一括編集フォームを追加する。
  4. 編集対象フィールドを設定し、送信ボタンを追加する。
  - この手順から、一括編集は「ポップアップ内にフォームブロックを別途配置し、そこに編集可能なフィールドと送信ボタンを設定する」という、一括更新より一段階複雑な構成であることが分かる（一括更新は「Field assignment」設定のみで完結する）。
- **User Guide → Form Submission**（実行側の手順、エンドユーザー操作）:
  1. 編集したい行を選択する。
  2. フィールドの編集モードを選び、送信する値を入力する。
  3. フォームを送信する。
  - 利用可能な編集モード（Available Edit Modes）:
    - **Do not update**: フィールドの値を変更しない。
    - **Change to**: 送信された値にフィールドを更新する。
    - **Clear**: フィールドのデータをクリアする。
  - つまり一括編集では、**実行時にエンドユーザーがフィールドごとに「更新しない/値を変更/クリア」を選択し、値を入力してから送信する**という対話的なフローになる。

日本語版 (`./repository/docs/docs/ja/interface-builder/actions/types/bulk-edit.md`) も内容が一致。「一括編集は、データを柔軟に一括更新する必要があるシーンに適しています」「更新しない」「指定した値に変更」「クリア」の3つの編集モードが同様に記載されている。

### `./repository/docs/docs/en/plugins/@nocobase/plugin-action-bulk-update/index.md` と `plugin-action-bulk-edit/index.md`

両方ともプラグイン一覧向けの最小限のメタデータページで、本文はタイトルのみ（見出しのみで説明文はほぼ空）。frontmatter の `description` フィールドにのみ簡潔な説明がある。

- bulk-update: `title: "Action: Batch update"`, `description: "Batch update all records or selected records."`, `builtIn: true`, `defaultEnabled: false`
- bulk-edit: `title: "Action: Batch edit"`, `description: "Batch edit all records or selected records."`, `builtIn: true`, `defaultEnabled: false`

両プラグインとも `builtIn: true`（組み込みプラグイン）だが `defaultEnabled: false`（デフォルトでは無効）という共通点がある。つまりどちらも標準搭載だが、使うには明示的に有効化が必要。日本語版も同一構成（`title: "アクション：一括更新"` 等）であることを確認。

### `./repository/docs/docs/en/interface-builder/blocks/data-blocks/table.md`（テーブルブロック）

テーブルブロックの Global Actions（グローバルアクション、テーブルヘッダー部に配置するアクション）の一覧（129〜141行目）:

```
- Filter
- Add New
- Delete
- Refresh
- Import
- Export
- Template Print
- Bulk Update
- Export Attachments
- Trigger Workflow
- JS Action
- AI Employee
```

**「Bulk Update（一括更新）」はこの一覧に含まれているが、「Bulk Edit（一括編集）」はこのリストに含まれていない。** Row Actions（行アクション、150〜158行目）の一覧にも Bulk Update/Bulk Edit いずれも含まれない（View, Edit, Delete, Pop-up, Link, Update Record, Template Print, Trigger Workflow, JS Action, AI Employee のみ）。

また、テーブルブロック自体には「Enable Quick Edit（クイック編集を有効化）」という第三の関連機能がある（59〜71行目）: 「Activate "Enable Quick Edit" in the block settings and table column settings to customize which columns can be quickly edited.」（ブロック設定とテーブル列設定で「クイック編集を有効化」をオンにすると、テーブル内で直接インライン編集ができる列をカスタマイズできる）。これは一括更新・一括編集とは異なる「1行ずつをテーブル内でその場編集する」機能であり、混同しないよう注意が必要（後述の疑問点参照）。

日本語版 `./repository/docs/docs/ja/interface-builder/blocks/data-blocks/table.md` の94行目でも `- [一括更新](/interface-builder/actions/types/bulk-update)` のみが列挙されており、「一括編集」へのリンクは無いことを確認（英語版と一致）。

### `./repository/docs/docs/en/interface-builder/blocks/data-blocks/grid-card.md` と `list.md`（グリッドカードブロック・リストブロック）

- grid-card.md の67行目: `- [Bulk update](/interface-builder/actions/types/bulk-update)` のみ記載。Bulk Edit は記載なし。
- list.md の71行目: `- [Bulk Update](/interface-builder/actions/types/bulk-update)` のみ記載。Bulk Edit は記載なし。
- 日本語版 (`./repository/docs/docs/ja/interface-builder/blocks/data-blocks/grid-card.md` 53行目、`./repository/docs/docs/ja/interface-builder/blocks/data-blocks/list.md` 59行目) も同様に「一括更新」のみでいずれも「一括編集」への言及なし。

→ ドキュメント上、Table / Grid Card / List のいずれのデータブロックの「利用可能なアクション一覧」にも Bulk Edit は明示的にリストアップされていない。これは Bulk Update と Bulk Edit の「使い分け」を考える上で重要な非対称性であり、疑問点として後述する。

### `./repository/docs/docs/en/interface-builder/_meta.json`（英語版サイドバー定義）

494〜513行目付近に以下のエントリがある（抜粋）:

```json
{
  "type": "custom-link",
  "label": "Bulk Update",
  "link": "/interface-builder/actions/types/update-record"
},
{
  "type": "custom-link",
  "label": "Bulk Edit",
  "link": "/interface-builder/actions/types/bulk-update"
},
{
  "type": "custom-link",
  "label": "Add Child",
  "link": "/interface-builder/actions/types/bulk-edit"
},
```

**英語版の `_meta.json` は label と link が一項目分ずれている**（"Bulk Update" ラベルが `update-record` にリンクし、"Bulk Edit" ラベルが `bulk-update` にリンクし、"Add Child" ラベルが `bulk-edit` にリンクしている）。これはサイドバーメニュー表示上の不整合（恐らくドキュメント側の編集ミス）であり、実際のページ内容（bulk-update.md, bulk-edit.md 本文）とは無関係。

一方、日本語版 `./repository/docs/docs/ja/interface-builder/_meta.json`（494〜503行目）は次の通り正しく対応している:

```json
{
  "type": "custom-link",
  "label": "一括更新",
  "link": "/interface-builder/actions/types/bulk-update"
},
{
  "type": "custom-link",
  "label": "一括編集",
  "link": "/interface-builder/actions/types/bulk-edit"
},
```

つまり日本語版サイドバーはラベルとリンクが一致しており、英語版側にのみメタデータの不整合が存在する。

### `./repository/docs/docs/en/interface-builder/actions/action-settings/linkage-rule.md`（連携ルール）

47行目: "Example 2: The bulk update button in the header of the orders block table is only available to the Admin role; other roles cannot perform this action."
→ Bulk Update ボタンを連携ルールで「Admin ロールのみに表示する」設定例。Bulk Update/Bulk Edit の違いそのものではなく、Bulk Update ボタンにも他の一般アクション同様に連携ルール（表示/非表示制御）が適用できることを示す例。

### `./repository/docs/docs/en/workflow/nodes/update.md`（ワークフローの更新ノード、参考: 別機能）

これはテーブルブロックの UI アクションではなく、ワークフロー（自動化）内の「更新」ノードの設定オプションについての説明。28〜32行目:

- "**Bulk update**: Does not trigger collection events for each updated record. It offers better performance and is suitable for large-volume update operations."
- "**Update one by one**: Triggers collection events for each updated record. However, it may cause performance issues with large volumes of data and should be used with caution."
- "The choice usually depends on the target data for the update and whether other workflow events need to be triggered. If updating a single record based on the primary key, 'Update one by one' is recommended. If updating multiple records based on conditions, 'Bulk update' is recommended."

→ ここでの「Bulk update」はワークフローノードの実行モード（コレクションイベントを都度発火するかどうかのパフォーマンス上のトレードオフ）を指す用語であり、テーブルブロックの Bulk Update アクションとは**別の概念**（名称が同じだけ）。混同しないよう注意。

### `./repository/docs/docs/en/runjs/context/form.md`（参考: 別機能）

11〜12行目、45〜46行目: RunJS のコンテキストで `ctx.form.setFieldsValue({...})` を使って「Batch update（フォームのフィールド値をまとめてセットする）」という用法。これもテーブルブロックの Bulk Update/Bulk Edit アクションとは無関係の、フォーム連携（linkage）における値の一括セット機能。

### `./repository/docs/docs/en/multi-app/multi-space/index.md`（参考: 間接的言及）

132行目: "Through the page created above, manually edit the data to gradually assign the correct space to the old data (you can also configure bulk editing yourself)."
→ マルチスペース機能の文脈で「自分で一括編集を設定してもよい」という一文のみ。Bulk Edit アクション自体の説明ではなく、利用例の一つとしての言及。

### `./repository/docs/docs/en/tutorials/v2/04-forms-and-details.md`（参考: Quick Edit の説明、別機能との比較用）

222行目: "**When is this useful?** Quick editing is perfect for batch updates like changing status or assignee. For example, an admin browsing the ticket list can click the 'Status' column to quickly change a ticket from 'Pending' to 'In Progress' without opening each one individually."
→ これは Table Block の「Enable Quick Edit」機能（テーブル内セルを直接編集する機能）についてのチュートリアル説明であり、Bulk Update/Bulk Edit アクションとは異なる第三の機能。ただし用途が近い（ステータス変更などの一括的な運用）ため、使い分けを整理する際の比較対象として調査結果に含めた。

### `./repository/docs/docs/en/api/server/migration.md`、`./repository/docs/docs/en/plugin-development/server/migration.md`（無関係、除外確認のみ）

いずれもサーバサイドのプラグインマイグレーション（DBスキーマ変更やレコードの一括書き換えAPI）に関する説明で、"batch update"/"batch modify" という語が使われているが、テーブルブロックのUIアクションとは完全に無関係。除外対象として確認のみ行った。

## 調査アプローチ

1. `grep -rli "bulk update\|bulk edit\|batch update\|batch edit" repository/docs/docs/en` で英語ドキュメント全体から関連ファイルを網羅的に洗い出した。
2. ヒットしたファイルのうち、明らかにテーブルブロックのアクションに直接関係するもの（`interface-builder/actions/types/bulk-update.md`, `bulk-edit.md`、プラグイン index ページ、テーブル/グリッドカード/リストのブロックドキュメント）を優先して詳細に読んだ。
3. `table.md` の Global Actions / Row Actions 一覧を確認したところ Bulk Edit が含まれていないことに気づき、他のデータブロック（grid-card, list）でも同様の傾向がないか `grep -n -i "bulk"` で横断確認した。
4. `_meta.json`（サイドバー定義）に Bulk Update / Bulk Edit のエントリがあったため確認したところ、英語版でラベルとリンクの対応がずれている不整合を発見。日本語版の同ファイルと比較し、日本語版は正しく対応していることを確認した。
5. 残りのヒットファイル（workflow/nodes/update.md, runjs/context/form.md, multi-space/index.md, migration 系, tutorials）は文脈を読み、テーブルブロックの Bulk Update/Bulk Edit アクションと直接関係する記述か、別機能（同名だが異なる概念）かを判別した。
6. CLAUDE.md の指示に従い、参照した各英語ドキュメントについて日本語版 (`./repository/docs/docs/ja/...`) の対応ファイルが存在するか確認し、存在するものは内容も併読して整合性を確認した。

## 会話内容

### [11:20] ユーザー指示（システム経由）
`/report-kit:report 0005` が実行され、タスク番号 `0005` が明示的に指定された。

### [11:20] Claude 対応
`reports/0005_table-block-bulk-update-vs-edit/0005_table-block-bulk-update-vs-edit.md` を読み込み、`## 完了サマリー` が存在しないことを確認。「知りたいこと」（一括更新と一括編集の違い）と「目的」（違いを知ることで使い分けをできるようにしたい、という Why に該当するセクション）が明記されていることを確認したため、確認プロンプトなしで調査に進んだ。

その後、`repository/docs/docs/en` 配下を `grep` で網羅的に検索し、関連ドキュメントを特定・精読。テーブルブロック・グリッドカードブロック・リストブロックのいずれにおいても「Bulk Edit」がアクション一覧に明記されていない点、および英語版 `_meta.json` のラベル/リンク不整合を発見した。日本語版ドキュメントの対応状況も確認し、本ログに記録した。

## 判断・意思決定

- **`workflow/nodes/update.md` の "Bulk update" は調査対象外と判断**: 名称は同じだが、これはワークフローの更新ノードの実行モード（パフォーマンス特性）を指す別概念であり、テーブルブロックの Bulk Update アクションの説明ではないため、調査結果には参考情報として区別して記載するに留めた。
- **`tutorials/v2/04-forms-and-details.md` の Quick Edit を比較対象として含めた**: ユーザーの目的が「使い分けができるようにしたい」であるため、Bulk Update と Bulk Edit だけでなく、用途が近い第三の機能（Quick Edit）についても触れておくことが目的（Why）に資すると判断した。
- **`_meta.json` の不整合をログに記録**: ドキュメント内容そのものの理解には直接影響しないが、サイドバー上でユーザーが迷う可能性がある実際の不整合であり、調査中に発見した客観的事実として記録に残す価値があると判断した。report ファイルの調査サマリーにも軽く触れる。

## 問題・疑問点

- **Bulk Edit がデータブロック（Table / Grid Card / List）の「利用可能なアクション一覧」ドキュメントに明記されていない**: `bulk-edit.md` 自体は "Add a Bulk Edit button" という手順を説明しており、UI 上は Table などのブロックに配置できることを前提とした記述（スクリーンショットのファイル名が "Orders-..." で受注テーブルを想定）になっているが、`table.md`／`grid-card.md`／`list.md` の Global Actions 一覧にはいずれも Bulk Edit が列挙されていない。これはドキュメントの記載漏れである可能性がある。実際の UI 上でどのブロックに Bulk Edit ボタンを追加できるかは、今回のドキュメント調査だけでは断定できなかった。
- **英語版 `_meta.json` のラベル/リンク不整合**: 前述の通り、英語版サイドバー定義で "Bulk Update" ラベルが `update-record` ページにリンクするなど、ラベルとリンク先が一項目分ずれている。日本語版では対応関係が正しいため、英語版側のみの記載ミスと推測されるが、ドキュメントソース調査のみでは原因（編集時のリスト挿入ミスなど）までは特定できない。
