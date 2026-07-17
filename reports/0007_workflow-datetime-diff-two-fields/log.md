# 0007_workflow-datetime-diff-two-fields 調査ログ

## 基本情報

- **タスクファイル**: reports/0007_workflow-datetime-diff-two-fields/0007_workflow-datetime-diff-two-fields.md
- **開始日時**: 2026-07-13T11:07:09+09:00
- **完了日時**: 2026-07-13T11:20:00+09:00
- **追加調査実施日時**: 2026-07-13T11:12:00+09:00（詳細は下記「追加調査」セクション参照）

## タスク概要

ワークフローにおいて、二つの時間フィールドの差分を日時計算ノードで算出できますか？

目的: 通知日時の任意の変更を考えている。通知日時には次回通知タイミングとして日時 (タイムゾーンあり)だが、時間フィールドの差分がわかれば目的の日時にずらすことができるから。

## 調査ファイル一覧

- `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（日本語版: `./repository/docs/docs/ja/workflow/nodes/date-calculation.md`）
- `./repository/docs/docs/en/plugins/@nocobase/plugin-workflow-date-calculation/index.md`（日本語版: `./repository/docs/docs/ja/plugins/@nocobase/plugin-workflow-date-calculation/index.md`）
- `./repository/docs/docs/en/workflow/advanced/variables.md`（日本語版: `./repository/docs/docs/ja/workflow/advanced/variables.md`）

## 調査結果

### `./repository/docs/docs/en/workflow/nodes/date-calculation.md`

「Date Calculation」ノード（日時計算ノード）についての説明ドキュメント。冒頭（1-9行目）で次のように紹介されている。

> The Date Calculation node provides nine calculation functions, including adding a time period, subtracting a time period, formatted output of a time string, and duration unit conversion. Each function has specific input and output value types, and can also receive results from other nodes as parameter variables. It uses a calculation pipeline to chain the results of configured functions to finally obtain an expected output.

日時計算ノードは9つの計算関数を提供し、パイプライン形式（前段の関数の出力が次の段の入力になる）で複数の計算・変換をチェーンできる。

**ノード作成・設定**（12-40行目）:
- ワークフロー設定画面で「+」ボタンから「Date Calculation」ノードを追加する。
- 入力値（Input Value）: 変数（このワークフローをトリガーしたデータや上流ノードの結果）または日付定数を指定できる。
- 入力値タイプ（Input Value Type）: 「Date」（日付・時刻に変換可能な値。数値タイムスタンプや時刻を表す文字列など）または「Number」の2種類があり、後続の計算ステップの選択肢に影響するため正しく選ぶ必要がある。
- 計算ステップ（Calculation Steps）: 各ステップは計算関数とそのパラメータ設定からなる。パイプライン設計で前段の出力が次段の入力になる。各ステップ後の出力型は固定され、次のステップで利用可能な関数に影響する。型が一致する場合のみ計算を継続でき、そうでない場合はそのステップの結果がノードの最終出力になる。

**9つの計算関数**（42-125行目）のうち、本調査で重要なのは「Calculate the difference with another time（別の時間との差分を計算する）」（62-75行目）:

> ### Calculate the difference with another time
> -   Receives input value type: Date
> -   Parameters
>     -   The date to calculate the difference with, which can be a date constant or a variable from the workflow context.
>     -   Time unit.
>     -   Whether to take the absolute value.
>     -   Rounding operation: Options include keeping decimals, rounding, rounding up, and rounding down.
> -   Output value type: Number
> -   Example: When the input value is `2024-7-15 00:00:00`, the comparison object is `2024-7-16 06:00:00`, the unit is "day", absolute value is not taken, and decimals are kept, the calculation result is `-1.25`.
>
> :::info{title=Tip}
> When absolute value and rounding are configured simultaneously, the absolute value is taken first, then rounding is applied.
> :::

この関数は、入力値（Date型）と、比較対象の日付（定数またはワークフローコンテキストの変数のいずれかを指定可能）との差分を、指定した時間単位（day, hour, minute など）で計算し、Number型で出力する。絶対値を取るかどうか、丸め処理（小数保持・四捨五入・切り上げ・切り下げ）も設定できる。

つまり、ワークフローの二つの時間フィールドについて、片方を「入力値」、もう片方を「比較対象の日付（変数）」として設定すれば、両者の差分を日時計算ノードで算出できる。

他の8関数（参考として記録）:
- Add a period of time（期間を加算）: Date → Date
- Subtract a period of time（期間を減算）: Date → Date
- Get the value of a time in a specific unit（特定単位での値取得。例: 日を取得）: Date → Number
- Set the date to the start of a specific unit（特定単位の開始時刻に設定）: Date → Date
- Set the date to the end of a specific unit（特定単位の終了時刻に設定）: Date → Date
- Check for leap year（うるう年判定）: Date → Boolean
- Format as string（文字列としてフォーマット、Day.js形式）: Date → String
- Convert unit（単位変換、例: 週→日）: Number → Number

**Example セクション**（127-137行目）: 商品作成時に、作成日時の翌週最終日23:59:59をプロモーション終了時刻として設定する例。「次週の時刻を計算する」→「その週の最終日23:59:59にリセットする」という2ステップのパイプラインで実現している。これはパイプラインによる複数計算のチェーンの実例。

### `./repository/docs/docs/en/plugins/@nocobase/plugin-workflow-date-calculation/index.md`

プラグインのメタ情報のみのファイル（frontmatter + タイトルのみ、本文なし）。
- `packageName: '@nocobase/plugin-workflow-date-calculation'`
- `description: "Used to do date calculation in workflows, such as manipulating, comparing and formatting, etc."`
- `isFree: true`, `builtIn: true`, `defaultEnabled: false`
- `supportedVersions: [1.x, 2.x]`

日時計算ノードは組み込み（builtIn）プラグインだが、デフォルトでは無効（defaultEnabled: false）であることが判明。使用するには有効化が必要と考えられる。

### `./repository/docs/docs/en/workflow/advanced/variables.md`

「System Built-in Variables」セクション（55-66行目）にタイムゾーン関連の記述がある。

> ### System Time
> Gets the system time at the moment the node is executed. The time zone of this time is the one set on the server.
>
> ### Date Range Parameters
> Can be used when configuring date field filter conditions in query, update, and delete nodes. It is only supported for "is" comparisons. Both the start and end times of the date range are based on the time zone set on the server.

これらの記述から、ワークフロー内で扱う日時（システム時刻や日付範囲パラメータ）は「サーバーに設定されたタイムゾーン」に基づくことがわかる。日時計算ノード自体のドキュメントにはタイムゾーンの扱いについての明示的な記述はなかったが、ワークフロー全体としてはサーバー設定のタイムゾーンで日時が扱われる、という前提を確認した。

## 調査アプローチ

1. まず `find ./repository/docs/docs/en -type d -iname "*workflow*"` および `grep -rl` で workflow 関連ディレクトリ・ファイルを洗い出し、日時計算に関連するドキュメントの候補を特定した。
2. `./repository/docs/docs/en/workflow/nodes/date-calculation.md` （ノードの使い方の詳細）と `./repository/docs/docs/en/plugins/@nocobase/plugin-workflow-date-calculation/index.md` （プラグインのメタ情報）の2ファイルが日時計算ノードの中心的なドキュメントであると判断し、全文を読み込んだ。
3. 「知りたいこと」の核心である「二つの時間フィールドの差分算出が可能か」について、date-calculation.md 内の「Calculate the difference with another time」関数がまさに該当する機能であることを確認した。
4. 「目的」に記載された「タイムゾーンあり」の日時を扱う点について補足情報がないか確認するため、`variables.md` 内でタイムゾーンに関する記述を `grep -n -i "timezone\|time zone"` で検索し、システム時刻・日付範囲パラメータがサーバー設定のタイムゾーンに基づくという記述を発見した。

## 会話内容

### [11:07] ユーザー指示（/add-report 経由）
知りたいこと: ワークフローにおいて、二つの時間フィールドの差分を日時計算ノードで算出できますか？
目的: 通知日時の任意の変更を考えている。通知日時には次回通知タイミングとして日時 (タイムゾーンあり)だが、時間フィールドの差分がわかれば目的の日時にずらすことができるから。

### Claude 対応
`reports/0007_workflow-datetime-diff-two-fields/0007_workflow-datetime-diff-two-fields.md` を作成し、ユーザーに `/report` の続行確認（AskUserQuestion）を行った。ユーザーは「はい（すぐ実行する）」を選択したため、`/report` スキルを `args: "0007"` で起動。

タスクファイルには「## 目的」セクションが存在したため、目的セクション追加の確認は不要と判断し、そのまま調査に進んだ。

日時計算ノードの英語ドキュメント（date-calculation.md）とプラグインメタ情報を読み込み、「Calculate the difference with another time」関数により二つの時間フィールドの差分算出が可能であることを確認。またタイムゾーンについては variables.md にサーバー設定タイムゾーンに基づく旨の記述があることを確認した。

## 判断・意思決定

- 「二つの時間フィールドの差分」を算出する機能として、date-calculation.md の「Calculate the difference with another time」関数が該当すると判断した。この関数は入力値（Date型）と比較対象の日付（定数 or ワークフロー変数）との差分を、指定単位でNumber型として出力する。
- ワークフロー内のフィールド（時間フィールド）を比較対象として「ワークフローコンテキストの変数」から指定できる、という記述（"a variable from the workflow context"）に基づき、二つのフィールドの差分計算が可能と結論づけた。
- タイムゾーンについては、日時計算ノード自体にタイムゾーンの明示的な取り扱いの記述はなかったが、ワークフロー全体としてサーバー設定のタイムゾーンに基づいて日時が扱われるという前提情報を variables.md から補足として記録した。ユーザーの目的（タイムゾーンあり日時のずらし）に関する明確な回答（例えばタイムゾーンをまたぐ差分計算時の挙動）は今回の調査対象ドキュメントには記載がなかった。

## 問題・疑問点

- 日時計算ノードにおいて、入力値・比較対象の日付がそれぞれ異なるタイムゾーンを持つ場合（例: UTC保存のフィールドとタイムゾーン付き日時フィールドの差分計算）にどのように扱われるかについて、明示的な記述は見つからなかった。この点は今回調査したドキュメントの範囲外であり、未解決の疑問として残る。
- 「defaultEnabled: false」との記述があり、日時計算ノードプラグインはデフォルトで無効になっている可能性がある点は、実際の利用にあたって有効化が必要になる場合があることを示唆しているが、有効化手順自体は本調査の範囲外。

---

## 追加調査: 「時間（Time）」フィールド（日付を含まない時刻のみ）について

### 追加調査の経緯

初回調査（上記）は、ユーザーの「知りたいこと」に書かれていた「時間フィールド」という語を、日付+時刻を持つ「日時（DateTime）」フィールドの意味で解釈して進めた。しかしユーザーからの指摘により、本来の意図は NocoBase のフィールドタイプのうち「時間（Time）」— **日付情報を持たず時刻のみを保持するフィールドタイプ** — 同士の差分計算を指していたことが判明した。ユーザーは当初から「日時」と「時間」を区別して記述していたとのことで、解釈を誤ったのは調査側（Claude）である。この訂正を受けて追加調査を行った。

### 追加調査ファイル一覧

- `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/index.md`（日本語版: `./repository/docs/docs/ja/data-sources/data-modeling/collection-fields/datetime/index.md`）
- `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/time.md`（日本語版: `./repository/docs/docs/ja/data-sources/data-modeling/collection-fields/datetime/time.md`）
- `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（既読、再確認）
- `./repository/docs/docs/en/workflow/nodes/loop.md`、`./repository/docs/docs/en/workflow/triggers/collection.md`、`./repository/docs/docs/en/workflow/triggers/schedule.md`（"field type" / "time field" の用例確認のため grep で参照）

