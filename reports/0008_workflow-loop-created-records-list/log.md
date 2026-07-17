# 0008_workflow-loop-created-records-list 調査ログ

## 基本情報

- **タスクファイル**: reports/0008_workflow-loop-created-records-list/0008_workflow-loop-created-records-list.md
- **開始日時**: 2026-07-13T13:59:55+09:00
- **完了日時**: 2026-07-13T14:02:24+09:00

## タスク概要

ワークフローのループにおいて、ループの中でレコードを作成したとき、ループを抜けたあと作成したレコードの一覧を取得することはできますか？

目的: ループの内側で作成したレコードの一覧が欲しい。取得できないのなら作成後にクエリ レコードしないといけないから。

## 調査ファイル一覧

- `./repository/docs/docs/en/workflow/nodes/loop.md` （日本語: `./repository/docs/docs/ja/workflow/nodes/loop.md`）
- `./repository/docs/docs/en/plugins/@nocobase/plugin-workflow-loop/index.md`
- `./repository/docs/docs/en/workflow/advanced/variables.md` （日本語: `./repository/docs/docs/ja/workflow/advanced/variables.md`）
- `./repository/docs/docs/en/workflow/advanced/executions.md` （日本語: `./repository/docs/docs/ja/workflow/advanced/executions.md`）
- `./repository/docs/docs/en/workflow/advanced/options.md` （日本語: `./repository/docs/docs/ja/workflow/advanced/options.md`）
- `./repository/docs/docs/en/workflow/nodes/variable.md` （日本語: `./repository/docs/docs/ja/workflow/nodes/variable.md`）
- `./repository/docs/docs/en/workflow/nodes/create.md` （日本語: `./repository/docs/docs/ja/workflow/nodes/create.md`）
- `./repository/docs/docs/en/workflow/nodes/query.md`
- `./repository/docs/docs/en/workflow/nodes/json-query.md`
- `./repository/docs/docs/en/workflow/nodes/index.md`

## 調査結果

### `./repository/docs/docs/en/workflow/nodes/loop.md`

- Loop ノードの概要（L9）:「for/while/forEach のような構文構造に相当する」と説明。
- L23: 「ループノードを作成すると、ループ内にブランチが生成される。このブランチ内にはノードをいくつでも追加できる。これらのノードは、ワークフローコンテキストの変数だけでなく、ループコンテキストのローカル変数（ループ対象コレクションの反復中のデータオブジェクトや、ループカウントのインデックスなど）も使用できる。ローカル変数のスコープはループ内に限定される。ネストしたループがある場合、各階層の特定のループのローカル変数を使用できる。」→ **ループ内で使えるローカル変数のスコープはループ内に限定される**、という明記がある。
- L31-43: Loop Object（ループ対象）の説明。Array/Number/String/Other の4種の扱い方の説明のみで、ループの実行結果（各回で生成したレコード等）を集約する仕組みについては記載なし。
- L45-59: Loop Condition（v1.4.0-beta以降）。while/do-while 相当の条件評価、break/continueに相当する「条件未達成時の挙動」の説明。
- L61-69: ループ内ノードのエラー処理（v1.4.0-beta以降）。Exit workflow / Exit loop and continue workflow / Continue to the next loop object の3種。
- L71-82: 環境変数 `WORKFLOW_LOOP_LIMIT` によるループ回数上限の説明。
- L84-149: 発注時の在庫チェックのサンプル（Products/Order Details/Orders の3コレクション、ループノードでOrder Detailsを反復し、Conditionノードで在庫を判定し、Update Recordノードで更新するフロー）。この例では「作成」ではなく「更新」だが、ループ内で処理した結果を集約して外部で使うという構成の記載はない。
- **結論**: このドキュメント内には、ループを抜けた後にループ内で作成された全レコードの一覧を自動的に取得できる仕組みについての記載は一切ない。

### `./repository/docs/docs/en/plugins/@nocobase/plugin-workflow-loop/index.md`

- frontmatter のみの短いページで、実質的な内容は無し（Loop ノードの説明は `workflow/nodes/loop.md` が本体）。

### `./repository/docs/docs/en/workflow/advanced/variables.md`

- L7-12: ワークフローで使える変数の分類（Trigger context data / Upstream node data / Local variables / System variables）。
  - L11: 「Local variables: ノードが特定のブランチ構造内にある場合、そのブランチ内の特定のローカル変数を使用できる。例えば、ループ構造では各反復のデータオブジェクトが使える」。
- L26-53: 変数のデータ構造の説明。クエリノードが複数件返す場合、ノード結果は複数行を含む配列になる例（L32-45）と、`Node data/Query node/Title` のようにフィールドを選択すると値の配列 `["Title 1", "Title 2"]` にマッピングされる例（L47-51）。
  - これはクエリノード（複数レコード取得）の結果構造の説明であり、ループノードの実行結果を配列として扱えるという記載ではない。
- **結論**: ループノード自体が「各回で作成されたレコードの配列」を出力するという仕組みへの言及は無い。

### `./repository/docs/docs/en/workflow/advanced/executions.md`

