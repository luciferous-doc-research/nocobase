# 調査ログ - ワークフローから承認依頼を投げる方法

## 調査ファイル一覧

英語版ドキュメント：
- `./repository/docs/docs/en/workflow/nodes/request.md` - HTTP Request ノード
- `./repository/docs/docs/en/workflow/advanced/variables.md` - ワークフロー変数の使い方
- `./repository/docs/docs/en/workflow/triggers/schedule.md` - スケジュール実行ワークフロー
- `./repository/docs/docs/en/workflow/nodes/index.md` - ワークフローノード一覧
- `./repository/docs/docs/en/workflow/nodes/subflow.md` - Invoke Workflow（ワークフロー呼び出し）ノード

日本語版ドキュメント：
- `./repository/docs/docs/ja/workflow/nodes/subflow.md` - ワークフローの呼び出しノード

その他参照ドキュメント（前回の調査から）：
- `./repository/docs/docs/en/workflow/triggers/approval.md` - 承認トリガー（API リファレンス含む）
- `./repository/docs/docs/ja/workflow/triggers/approval.md` - 承認トリガー（日本語）

## 調査アプローチ

1. **初期仮説の構築**
   - 前回の調査（0002）から、`/api/approvals:create` という承認申請 API が存在することを把握
   - HTTP Request ノードを使ってこの API を呼び出すことでワークフローから承認依頼を投げられるのではないかという仮説

2. **HTTP Request ノードの確認**
   - `request.md` で HTTP Request ノードの機能を確認
   - JSON 形式でリクエストボディを送信可能、変数も使用可能
   - Authorization ヘッダーなどのカスタムヘッダーも設定可能

3. **スケジュール実行ワークフローの特性確認**
   - `schedule.md` で、スケジュール実行ワークフローの仕組みを確認
   - ユーザーアクションではなく、時間トリガーで自動実行されることを確認
   - ドキュメントにはユーザーコンテキストについての明示的な記載がない

4. **ワークフロー内での操作方法の調査**
   - `nodes/index.md` で全ノード一覧を確認
   - **重要な発見**：「Invoke Workflow」ノード（subflow）の存在を発見
   - Collection Actions カテゴリにあるノード（Create Data、Update Data など）は認証不要で動作

5. **Invoke Workflow ノードの詳細調査**
   - `subflow.md` で Invoke Workflow の詳細を確認
   - ワークフロー内から別のワークフローを呼び出せることを確認
   - トリガー変数を設定して、呼び出し先ワークフローに入力を渡せることを確認
   - 無効なワークフローもサブワークフローとして呼び出し可能

## 調査結果

### 1. ワークフローから承認依頼を投げることは可能です

**結論：Yes** - 以下の2つの方法で実装可能です。

### 2. 実装方法の詳細

#### 方法1：Invoke Workflow ノードを使う（推奨）

**基本的な流れ：**

英語版 `./repository/docs/docs/en/workflow/nodes/subflow.md` の line 5-24 に記載：
> "Used to invoke other workflows from within a workflow. You can use variables from the current workflow as input for the sub-workflow, and use the sub-workflow's output as variables in the current workflow for use in subsequent nodes."

日本語版 `./repository/docs/docs/ja/workflow/nodes/subflow.md` の line 13 に同等の説明。

**実装ステップ：**

1. スケジュール実行ワークフロー内に「Invoke Workflow」ノードを追加（英語版 line 28-31）
2. 呼び出すワークフローを選択：承認ワークフロー（Approval トリガーを持つワークフロー）
3. トリガー変数を設定：承認ワークフローのトリガーが要求する入力を設定（英語版 line 49-57）
   - 承認対象のコレクションデータなど

**利点：**
- ワークフロー間の連携が明示的で管理しやすい
- ユーザーコンテキストの問題が発生しにくい
- ネイティブな実装方法

**制限事項：**
英語版 line 44-47：
> "* Disabled workflows can also be invoked as sub-workflows.
> * When the current workflow is in synchronous mode, it can only invoke sub-workflows that are also in synchronous mode."

同期/非同期モードの一致が必要

**出力の利用：**
英語版 line 64-67：
> "Back in the main workflow, in other nodes below the Invoke Workflow node, when you want to use the output value of the sub-workflow, you can select the result of the Invoke Workflow node."

サブワークフローの出力は「Workflow Output」ノードで設定し、メインワークフローで利用可能

#### 方法2：HTTP Request ノードで API を直接呼び出す

**基本的な流れ：**