### 追加調査結果

#### `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/index.md`

「DateTime Field Types」ページ。NocoBase の日時系フィールドタイプが5種類（DateTime with Time Zone / DateTime without Time Zone / Date without Time / Time / Unix Timestamp）に分類されることを説明している（1-9行目）。

> - **DateTime (with Time Zone):** These values are standardized to UTC (Coordinated Universal Time) and are subject to time zone adjustments when necessary.
> - **DateTime (without Time Zone):** This type stores date and time data without incorporating any time zone information.
> - **Date (without Time):** This format exclusively stores date information, omitting any time component.
> - **Time:** Stores only time information, excluding the date.
> - **Unix Timestamp:** This type represents the number of seconds that have elapsed since January 1, 1970, and is stored as a Unix timestamp.

各フィールドタイプの例値を示す表（13-19行目）:

| Field Type | Example Value | Description |
|---|---|---|
| DateTime (with Time Zone) | 2024-08-24T07:30:00.000Z | Converted to UTC and can be adjusted for time zones |
| DateTime (without Time Zone) | 2024-08-24 15:30:00 | Stores date and time without time zone considerations |
| Date (without Time) | 2024-08-24 | Captures only the date, with no time information |
| **Time** | **15:30:00** | **Captures only the time, excluding any date details** |
| Unix Timestamp | 1724437800 | Represents seconds since 1970-01-01 00:00:00 UTC |