- L25: 「In some advanced workflows and nodes, a node may have multiple results, such as the result of a loop node」。実行履歴の詳細画面で、ループノードは複数の実行結果（反復ごとの結果）を持つ、という説明。
- 続く画像キャプション「Node results from multiple executions」（L29）は、実行履歴UI上でループの各回の結果を確認できることを示しているだけで、これはあくまで**実行履歴の閲覧UI上の話**であり、後続ノードから変数として参照できる「配列としての集約結果」の話ではない。
- L54: 「When a node is in a branch flow (parallel branch, condition, loop, etc.), the final state produced by the node's execution will be handled by the node that initiated the branch, and this determines the flow of the entire workflow.」→ ブランチ内ノードの最終状態はブランチを開始したノード（ここではLoopノード）が処理し、それがワークフロー全体のフローを左右する、という説明。ここでも「生成されたレコードの一覧」を後続ノードに渡す仕組みには触れていない。

### `./repository/docs/docs/en/workflow/advanced/options.md`

- L36: ループノードのタイムアウトに関する記載のみ。今回の調査に直接関係する情報はなし。

### `./repository/docs/docs/en/workflow/nodes/variable.md`

- L9: Variable ノードの導入。「フロー内で変数を宣言したり、宣言済みの変数に値を代入したりできる。これは通常、フロー内で一時データを保存するために使用される。」
- L23-32: モード選択。「Declare a new variable（新規宣言）」と「Assign to an existing variable（既存変数への代入）」の2モード。
- L40-53: Value（値）。宣言モードでは初期値、代入モードでは既存の宣言済み変数の値を新しい値に更新する、という説明。「Subsequent uses will retrieve this new value.」（以降の使用では新しい値が取得される）
- **L64-104: Example（最重要）**
  - L66: 「A more useful scenario for the variable node is in branches, where new values are calculated or merged with previous values (similar to reduce/concat in programming), and then used after the branch ends.」→ **ブランチ（ループ含む）内で値を計算・以前の値とマージ（reduce/concatに類似）し、ブランチ終了後に使用する、という用途が Variable ノードの主要なユースケースとして明記されている。**
  - 具体例: Article 更新トリガー→ Author 関連データをプリロード → Variable ノード（宛先文字列を格納、初期値は空など）を宣言 → Loop ノードで Article の著者を反復 → ループ内で Calculation ノードにより現在の著者と既存の宛先文字列を連結 → その後 Variable ノード（代入モード、対象は最初に宣言した宛先変数、値は Calculation ノードの結果）で更新。
  - L98: 「This way, after the loop branch finishes, the recipient variable will store the recipient string of all the article's authors. Then, after the loop, you can use an HTTP Request node...」→ **ループが終了した後、Variable ノードに蓄積された値（全著者の宛先文字列）をループ外の後続ノードで使用できる、という明確な実例。**
- **結論**: NocoBase ではループノード自体が「反復ごとに作成したレコードの配列」を自動的に外部へ出力する機能は無いが、**ループ開始前に配列型（または連結対象の文字列など）の Variable ノードを宣言しておき、ループ内で毎回その配列に要素を追加（またはconcat）するよう Variable ノード（代入モード）を組み込むことで、ループを抜けた後もその配列を変数として参照できる**、という設計パターンが公式ドキュメントの Example として提示されている。

### `./repository/docs/docs/en/workflow/nodes/create.md`

- L3: Create Record ノードは「新しいレコードをコレクションに追加するために使用する」。
- L21-23: Collection（対象コレクション選択）。
- L25-31: Field Values（フィールド値の割当て）。「Data created by the "Create Record" node in a workflow does not automatically handle user data like "Created by" and "Last modified by". You need to configure the values for these fields yourself as needed.」（作成日時・作成者などは自動処理されないため手動設定が必要、という注意書き）。
- L33-35: Preload association data（関連データのプリロード）。「関連フィールドを新規レコードのフィールドに含み、後続のワークフローステップで対応する関連データを使用したい場合、プリロード設定で対応する関連フィールドをチェックできる。これにより、新規レコード作成後、対応する関連データが自動的にロードされ、ノードの結果データに一緒に格納される。」
- L37-49: サンプル（Postsコレクションのレコード作成・更新時にPost Versionsレコードを自動作成し変更履歴を記録する）。
- **結論**: Create Record ノードの「ノード結果（node result）」は、そのノードが実行された1回分の作成レコード（+プリロードした関連データ）のみを保持する。ループ内で複数回実行された場合、この結果はループの反復ごとに上書きされる形になり、ループを抜けた後に「これまでに作成した全レコードの配列」を直接参照できる仕組みは、このノード自体には無い。

### `./repository/docs/docs/en/workflow/nodes/query.md`

- L30: 「Multiple Records: The result will be an array containing records that match the conditions. If no records match, it will be an empty array. You can process them one by one using a Loop node.」→ クエリノードで複数レコードを取得した場合の結果は配列になり、それをLoopノードで1件ずつ処理できる、という説明。これはループの「入力」側の話であり、ループの「出力（作成したレコードの集約）」の話ではない。