英語版 `./repository/docs/docs/en/workflow/nodes/request.md` の line 5-11 に記載：
> "When you need to interact with another web system, you can use the HTTP Request node. When executed, this node sends an HTTP request to the specified address according to its configuration."

**実装内容：**

1. HTTP Request ノードを追加
2. メソッド：POST
3. URL：`http://localhost:3000/api/approvals:create`（または実際のサーバーアドレス）
4. Content-Type：`application/json`
5. リクエストボディ：

前回の調査から（0002_approval-plugin-usage/log.md）、以下の情報を活用：

```json
{
  "collectionName": "コレクション名",
  "workflowId": ワークフロー ID,
  "data": { "フィールド名": "値" },
  "status": 1
}
```

6. Request Headers に認証情報を設定：
   - `Authorization: Bearer <your token>`
   - `X-Role: <ロール名>`

**利点：**
- より細粒度の制御が可能
- 既存の API エンドポイントを直接利用

**課題（重要）：**

英語版 `./repository/docs/docs/en/workflow/triggers/approval.md` の line 174-176 に記載：
> "Since external calls also need to be based on user identity, when calling via HTTP API, just like requests sent from the regular interface, authentication information must be provided, including the `Authorization` header or the `token` parameter (the token obtained upon login), and the `X-Role` header (the user's current role name)."

日本語版 `./repository/docs/docs/ja/workflow/triggers/approval.md` の line 132-134 に同等の記載。

**スケジュール実行時の課題：**
- 認証トークンをどこから取得するか
- 承認申請者（X-Role）をどのように指定するか
- これらはドキュメントに明示的な方法が記載されていない

### 3. スケジュール実行ワークフロー固有の考慮事項

英語版 `./repository/docs/docs/en/workflow/triggers/schedule.md` の line 10 に記載：
> "When the system reaches the time point (accurate to the second) that meets the configured trigger conditions, the corresponding workflow will be triggered."

日本語版でも同等の記載。

**ユーザーコンテキストについて：**
- スケジュール実行は自動実行のため、特定のユーザーアクションではない
- ドキュメントには、スケジュール実行時のユーザーコンテキストについて明示的な記載がない
- Invoke Workflow を使う場合、呼び出し先の承認ワークフローで「申請者」の設定が必要
  - トリガー変数で申請者情報を指定できるか、ドキュメント上は明確でない
  - 実装時には、この点の確認が必要

### 4. 実装可能性の評価

**Invoke Workflow ノードを使う方法（推奨）：**
- 実装可能性：高
- 複雑性：低
- ドキュメント上の根拠：明確
- 推奨理由：ネイティブな実装で、ワークフロー間連携が明示的

**HTTP Request ノードを使う方法：**
- 実装可能性：中（認証情報の取得が課題）
- 複雑性：中
- ドキュメント上の根拠：一部未記載
- 制限：スケジュール実行時の認証トークン取得方法がドキュメント上に記載されていない

## 会話内容

- 前回の 0002 タスク「承認プラグインについて詳しく知りたい」の追加質問として実施
- ユーザーの具体的な要件：棚卸し作業の一環で、スケジュール実行されたワークフローから承認依頼を投げたい

## 判断・意思決定

1. **Invoke Workflow ノードを優先推奨**
   - ドキュメント上で実装方法が明確
   - ワークフロー内部の操作で認証問題が発生しない
   - スケジュール実行ワークフロー → Invoke Workflow → 承認ワークフロー の流れが最も自然

2. **HTTP Request ノードはセカンダリ選択肢**
   - 認証トークン取得方法の不明確さが実装上の障害
   - NocoBase のシステム内部で使用するシステムレベルのトークンが必要な可能性
   - より詳しい調査や実装試験が必要

## 問題・疑問点

1. **スケジュール実行時の申請者（Initiator）の設定方法**
   - Invoke Workflow で承認ワークフローを呼び出す場合、誰が申請者となるのか
   - ドキュメント上に明示的な記載がない
   - 実装時には実際に試験して確認する必要がある

2. **HTTP API 経由での認証トークン**
   - スケジュール実行ワークフロー内から、有効な認証トークンをどのように取得するか
   - NocoBase 内部用のシステムレベルトークンが存在するのか
   - ドキュメント上に記載されていない

3. **トリガー変数設定の詳細**
   - Invoke Workflow で承認ワークフローのトリガー変数を設定する際の具体的な方法
   - ドキュメントは「ワークフロー」一般について説明しているが、Approval トリガー固有の情報がない