DBとの対応表（25-32行目）: NocoBase の「Time」はMySQLの `TIME`、PostgreSQLの `TIME WITHOUT TIME ZONE` に対応する。PostgreSQLの `TIME WITH TIME ZONE` に対応するNocoBaseフィールドタイプは「-」（存在しない）と明記されている（32行目）。つまり NocoBase の「Time」フィールドタイプにはタイムゾーンの概念がない。

「DateTime Storage Processing Workflow」セクション（37-55行目）では、タイムゾーンあり（DateTime with Time Zone, Unix Timestamp）とタイムゾーンなしの保存処理のワークフロー図が示されているが、「Time」型固有の処理フローについての説明はない。

「UTC」セクション（57-72行目）ではタイムゾーンとUTCの一般的な説明があるのみで、「Time」型には直接関係しない。

#### `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/time.md`

「Time」フィールドタイプの個別ページ。内容は非常に簡素で、以下のみ:

```
# Time

## Introduction

## Field configuration

![20240512181216](https://static-docs.nocobase.com/20240512181216.png)

## Example

To be added.
```

「Introduction」セクションは空、「Example」セクションも「To be added.」（今後追加予定、未整備）となっており、実質的な説明内容はフィールド設定画面のスクリーンショットのみ。ワークフロー（日時計算ノードなど）との連携についての記述は一切ない。

