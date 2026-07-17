# ワークフローの日時（タイムゾーンあり）フィールドから曜日を取得できるか
## 知りたいこと

ワークフローにおいて、日時 (タイムゾーンあり)で曜日も取得可能か？

## 目的

スケジュールトリガーのワークフローにおいて、平日(月曜から金曜)だけ動くようにしたい。実行の度に日時フィールドを一日ずつずらしていくが、金曜日だけ3日ずらしたい。

---

## 調査サマリー

**結論: 曜日は取得可能。** 「Date Calculation（日時計算）」ノードを使えば、「Datetime (with time zone)」フィールドの値から曜日を取得できる。

- 参照: `./repository/docs/docs/en/workflow/nodes/date-calculation.md`（日: `./repository/docs/docs/ja/workflow/nodes/date-calculation.md`）

### 曜日を取得する2つの方法

1. **「Format as string」関数（推奨）**
   - パラメータ `Format` に Day.js の曜日フォーマットトークンを指定する。
   - `dddd` → 曜日名フル表記（例: `Friday`）、`ddd` → 省略形（例: `Fri`）、`d` → 曜日番号（`0`=日曜〜`6`=土曜、数値文字列）。
   - 内部実装（`plugin-workflow-date-calculation` の `dateFunction.ts` の `format` 関数）は dayjs の `.format()` をそのまま呼んでいるだけなので、[Day.js Format](https://day.js.org/docs/en/display/format) がサポートする曜日トークンはすべて利用可能。
2. **「Get value on specific unit of input date」関数で unit=Week を選ぶ（非推奨・注意）**
   - ソースコード上、UI の unit ラベル "Week" は実は dayjs の `.day()`（曜日番号 0-6、日曜=0）にマッピングされている（`getMethodsMap.week = 'day'`）。つまり「週」というラベルにもかかわらず、実際に返るのは「曜日番号」であり、週番号ではない。
   - ドキュメント本文にはこの対応関係が明記されておらず、ソースコード（`src/server/dateFunction.ts` 20-30行目、101-115行目）を確認して判明した挙動。UIの見た目と挙動が食い違うため誤解を招きやすく、**曜日取得には「Format as string」を使う方が明確で安全**。

### タイムゾーンの扱い

- Date Calculation ノードには `changeTimezone` 関数があり、入力の日時を任意のタイムゾーンに変換してからパイプラインの後続関数（`get` や `format`）に渡せる。「日時 (タイムゾーンあり)」フィールドの値を明示的に狙ったタイムゾーンでの曜日として取得したい場合は、`changeTimezone` → `format('dddd')`（または `d`）の順にパイプラインを組むとよい。
- 代替として JavaScript ノードで `new Date(iso).getDay()` を使うことも可能だが、ISO文字列に変換されて渡された後の `.getDay()` は **サーバーのシステムタイムゾーンで評価される**ため、意図したタイムゾーンでの曜日と一致しない可能性がある（ドキュメントに明記なし、実装上の注意点）。タイムゾーンを明示制御したい場合は Date Calculation ノードの方が安全。

### 「平日のみ実行・金曜だけ+3日」の実現方法（目的への回答）

以下のノード構成で実現可能:

1. Date Calculation ノードで、日時フィールドの値から曜日を取得する（`Format as string` で `format: 'd'`、結果は文字列 `"0"`〜`"6"`。または `dddd` で曜日名）。
2. Condition ノード（**Branch on true/false モード**、`./repository/docs/docs/en/workflow/nodes/condition.md` 参照）で、上記の曜日が「金曜（`d`=='5'、または `dddd`=='Friday'）」かどうかを判定。
   - true 分岐: Date Calculation ノードで日時フィールドに **+3日**（Add a period of time、unit=Day、amount=3）。
   - false 分岐: Date Calculation ノードで日時フィールドに **+1日**（同、amount=1）。
3. 分岐後は自動的に合流するので、続けて対象コレクションの日時フィールドを更新する Update ノードなどにつなげる。

なお、スケジュールトリガー自体（`./repository/docs/docs/en/workflow/triggers/schedule.md`）の Custom time モードには cron ライクな「高度モード」があり、曜日を指定した定期実行自体は可能だが、これは「ワークフローが実行されるタイミング」を制御するものであり、「日時フィールドの値を平日分だけずらす」という要件（金曜のみ+3日）はワークフロー内の Date Calculation + Condition ノードの組み合わせで別途実装する必要がある。

### 未解決の疑問点

- `date-calculation.md`（英語・日本語とも）には「Get value」関数の unit=Week の挙動（曜日番号を返す）についての具体例・説明が記載されておらず、ドキュメントの改善余地があると思われる（本調査では指摘のみ）。
- 「Datetime (with time zone)」フィールドのドキュメント自体が非常に薄く（Example が未記載）、内部的なタイムゾーン保存・変換の仕様の詳細はドキュメントからは確認できなかった。

詳細な調査過程・引用箇所は `reports/0006_workflow-datetime-day-of-week/log.md` を参照。

---

## 完了サマリー

- **完了日時**: 2026-07-13T10:51:45+09:00
- **ログファイル**: `reports/0006_workflow-datetime-day-of-week/log.md`
- **備考**: 曜日取得は可能。「Format as string」関数（Day.js フォーマットトークン `d`/`dddd`）の利用を推奨。「Get value」関数の unit=Week はラベルと実装（曜日番号を返す）にズレがあるため注意。