### `./repository/docs/docs/en/workflow/nodes/json-query.md`

- L144: 「Then, loop through the resulting order array to update the total price of the orders.」JSON Query ノードの結果配列をループで処理する例。今回の疑問（ループ内で作成したレコードの集約取得）には直接関係しない。

### `./repository/docs/docs/en/workflow/nodes/index.md`

- L16: ノード一覧における Loop ノードへのリンクのみ。

## 調査アプローチ

1. まず `./repository/docs/docs/en` 配下で "loop" を含むファイル名を検索し、`workflow/nodes/loop.md` と `plugins/@nocobase/plugin-workflow-loop/index.md` を特定した。
2. `loop.md` を読み、ループノードの仕様（ローカル変数のスコープがループ内に限定される旨）を確認。ここで「ループの結果を外部で集約する」機能への言及がないことを確認した。
3. `grep -rn "loop"` で `workflow/` 配下の関連ファイル（`advanced/executions.md`, `advanced/options.md`, `advanced/variables.md`, `nodes/variable.md`, `nodes/json-query.md`, `nodes/index.md`, `nodes/query.md`, `development/api.md`, `triggers/pre-action.md`）を洗い出した。
4. `advanced/variables.md` で変数の分類・スコープの説明を確認。「Local variables はブランチ内に限定される」という記述を再確認。
5. `advanced/executions.md` で「ループノードは複数の実行結果を持つ」という記述（L25）を発見したが、これは実行履歴UIでの表示の話であり、後続ノードから変数として参照できることを意味しないと判断した。
6. `nodes/variable.md` を読み、Variable ノードの Example セクション（L64-104）で「ループ内で reduce/concat 的に値を蓄積し、ループ終了後に使用する」という設計パターンが公式に提示されていることを発見。これが本調査の核心的な回答になると判断した。
7. `nodes/create.md` を読み、Create Record ノードの結果データが1回分の作成レコードのみであることを確認。ループ内で複数回実行された場合、この結果はループの反復ごとに上書きされ、ループ外から直接「作成した全レコードの配列」として参照する仕組みはノード自体には無いことを確認した。
8. `nodes/query.md`, `nodes/json-query.md` も確認したが、いずれもループの「入力」側（既存データをループで処理する）の話であり、「ループの出力を集約する」話ではないことを確認した。
9. 日本語版ドキュメントの対応パス（`./repository/docs/docs/ja/...`）の存在を `ls` で確認した。

## 会話内容

### [13:57] ユーザー指示（/add-report 実行時）
知りたいこと: ワークフローのループにおいて、ループの中でレコードを作成したとき、ループを抜けたあと作成したレコードの一覧を取得することはできますか？
目的: ループの内側で作成したレコードの一覧が欲しい。取得できないのなら作成後にクエリ レコードしないといけないから。

### [13:59] Claude 対応
`/add-report` スキルに従い、`reports/0008_workflow-loop-created-records-list/0008_workflow-loop-created-records-list.md` を作成。ユーザーに続けて `/report` を実行するか確認し、「はい」を選択したため `/report` スキルを起動（args: "0008"）。

### [13:59〜] Claude 対応（調査実施）
上記「調査ファイル一覧」「調査結果」の通り、`repository/docs/docs/en/workflow/` 配下のLoopノード・Variableノード・Create Recordノード関連ドキュメントを調査。結論として、ループノード自体には反復ごとの作成レコードを自動集約する機能はないが、Variableノードを使った accumulator パターン（`nodes/variable.md` の Example）が公式に提示されていることを確認した。

## 判断・意思決定

- 「ループを抜けた後に作成レコード一覧を取得できるか」という問いに対して、Loopノード自体の仕様書（`loop.md`）にはその機能の記載がないため、「標準機能としては存在しない」と判断した。
- ただし `nodes/variable.md` の Example が「ループ内でVariableノードにreduce/concatし、ループ後に使用する」という代替パターンを明確に示しているため、これを「取得できないなら作成後にクエリしないといけない」というユーザーの目的に対する代替解として提示することにした。
- Create Recordノードの結果データ構造（`create.md`）から、ループ内でCreate Recordノードを直接後続の main フローで参照することはできない（スコープの限定、かつ反復ごとに上書きされる）ことを補足情報として整理した。

## 問題・疑問点

- Variableノードの「配列に要素を追加（push）」という操作が具体的にどのような設定（Calculationノードでの配列結合の実装方法など）になるかは、`variable.md` の文字列連結（`concat`）の例からの類推であり、配列型変数に対する具体的なpush操作のUIキャプチャや詳細な設定例はドキュメント上に見当たらなかった。この点は、実際に配列で試す場合はCalculationノード側でどのような関数（例: JSONオブジェクトのマージ、配列結合など）が利用可能かを別途確認する必要がある。
- ループノード自体が「反復結果の配列」を持つ内部データ構造を持っているかどうか（実行履歴UI上で複数結果として表示される仕組みの内部実装）については、ドキュメントレベルでは確認できず、これ以上の言及は見当たらなかった。