#### `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（再確認）

初回調査時に読んだ内容を「Time」フィールドタイプの観点で再確認した。

- 「Input Value Type」の説明（29-34行目）は次の2択のみ:
  > - Date type: Means the input value can ultimately be converted to a date-time type, such as a numeric timestamp or a string representing time.
  > - Number type: ...
  
  「Date type」の説明文中に "a string representing time" という語句があるが、これは「時刻を表す文字列」全般を指す一般的な説明であり、NocoBase の「Time」フィールドタイプ（日付を持たない `15:30:00` 形式の値）を名指しで扱っているわけではない。この文言だけでは、「Time」型の値をそのまま「Date」型の入力値として扱えるかどうかは判断できない。
- 「Calculate the difference with another time」関数（62-75行目）の説明・パラメータ・例のいずれにも、「Time」フィールドタイプ（日付なし時刻のみ）に関する言及は存在しない。例として示されている値は全て `2024-7-15 00:00:00` のような日付+時刻形式であり、時刻のみの値の例は一切示されていない。
- ドキュメント全文（9-138行目）を通じて "Time field"（NocoBaseのフィールドタイプとしての「Time」）という語は一度も登場しない。「time」という単語自体は "period of time"（期間）や "time unit"（時間単位）といった文脈でのみ使われている。

#### `./repository/docs/docs/en/workflow/nodes/loop.md`, `./repository/docs/docs/en/workflow/triggers/collection.md`, `./repository/docs/docs/en/workflow/triggers/schedule.md`

`grep -n -i "time field\|field type"` で該当箇所を確認したところ、loop.md と triggers/collection.md でヒットしたのは表のヘッダー「Field Name | Field Type」のみで、具体的な「Time」型についての言及ではなかった。

triggers/schedule.md では「Collection time field」（コレクションの時間フィールド）という語がスケジュールトリガーのモード名として登場する（8, 33, 35, 71, 73行目）。

> - Collection time field: Triggering based on the value of a time field in a collection when the time is reached.
> Using a collection's time field to determine the start time is a trigger mode that combines regular scheduled tasks with collection time fields.

