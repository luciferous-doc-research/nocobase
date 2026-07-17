# ワークフローの日時計算ノードで二つの時間フィールドの差分を算出

## 知りたいこと

ワークフローにおいて、二つの時間フィールドの差分を日時計算ノードで算出できますか？

## 目的

通知日時の任意の変更を考えている。通知日時には次回通知タイミングとして日時 (タイムゾーンあり)だが、時間フィールドの差分がわかれば目的の日時にずらすことができるから。

## 調査サマリー

**結論: 可能です。** 日時計算ノード（Date Calculation ノード）には「Calculate the difference with another time（別の時間との差分を計算する）」という関数があり、これを使うことで二つの時間フィールドの差分を算出できます。

- 参照ドキュメント: `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（日本語版: `./repository/docs/docs/ja/workflow/nodes/date-calculation.md`）
- 日時計算ノードは9つの計算関数をパイプライン形式（前段の出力が次段の入力）でチェーンできるノード。入力値（Input Value）は変数または日付定数、入力値タイプは「Date」または「Number」を選択する。
- 「Calculate the difference with another time」関数の仕様:
  - 入力値タイプ: Date
  - パラメータ: 比較対象の日付（定数 または ワークフローコンテキストの変数を指定可能）／時間単位／絶対値を取るか／丸め処理（小数保持・四捨五入・切り上げ・切り下げ）
  - 出力値タイプ: Number
  - 例: 入力値 `2024-7-15 00:00:00`、比較対象 `2024-7-16 06:00:00`、単位「day」、絶対値なし、小数保持 → 結果は `-1.25`
- したがって、ワークフロー上の一方の時間フィールドを「入力値」、もう一方の時間フィールドを「比較対象の日付（ワークフローコンテキストの変数）」として設定すれば、両者の差分を数値として取得できる。この差分値を後続のノード（例: 加算/減算関数や別の日時計算ノード）で通知日時の調整に利用できる。
- 参照ドキュメント: `./repository/docs/docs/en/workflow/advanced/variables.md`（日本語版: `./repository/docs/docs/ja/workflow/advanced/variables.md`）
  - ワークフロー内のシステム時刻・日付範囲パラメータは「サーバーに設定されたタイムゾーン」に基づいて扱われる旨の記述あり。
- **未解決の疑問点**: 入力値・比較対象の日付が異なるタイムゾーンを持つ場合の差分計算の具体的な挙動について、今回調査したドキュメントには明示的な記述がなかった。
- 参照ドキュメント（プラグインメタ情報）: `./repository/docs/docs/en/plugins/@nocobase/plugin-workflow-date-calculation/index.md`（日本語版: `./repository/docs/docs/ja/plugins/@nocobase/plugin-workflow-date-calculation/index.md`）
  - `defaultEnabled: false` と記載があり、デフォルトでは無効になっている可能性がある点に留意（有効化手順は本調査の範囲外）。

詳細はログファイル（`reports/0007_workflow-datetime-diff-two-fields/log.md`）を参照。

---

## 完了サマリー

- **完了日時**: 2026-07-13T11:20:00+09:00
- **ログファイル**: `reports/0007_workflow-datetime-diff-two-fields/log.md`
- **備考**: なし

---

## 追加調査（訂正）: 「時間」フィールド（日付を含まない時刻のみ）について

初回の調査サマリーは NocoBase の「日時（DateTime）」フィールドタイプ（日付+時刻）を前提とした内容だった。ユーザーの本来の意図は、NocoBase のフィールドタイプのうち「時間（Time）」— **日付を含まず時刻のみを保持するフィールド** — 同士の差分計算だったため、この点について追加調査した。

### 調査結果

- NocoBase のフィールドタイプ「Time」は、日付情報を持たず時刻のみを保持するフィールドタイプである（例値: `15:30:00`）。DB上では MySQL の `TIME`、PostgreSQL の `TIME WITHOUT TIME ZONE` に対応する。
  - 参照ドキュメント: `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/index.md`（日本語版: `./repository/docs/docs/ja/data-sources/data-modeling/collection-fields/datetime/index.md`）
  - 参照ドキュメント: `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/time.md`（日本語版: `./repository/docs/docs/ja/data-sources/data-modeling/collection-fields/datetime/time.md`、本文は "To be added" のみで内容未整備）
- 日時計算ノード（Date Calculation）のドキュメントには、入力値タイプとして「Date」「Number」の2種類のみが定義されており、「Time」フィールドタイプを専用に扱う記述は存在しない。「Date type」の説明は次の通り: 「the input value can ultimately be converted to a date-time type, such as a numeric timestamp or a string representing time.」— これは日時に変換可能な値全般を指す一般的な説明であり、NocoBase の「Time」フィールドタイプ（日付を持たない時刻のみ）を明示的に対象としたものではない。
  - 参照ドキュメント: `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（日本語版: `./repository/docs/docs/ja/workflow/nodes/date-calculation.md`）
- **結論（ドキュメント調査の範囲では不明瞭）**: 「時間（Time）」フィールドタイプ同士の差分を日時計算ノードで計算できるかどうかについて、ドキュメント上に明示的な記述は見つからなかった。「Calculate the difference with another time」関数は入力値タイプが「Date」であることが前提であり、「Time」型の値（例: `15:30:00` のような日付を持たない時刻文字列）を直接この関数に渡した場合にどう解釈されるか（例えば当日の日付を補完して計算されるのか、エラーになるのか）は、調査したドキュメントの範囲では確認できなかった。
- **未解決の疑問点（追加）**: 「Time」フィールドタイプの値を日時計算ノードの入力値・比較対象として使用した場合の具体的な挙動（日付補完の有無、エラーハンドリング等）はドキュメントに記載がなく、確認するには実装（ソースコード）の調査または実機検証が必要と考えられる。

詳細はログファイル（`reports/0007_workflow-datetime-diff-two-fields/log.md`）の追加調査セクションを参照。
