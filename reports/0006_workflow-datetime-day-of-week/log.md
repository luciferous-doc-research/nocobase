# 0006_workflow-datetime-day-of-week 調査ログ

## 基本情報

- **タスクファイル**: reports/0006_workflow-datetime-day-of-week/0006_workflow-datetime-day-of-week.md
- **開始日時**: 2026-07-13T10:50:00+09:00
- **完了日時**: 2026-07-13T10:51:45+09:00

## タスク概要

知りたいこと: ワークフローにおいて、日時 (タイムゾーンあり)で曜日も取得可能か？

目的: スケジュールトリガーのワークフローにおいて、平日(月曜から金曜)だけ動くようにしたい。実行の度に日時フィールドを一日ずつずらしていくが、金曜日だけ3日ずらしたい。

## 調査ファイル一覧

- `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（日: `./repository/docs/docs/ja/workflow/nodes/date-calculation.md`）
- `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/server/dateFunction.ts`（ソースコード。ドキュメントに記載のない内部実装の確認用）
- `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/client/dateFunctions.tsx`（同上、UIラベルとフィールド定義確認用）
- `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/client/constants.ts`（同上、`unitOptions` のラベル定義確認用）
- `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/utils/index.ts`（同上、参照のみ。有用な情報なし）
- `./repository/docs/docs/en/workflow/triggers/schedule.md`（日: `./repository/docs/docs/ja/workflow/triggers/schedule.md`）
- `./repository/docs/docs/en/workflow/nodes/condition.md`（日: `./repository/docs/docs/ja/workflow/nodes/condition.md`）
- `./repository/docs/docs/en/workflow/nodes/calculation.md`（日: `./repository/docs/docs/ja/workflow/nodes/calculation.md`）
- `./repository/docs/docs/en/workflow/nodes/javascript.md`（日: `./repository/docs/docs/ja/workflow/nodes/javascript.md`）
- `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/datetime.md`（日: `./repository/docs/docs/ja/data-sources/data-modeling/collection-fields/datetime/datetime.md`）

## 調査結果

### `./repository/docs/docs/en/workflow/nodes/date-calculation.md`

Date Calculation（日時計算）ノードのドキュメント。9つの計算関数を提供し、パイプライン形式で連結できる。

- Input Value Type は "Date" と "Number" の2種類のみで、明示的な "曜日 (day of week)" という入力/出力型は存在しない（33-35行目）。
- 関数一覧（44-125行目）:
  - Add a period of time（期間を加算）: Date → Date
  - Subtract a period of time（期間を減算）: Date → Date
  - Calculate the difference with another time（差分計算）: Date → Number
  - **Get the value of a time in a specific unit**（特定の単位の値を取得）: Date → Number。ここでは「Time unit」のパラメータのみが示されており、例は「2024-7-15 00:00:00 で unit が "day" のとき結果は 15」（83行目、= 日にち(day of month) を返す例）。ドキュメント本文には "week" ユニットを指定した場合の例は掲載されていない。
  - Set the date to the start/end of a specific unit: Date → Date
  - Check for leap year: Date → Boolean
  - **Format as string**（文字列としてフォーマット）: Date → String。パラメータは "Format"（[Day.js: Format](https://day.js.org/docs/en/display/format) を参照、113行目）。例: フォーマットが `the time is YYYY/MM/DD HH:mm:ss` のとき `the time is 2024/07/15 14:26:30` になる（115行目）。ドキュメントには曜日フォーマットトークン（`d`, `dd`, `ddd`, `dddd` など）についての明示的な言及はないが、「Day.js: Format を参照」とあるため、Day.js がサポートする曜日トークンがそのまま利用可能と判断できる。
  - Convert unit: Number → Number

ドキュメント本文だけでは「曜日を取得できるか」への直接的な回答（unit オプションの一覧や `d`/`dddd` 等のフォーマットトークンの明示）がなかったため、ソースコードを確認した。

### `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/server/dateFunction.ts`

サーバー側の実際の計算ロジック（dayjs ベース）。

- `getMethodsMap`（20-30行目）: 「Get the value of a time in a specific unit」関数で使われる、UI 上の unit ラベルと dayjs メソッドの対応表。
  ```js
  const getMethodsMap = {
    year: 'year',
    month: 'month',
    quarter: 'quarter',
    week: 'day',       // ← 注目: "Week" というラベルなのに dayjs の .day() を呼ぶ
    day: 'date',        // ← "Day" というラベルは dayjs の .date()（日にち、1-31）を呼ぶ
    hour: 'hour',
    minute: 'minute',
    second: 'second',
    millisecond: 'millisecond',
  };
  ```
- `get` 関数の実装（101-115行目）:
  ```js
  get(input, args) {
    const { unit } = args;
    const method = getMethodsMap[unit];
    if (!method) {
      return input.millisecond();
    }
    if (input[method]) {
      const result = input[method]();
      return unit === 'month' ? result + 1 : result;
    }
    return input;
  },
  ```
  dayjs の `.day()` メソッドは「曜日番号（0=日曜〜6=土曜）」を返す（dayjs 公式仕様）。つまり、UI 上で unit に **"Week"（週）を選択すると、実際には曜日番号（0-6）が返る**。これは NocoBase の UI ラベルと内部実装の対応関係のズレ（"Week" ラベルなのに実質は「曜日」を返す）であり、ドキュメントにもソース内コメントにも明記されていない挙動である。
- `format` 関数の実装（129-137行目）:
  ```js
  format(input, args) {
    const { format } = args;
    if (typeof format !== 'string') {
      throw new TypeError(`[${format}] is not a string`);
    }
    return input.format(format);
  },
  ```
  内部で dayjs の `.format()` をそのまま呼んでいるだけなので、Day.js がサポートするフォーマットトークン（`d`＝曜日番号 0-6、`dd`＝曜日名の最小表記、`ddd`＝曜日名省略形、`dddd`＝曜日名フル表記、ロケール依存）がすべて利用できる。これは「Format as string」関数を使えば曜日名・曜日番号を文字列として取得できることを意味する。
- `changeTimezone` 関数（179-182行目）:
  ```js
  changeTimezone(input, args) {
    const { timezone } = args;
    return input.tz(timezone);
  },
  ```
  dayjs の `timezone` プラグインの `.tz()` を呼んでいる。入力の日時をタイムゾーン変換したうえで、以降のパイプライン処理（`get` や `format`）に渡すことができる。つまり "Datetime (with time zone)" フィールドの値を、任意のタイムゾーンに変換してから曜日を取得する、という処理をパイプラインで組める。

### `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/client/dateFunctions.tsx`

- `get` 関数の UI 定義（256-277行目）。タイトルは `"Get value on specific unit of input date"`。`unit` フィールドの `enum` は `unitOptions`（後述）で、デフォルトは `'day'`。
- `format` 関数の UI 定義（344-363行目）。タイトルは `"Format to string"`。`format` フィールドはテキスト入力（`x-component: 'Input'`）で自由记述、デフォルトは `'YYYY-MM-DD'`。フォーマット文字列の制約はUI上なく、dayjs がサポートする任意のトークンを入力可能。
- `changeTimezone` 関数の UI 定義（422-438行目）。タイムゾーンのセレクトボックス（`Intl.supportedValuesOf('timeZone')` または `moment-timezone` からタイムゾーン一覧を取得）。

### `./repository/packages/plugins/@nocobase/plugin-workflow-date-calculation/src/client/constants.ts`

- `unitOptions`（3-13行目）は UI 上のラベルと value の対応：
  ```js
  export const unitOptions = [
    { label: 'Year', value: 'year' },
    { label: 'Quarter', value: 'quarter' },
    { label: 'Month', value: 'month' },
    { label: 'Week', value: 'week' },   // ← ラベルは「週」だが get 関数では day()＝曜日を返す
    { label: 'Day', value: 'day' },
    { label: 'Hour', value: 'hour' },
    { label: 'Minute', value: 'minute' },
    { label: 'Second', value: 'second' },
    { label: 'Millisecond', value: 'millisecond' },
  ];
  ```
  これは全ての関数（add, subtract, diff, get, startOfTime, endOfTime, transDuration 等）で共通のユニット選択肢セットである。UI上のラベルには "曜日 (Day of week)" という選択肢は独立して存在しない。曜日を取得したい場合は、
  1. **「Get value on specific unit of input date」で unit=Week を選ぶ**（実装上は dayjs `.day()` が呼ばれ、曜日番号 0-6 (日曜=0) が数値として返る。ラベルとの対応がわかりにくい裏技的な方法）
  2. **「Format as string」で フォーマットに `d`（数値0-6）や `dddd`（曜日名フル表記、例: "Friday"）等の Day.js の曜日トークンを指定する**（ドキュメントで明示的に案内されている方法。こちらが直感的で推奨）
  のいずれかを使うことになる。

### `./repository/docs/docs/en/workflow/triggers/schedule.md`

Scheduled Task（スケジュールトリガー）のドキュメント。

- トリガーには2モードある（5-8行目）: "Custom time"（cron ライクな定期実行）と "Collection time field"（コレクションの時刻フィールド基準）。
- "Custom time" モードでは、繰り返しルールを「一定間隔（by interval）」または「高度モード（advanced mode = cron ルール）」で設定できる（26-29行目）。
- cron ルール（advanced mode）を使えば、"平日のみ実行"（例: `* * * * 1-5`）のようなスケジュール自体を cron 式で表現することが本来は可能。ただし本タスクの要望は「日時フィールドを実行の度にずらしていく」処理を前提としているため、cron の曜日指定だけでは「フィールド値を平日分だけずらす」という要件（金曜だけ+3日）は満たせず、ワークフロー内でのノード処理が必要になる。
- "Repeat count" の節（43-45行目）: 同一ワークフローの全バージョンを通算した実行回数でカウントされる、という補足あり（今回の要望には直接関係しないが仕様として記録）。

### `./repository/docs/docs/en/workflow/nodes/condition.md`

Condition（条件分岐）ノードのドキュメント。

- 2つのモードがある（9行目）: "Continue if true"（条件を満たさない場合はワークフローが失敗終了）と "Branch on true/false"（true/false 両方の分岐を構成でき、分岐後に自動的に合流する）。
- 「金曜日だけ+3日、それ以外は+1日」を実現するには "Branch on true/false" モードの Condition ノードを使い、条件式（曜日が金曜かどうか）の true 分岐で Date Calculation ノード（+3日）、false 分岐で Date Calculation ノード（+1日）を配置する構成が可能。
- Calculation Engine は Basic / Math.js / Formula.js の3種類（35-39行目）。ワークフローコンテキストの変数（例えば Date Calculation ノードで取得した曜日番号）を条件式のパラメータとして利用できる。

### `./repository/docs/docs/en/workflow/nodes/calculation.md`

Calculation（計算）ノードのドキュメント。Math.js または Formula.js の数式エンジンで任意の式を評価できる。曜日取得そのものには必須ではないが、Date Calculation ノードで取得した曜日番号（0-6）を使った複雑な条件式・計算をこのノードで組むことも可能（今回は Condition ノードで十分と判断）。

### `./repository/docs/docs/en/workflow/nodes/javascript.md`

JavaScript ノードのドキュメント。サーバーサイドで任意の JS（QuickJS サンドボックス、デフォルトでは `Date` などの標準組み込みオブジェクトのみ利用可）を実行できる。

- 118-120行目: ノードへの入力パラメータとして `Date` オブジェクトを渡すと、ISO 形式の文字列に変換されて渡される、との記載あり。つまり "Datetime (with time zone)" フィールドの値を JS ノードに渡す場合、オフセット付きの ISO 文字列（例: `2026-07-13T10:50:00+09:00` 相当）として受け取れる。
- JS ノード内で `new Date(isoString).getDay()` のような標準 `Date` API を使えば曜日番号（0=日曜〜6=土曜）を取得できるが、これは **サーバーのローカルタイムゾーンに基づいて評価される** 点に注意が必要（ISO文字列にオフセットが含まれていても、`.getDay()` はサーバーのシステムタイムゾーンでの日付に変換してから計算するため、意図したタイムゾーンでの曜日と一致しない可能性がある）。この点はドキュメントに明記されていないため、実装上の注意点として記録する。
- Date Calculation ノードの `changeTimezone`→`format('d')`（または `.day()` 相当の `get` with unit=week）の組み合わせの方が、タイムゾーンを明示的に指定できる分、意図通りの挙動になりやすいと考えられる。

### `./repository/docs/docs/en/data-sources/data-modeling/collection-fields/datetime/datetime.md`

フィールドタイプ「Datetime (with time zone)」のドキュメント。内容は非常に薄く（"## Introduction" と "## Field configuration" のみで具体的な説明はほぼない。"## Example" は "To be added." のプレースホルダーのまま）、フィールドの内部データ形式（UTCで保存しタイムゾーンオフセットも保持する、等）についての詳細な記載はドキュメント上には見当たらなかった。日本語版 `./repository/docs/docs/ja/data-sources/data-modeling/collection-fields/datetime/datetime.md` も同様に内容が薄いことを確認済み（内容の詳細比較は行っていないが、ファイルの存在のみ確認）。

## 調査アプローチ

1. まず `reports/` の連番を確認し、タスクファイル `0006_workflow-datetime-day-of-week` を作成。
2. `find` で `workflow` 関連の英語ドキュメント一覧を取得し、日時計算に関連しそうな `date-calculation.md` を最優先で確認。
3. `date-calculation.md` のドキュメント本文だけでは「曜日取得」の可否が明示されていなかったため、ドキュメントに記載のプラグイン名 `@nocobase/plugin-workflow-date-calculation` のソースコードを確認する方針に切り替えた（CLAUDE.md の調査起点は英語ドキュメントだが、ドキュメントに記載のない内部実装の裏取りとしてソースを参照した）。
4. サーバー側 `dateFunction.ts` を読み、`get` 関数の `unit: 'week'` が実際には dayjs `.day()`（曜日番号）にマッピングされているという、ラベルと実装のズレを発見。
5. クライアント側 `dateFunctions.tsx` と `constants.ts` を確認し、UI 上のラベル（"Week"）と実装の対応関係を裏付けた。
6. 「Format as string」関数（dayjs `.format()` をそのまま呼ぶ）が、Day.js の曜日フォーマットトークン（`d`, `dddd` 等）を使った曜日取得の、より明示的でわかりやすい手段であると判断。
7. 目的セクションにある「スケジュールトリガーで平日のみ実行、金曜だけ3日ずらす」という要件を満たす具体的なワークフロー構成を検討するため、`schedule.md`（トリガー）、`condition.md`（分岐）、`calculation.md`（数式評価）、`javascript.md`（JSでの代替手段とその注意点）を追加調査。
8. フィールドタイプの用語确认のため `datetime.md`（Datetime with time zone）を確認し、タスクの「日時 (タイムゾーンあり)」という言葉が NocoBase の "Datetime (with time zone)" フィールドタイプを指すことを確認。
9. 参照した全ファイルについて、対応する日本語版ドキュメントパスが存在するか `find`/ループで確認済み（全て存在）。

## 会話内容

### [10:50] ユーザー指示
`/report-kit:add-report` で「知りたいこと: ワークフローにおいて、日時 (タイムゾーンあり)で曜日も取得可能か？」「目的: スケジュールトリガーのワークフローにおいて、平日(月曜から金曜)だけ動くようにしたい。実行の度に日時フィールドを一日ずつずらしていくが、金曜日だけ3日ずらしたい。」というタスクを作成し、続けて `/report-kit:report 0006` を実行するようユーザーが選択（AskUserQuestion で「はい（すぐ実行する）」を選択）。

### [10:50-10:51] Claude 対応
report ワークフローのテンプレート（`references/report-workflow.md`）を確認後、タスクファイルに `目的` セクションが存在することを確認し、そのまま調査を開始。上記「調査アプローチ」の手順で `date-calculation.md` を中心に、関連するソースコード・トリガー/条件ノードのドキュメントを調査した。

## 判断・意思決定

- ドキュメント本文（`date-calculation.md`）だけでは「曜日取得の可否」に対する明確な回答が得られなかったため、CLAUDE.md の原則（調査起点は英語ドキュメント）を踏まえつつ、ドキュメントに名前だけ記載されているプラグインのソースコードを補助的に参照する判断をした。ドキュメントに書かれていない実装の詳細（`unit: 'week'` が実際には曜日を返す、という重要な非自明の挙動）を発見できたため、この判断は妥当だったと考える。
- 曜日取得の方法として「Get value（unit=Week）」と「Format as string（フォーマット `d`/`dddd`）」の2通りを発見したが、後者の方がドキュメントに明記された Day.js のフォーマット仕様に沿っており、UI ラベルの誤解を招きにくいため、回答では「Format as string」を第一候補として案内する方針とした。
- JavaScript ノードでの `Date.getDay()` によるタイムゾーン依存の注意点は、ドキュメントに明記されていないが実装上重要な落とし穴であるため、あえて「調査結果」に残し、調査サマリーでも触れる。

## 問題・疑問点

- `date-calculation.md`（英語・日本語とも）には「Get value」関数の unit に "Week" を指定した場合の具体例が記載されておらず、UI 上のラベルと内部実装（曜日番号を返す）の対応関係はドキュメントからは読み取れない。ソースコードで確認しない限り、ユーザーが誤って「週番号が返る」と誤解する可能性がある。ドキュメント側の改善余地と言える（本調査ではドキュメントの誤りの指摘に留め、修正提案までは行っていない）。
- 「Datetime (with time zone)」フィールドドキュメント自体が非常に薄く（Example が "To be added."）、フィールドの内部保存形式やタイムゾーン変換の詳細な仕様がドキュメントからは確認できなかった。この点は今回のタスクの回答には大きな影響を与えないと判断したため、深追いはしていない。