ただしこの文脈における「time field」は、トリガーの起点となる日時（例: 注文作成から30分後、など）を保持するフィールド全般を指す用語であり、NocoBase のフィールドタイプ「Time」（日付なし時刻のみ）に限定した説明ではない。実際の例（"30 minutes after the order is created"）も日時（DateTime）フィールドを前提とした説明になっている。したがって、この箇所も今回の疑問（Time型フィールドの差分計算）への直接的な回答にはならないと判断した。

### 追加調査アプローチ

1. NocoBase のコレクションフィールドタイプに関するドキュメントディレクトリを `find` で特定し、`datetime/` サブディレクトリ配下に `time.md`（Timeフィールドタイプ）と `index.md`（DateTime系フィールドタイプの総論）が存在することを確認した。
2. 両ファイルを全文読み込み、「Time」フィールドタイプの定義・特性（日付を持たない、タイムゾーン概念がない）を確認した。
3. 初回調査で読んだ `date-calculation.md` を「Time」フィールドタイプの観点で再読し、「Time」型への言及が一切ないことを確認した。
4. 念のため、ワークフロー関連ドキュメント全体で "time field" という語がどのような文脈で使われているかを `grep` で横断的に確認し、「Collection time field」（スケジュールトリガーのモード名）はNocoBaseの「Time」フィールドタイプとは異なる一般的な意味で使われていることを確認した。

### 追加の会話内容

### [11:10頃] ユーザー指摘
「この調査について追加の調査をして欲しいです。今回差分を比較するのは 日付と時刻の `日時` ではなく、時刻だけの `時間` です。」

### [11:11頃] Claude 対応
当初、新規タスク（0008）として `/add-report` を起動しようとしたが、ユーザーから割り込みが入った。

### [11:12頃] ユーザー訂正
「この調査タスク自体は本来時刻だけの `時間` を想定したものでした。そのため私はNocoBaseのフィールドタイプの `日時` と `時間` を区別して書いていたつもりでした。間違えたのはそちらなので、この調査タスクに追記してください。」

### Claude 対応
新規タスク作成を中止し、既存タスク（0007）のタスクファイル・ログファイルに追加調査として追記する方針に切り替えた。NocoBase の「Time」フィールドタイプに関するドキュメントを調査し、上記の追加調査結果を得た。

### 追加の判断・意思決定

- ユーザーの指摘（誤解釈の訂正）を受け、新規タスク（0008）を作成せず、既存タスク 0007 に追記することとした。理由: これは新しい調査項目ではなく、既存タスクにおける調査対象の誤解を正すものであるため。
- 「Time」フィールドタイプが日時計算ノードで具体的にどう扱われるか（日付補完の有無等）はドキュメントに記載がなかったため、断定的な結論は避け、「ドキュメント調査の範囲では不明瞭」という形で報告することとした。憶測でのソースコード的推測（例: Day.jsの一般的挙動からの類推）は行わず、ドキュメントに書かれている事実のみを根拠として報告した。

### 追加の問題・疑問点

- 「Time」フィールドタイプ（日付なし時刻のみ、例: `15:30:00`）の値を日時計算ノードの「入力値」または「Calculate the difference with another time」関数の比較対象として使用した場合に、具体的にどう解釈されるか（例: 当日の日付が自動補完されるのか、パースエラーになるのか、そもそも入力値タイプ「Date」の選択肢自体を使用できないのか）は、調査対象のドキュメントには記載がなかった。
- この点を確認するには、(a) NocoBase の実装（`@nocobase/plugin-workflow-date-calculation` のソースコード。Day.js を用いた実装と推測されるが未確認）を調査する、または (b) 実機（NocoBase環境）でTimeフィールドを日時計算ノードに接続して動作検証する、のいずれかが必要と考えられる。いずれも本ドキュメント調査（`/report` スキル）の対象範囲外であり、別途対応が必要な事項として記録する。
- `time.md`（Timeフィールドタイプの個別ページ）自体が「Introduction」「Example」ともに未整備（"To be added."）であり、NocoBase公式ドキュメント自体がTimeフィールドタイプについて十分な説明を提供できていない状態であることも一因と考えられる。
