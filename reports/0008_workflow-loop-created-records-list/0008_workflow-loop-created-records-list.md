# ワークフローのループ内で作成したレコード一覧の取得

## 知りたいこと

ワークフローのループにおいて、ループの中でレコードを作成したとき、ループを抜けたあと作成したレコードの一覧を取得することはできますか？

## 目的

ループの内側で作成したレコードの一覧が欲しい。取得できないのなら作成後にクエリ レコードしないといけないから。

## 調査サマリー

**結論: 標準機能としては「取得できない」。ただし Variable ノードを使った回避策がある。**

- Loop ノード（`./repository/docs/docs/en/workflow/nodes/loop.md` / `./repository/docs/docs/ja/workflow/nodes/loop.md`）の仕様上、ループ内で使えるローカル変数（反復対象データ、インデックス）の**スコープはループ内に限定される**。ループを抜けた後続ノードから、ループ内で実行した Create Record ノードの結果（各回で作成したレコード）を直接参照する仕組みは存在しない。
- 実行履歴（`./repository/docs/docs/en/workflow/advanced/executions.md`）ではループノードは「複数の実行結果」を持つとされているが、これは**実行履歴閲覧UI上**の話であり、後続ノードが変数として参照できる「作成レコードの配列」ではない。
- Create Record ノード（`./repository/docs/docs/en/workflow/nodes/create.md` / `./repository/docs/docs/ja/workflow/nodes/create.md`）の結果データは、実行1回分（＝ループの1反復分）の作成レコードのみを保持し、反復ごとに上書きされる。
- **回避策**: Variable ノード（`./repository/docs/docs/en/workflow/nodes/variable.md` / `./repository/docs/docs/ja/workflow/nodes/variable.md`）の公式 Example では、ループの**前**に変数を宣言し、ループの**内側**で「計算ノードなどで新しい値と既存値をマージ（reduce/concat 相当）→ Variable ノード（代入モード）で更新」を行うことで、ループを**抜けた後**もその変数（蓄積された値）を後続ノードから参照できる、というパターンが明記されている（文字列連結の例だが、配列への蓄積にも応用可能な設計）。
  - この方法を使えば、ループ内で Create Record ノードの実行結果（作成されたレコードのID等）を毎回 Variable ノードに追記していくことで、ループを抜けた後に「作成したレコードの一覧」を変数として保持でき、目的にある「作成後にクエリレコードし直す」手間を回避できる可能性が高い。
  - ただし、配列に対する具体的な push 操作のUI設定例（Calculationノードでどの関数を使うか等）はドキュメント上に見当たらず未確認。詳細はログの「問題・疑問点」を参照。

詳細な調査内容・引用箇所はログファイル（`reports/0008_workflow-loop-created-records-list/log.md`）を参照。

---

## 完了サマリー

- **完了日時**: 2026-07-13T14:02:24+09:00
- **ログファイル**: `reports/0008_workflow-loop-created-records-list/log.md`
- **備考**: 標準機能としては不可。Variableノードによる蓄積パターン（reduce/concat）が公式ドキュメントのExampleとして提示されている。配列へのpush操作の具体的なUI設定手順はドキュメント上未確認。
