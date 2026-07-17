# 0004_workflow-pre-action-event-source 調査ログ

## 基本情報

- **タスクファイル**: reports/0004_workflow-pre-action-event-source/0004_workflow-pre-action-event-source.md
- **開始日時**: 2026-06-12T18:13:07+09:00
- **完了日時**: TBD

## タスク概要

# ワークフロー Pre Action Event の起動元区別
## 知りたいこと

ワークフローにおいてPre Action Eventで起動するとき、NocoBaseのフォーム操作から起動したのか、API Callから起動したのかなどを区別できますか？

## 目的

操作元を特定できるのなら、それを設計に反映したい。

## 調査ファイル一覧

- `repository/docs/docs/en/workflow/triggers/pre-action.md`
- `repository/docs/docs/ja/workflow/triggers/pre-action.md`
- `repository/docs/docs/en/workflow/triggers/post-action.md`
- `repository/docs/docs/ja/workflow/triggers/index.md`
- `repository/docs/docs/en/workflow/triggers/index.md`
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/server/RequestInterceptionTrigger.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/server/Plugin.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/common/constants.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/client/RequestInterceptionTrigger.tsx`
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/server/__tests__/trigger.test.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow/src/server/Processor.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow/src/client/hooks/useTriggerWorkflowActionProps.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow/src/client/flows/triggerWorkflows.tsx`
- `repository/packages/plugins/@nocobase/plugin-workflow-action-trigger/src/server/ActionTrigger.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow-javascript/src/server/ScriptInstruction.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow-javascript/src/server/Vm.js`
- `repository/packages/plugins/@nocobase/plugin-workflow/src/server/development/trigger.md` (en)

## 調査結果

### `repository/docs/docs/en/workflow/triggers/pre-action.md` (および対応する `repository/docs/docs/ja/workflow/triggers/pre-action.md`)

このファイルは "Before Action Event" (操作前イベント、Pre Action Event) の公式ドキュメント。pkg ヘッダ: `@nocobase/plugin-workflow-request-interceptor`

重要な記述（抜粋・要約せず原文ベースで記録）:

- "The Before Action Event plugin provides an interception mechanism for actions, which can be triggered after a request for create, update, or delete action is submitted but before it is processed."
- "If an "End workflow" node is executed in the triggered workflow, or if any other node fails to execute (due to an error or other incompletion), the form action will be intercepted."
- トリガー設定: collection選択、インターセプトモード (local: このワークフローにバインドされたボタンのみ / global: コレクションの選択アクションすべて)
- "Currently supported action types are "Create", "Update", and "Delete"."
- バインド: "Trigger interception only when a form bound to this workflow is submitted" の場合、フォームのボタンにバインド必要。"The buttons that can be bound to a Before Action Event currently only support "Submit" (or "Save"), "Update data", and "Delete" buttons in create or update forms. The "Trigger workflow" button is not supported (it can only be bound to an "After action event")."
- 利用可能変数テーブル:
  | Action Type \ Variable | "Operator" | "Operator role identifier" | Action parameter: "ID" | Action parameter: "Submitted data object" |
  | Create | ✓ | ✓ | - | ✓ |
  | Update | ✓ | ✓ | ✓ | ✓ |
  | Delete | ✓ | ✓ | ✓ | - |
- "The "Trigger data / Action parameters / Submitted data object" variable in a Before Action Event is not the actual data from the database, but rather the parameters submitted with the action."
- "## External Invocation" セクション:
  "The Before Action Event itself is injected during the request processing phase, so it also supports being triggered via HTTP API calls."
  例:
  ```
  curl -X POST ... "http://localhost:3000/api/posts:create?triggerWorkflows=workflowKey"
  ```
  "After the above call is made, the Before Action Event for the corresponding `posts` collection will be triggered."
  "If the Before Action Event is configured in global mode, you do not need to use the `triggerWorkflows` URL parameter to specify the corresponding workflow when calling the HTTP API. Simply calling the corresponding collection action will trigger it."
  ```
  curl ... "http://localhost:3000/api/posts:create"
  ```
- 日本語版でも同等の内容（「外部からの呼び出し」セクションで HTTP API によるトリガーを明示的にサポートと記述）。

このドキュメントから、フォーム操作と API 呼び出しの両方が同じ Pre Action Event をトリガーしうることが明記されているが、トリガーされたワークフロー内部で「どちらから来たか」を区別するための変数やフラグについては一切言及がない。

### `repository/docs/docs/en/workflow/triggers/post-action.md`

Post-Action Event についても同様に "External Call" セクションが存在:
"At the implementation level, since post-action event handling is at the middleware layer (Koa's middleware), HTTP API calls to NocoBase can also trigger defined post-action events."
"## External Call" で `?triggerWorkflows=...` を使った API 呼び出し例を記載。
"FAQ" では Pre-Action と Post-Action、Collection Event との違いを説明しているが、origin 区別については触れず。

### `repository/docs/docs/en/workflow/triggers/index.md` (および ja 版)

- トリガータイプ一覧に "Before Action" (pre-action), "After Action" (post-action) を明記。
- "For example, when a user submits a form, or when data in a collection changes due to user action or a program call..."
- "Data-related triggers (such as actions, collection events) usually carry trigger context data."

origin 区別の言及なし。

### `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/server/RequestInterceptionTrigger.ts`

Pre Action の実装本体。

主要コード抜粋（詳細に記録）:

```ts
middleware = async (context, next) => {
  const {
    resourceName,
    actionName,
    params: { filterByTk, filter, values, triggerWorkflows = '' },
  } = context.action;

  const dataSourceHeader = context.get('x-data-source') || 'main';
  ...
  const workflows = ... filter by type 'request-interception' and collection

  const globalWorkflows = workflows.filter((item) => item.config.global && item.config.actions?.includes(actionName))...
  const localWorkflows = ... 

  for (const workflow of localWorkflows.concat(globalWorkflows)) {
    if (!workflow.config.global && !triggerWorkflowsMap.has(workflow.key)) {
      continue;
    }

    const processor = await this.workflow.trigger(
      workflow,
      {
        user: context.state.currentUser,
        roleName: context.state.currentRole,
        params: {
          filterByTk,
          filter,
          values,
        },
      },
      { httpContext: context },
    );
    ...
  }

  await next();
};
```

- trigger に渡されるデータ: `{ user, roleName, params: { filterByTk, filter, values } }`
- 第3引数 options に `{ httpContext: context }` (koa context 全体) を渡している。
- `triggerWorkflows` は action params から来ており、local workflow のマッチングに使われる（`?triggerWorkflows=...` または同等）。
- `context.action` から resource/action/params を取得（NocoBase の action コンテキスト）。
- httpContext は processor.options 経由で一部 instruction (response-message) が利用。

validateContext と execute メソッドもあり（後者は手動トリガー用?）。

このコードから、サーバー側で受け取る情報は request に含まれる triggerWorkflows param、auth された user/role、submitted values のみで、「UIフォーム経由で setSaveRequestConfig されたものか、手動 curl/API か」の区別フラグは存在しない。

### `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/client/RequestInterceptionTrigger.tsx`

Client 側の Trigger 定義。変数公開と設定 UI を担当。

重要な関数:

```ts
function useVariables(config, options) {
  ...
  const userFields = ... 'User acted', 'Role of user acted'
  const params = [
    {
      label: lang('Parameters'),
      value: 'params',
      children: useActionParamVariables(...)  // filterByTk / values + collection fields
    }
  ];
  return [...userFields, ...params];
}
```

useInitializers で trigger data initializer の dataPath: '$context.params.values'

triggerFieldset でテスト/手動実行時の target/userId/roleName 設定あり。

useVariables が返す内容が、ワークフロー内で Condition ノード等で使える変数として公開されるもの。これに origin/source 関連のものは一切ない。

### `repository/packages/plugins/@nocobase/plugin-workflow/src/server/Processor.ts`

変数スコープの定義（getScope 周辺）:

```ts
const scopes = {
  $context: this.execution.context,   // <-- ここに trigger で渡した {user, roleName, params} が入る
  $jobsMapByNodeKey: this.jobResultsMapByNodeKey,
  $system: systemFns,
  $scopes,
  $env: this.options.plugin.app.environment.getVariables(),
};
```

httpContext は `processor.options.httpContext` に残るが、$context や標準 variable scope には注入されていない。

getParsedValue などもこの scope を使う。

### `repository/packages/plugins/@nocobase/plugin-workflow-javascript/src/server/ScriptInstruction.ts` と `Vm.js`

JS ノードでも:

- 渡されるのは node.config.arguments を getParsedValue で解決したもののみ。
- Vm.js の sandbox: `const context = { ...args, require: ..., console: ... }`
- processor や options.httpContext は一切渡されない。したがって JS コード内でも raw request / headers / trigger origin にアクセスできない（引数として明示的に workflow 変数で渡せない限り）。

### `repository/packages/plugins/@nocobase/plugin-workflow/src/client/flows/triggerWorkflows.tsx` および `useTriggerWorkflowActionProps.ts`

FormSubmitActionModel / UpdateRecordActionModel の registerFlow で:

```ts
handler(ctx, params) {
  const triggerWorkflows = ...;
  ctx.model.setSaveRequestConfig({
    params: { triggerWorkflows },
  });
}
```

これにより、UI のフォーム submit 時に、裏側の collection resource action (e.g. posts:create) のリクエスト params に triggerWorkflows が付与される。

deprecated な専用 "Trigger workflow" アクションでも同様に workflows.trigger resource に triggerWorkflows を渡す。

つまり、UI フォームバインドの効果は「リクエストに triggerWorkflows パラメータを付ける」ことであり、サーバーからは純粋な HTTP リクエストとして見え、curl 等で同じパラメータを付与すれば完全に同一になる。

### `repository/packages/plugins/@nocobase/plugin-workflow-action-trigger/src/server/ActionTrigger.ts`

Post-action も同様に ` { httpContext: context } ` を渡し、trigger data は `{ data, ...userInfo }` の形。origin 区別の仕組みは同じく存在しない。

## 調査アプローチ

1. add-report スキルに従い、まず `ls reports/ 2>/dev/null | grep -E '^[0-9]{4}_' | sort | tail -1` のみで連番計算（0003 の次として 0004 を採用）。他の reports/ 読み込みは行わず、知りたいこと/目的から英語ファイル名タイトル `workflow-pre-action-event-source` を生成、日本語見出し「ワークフロー Pre Action Event の起動元区別」を決定してタスク md を作成。
2. ユーザーが「はい（すぐ実行する）」を選択したため、report スキル手順に従い即時実行開始。log.md をテンプレートに従って初期化（開始日時記録、タスク概要転記、会話内容に add-report 経緯を追記）。
3. ドキュメント調査の起点として `repository/docs/docs/en/workflow/` （および ja 併記対象として ja 版）を list_dir。関連トリガードキュメントを特定: pre-action.md, post-action.md, index.md 等を優先。
4. 内容検索: grep で "pre-action|Before action|request-interceptor" をリポジトリ全体で実行し、関連ロケール・CHANGELOG・ソースパスを特定。特に plugin-workflow-request-interceptor ディレクトリを発見。
5. 主要ドキュメントとソースを read_file で順次精読。pre-action.md の "External Invocation" セクション、RequestInterceptionTrigger.ts の middleware と trigger 呼び出し箇所、client RequestInterceptionTrigger.tsx の useVariables、Processor.ts の $context スコープ定義、triggerWorkflows.tsx の setSaveRequestConfig、ScriptInstruction/Vm.js の sandbox 内容等を重点的に抽出（要約禁止で長文抜粋）。
6. 比較のため post-action 実装（ActionTrigger.ts）や JS ノードの引数渡しも読む。httpContext の使われ方（response-message など）を確認。
7. クライアント側のバインド機構（FormSubmitActionModel.registerFlow での triggerWorkflows 付与）が「UI フォーム submit 時に params を付加する」だけであることを確認。これにより API エミュレーションが可能である点を論理的に導出。
8. ログに段階的に追記（各ファイルごとに「調査結果」サブセクションで発見事実を詳細記述）。調査ファイル一覧を更新。
9. 完了時にタイムスタンプ取得、調査アプローチ・会話内容・判断・問題点を追記。report md へサマリー追記。

検索キーワード例: "pre-action", "Before Action Event", "triggerWorkflows", "httpContext", "request-interception", "setSaveRequestConfig", "$context", "useVariables" など。

## 会話内容

### [2026-06-12T18:13:07+09:00] ユーザー指示（add-report + 即時 report 実行選択）
ユーザーは `/add-report` で以下の調査タスクを作成した：

知りたいこと: ワークフローにおいてPre Action Eventで起動するとき、NocoBaseのフォーム操作から起動したのか、API Callから起動したのかなどを区別できますか？
目的: 操作元を特定できるのなら、それを設計に反映したい。

add-report により reports/0004_workflow-pre-action-event-source/0004_workflow-pre-action-event-source.md が生成された後、AskUserQuestion で「続けて `/report` を実行しますか？」と確認し、ユーザーは「はい（すぐ実行する）」を選択したため、本 report タスクを 0004 番として即時開始。

### [2026-06-12T18:13:07+09:00 〜 18:15:15+09:00] 調査実行中の Claude 内部進行
- スキル指示に従い、連番計算（bash ls のみ使用）、ファイル生成、ask_user_question 実行。
- ユーザ選択「はい」により report 手順開始: タスク md 読込、log.md 作成、関連ドキュメント/ソースの list + read + grep を実施。
- 各 read 結果を基に、log.md へ「調査ファイル一覧」「調査結果（ファイルごと詳細事実）」を追記しながら進行。
- 追加で client バインド機構、Processor スコープ、JS sandbox などを調査し、origin 情報が $context に現れないことを確認。
- 最終化のため完了タイムスタンプ取得、残りセクション追記予定。

（本セッション外の追加会話はなし。すべてのやり取りは上記ツール呼び出しと結果経由。）

## 判断・意思決定

- **ドキュメント優先 + ソース確認**: 公式ドキュメントだけでは「区別できるか」の実装詳細が不明瞭だったため、即座に実装ソース（RequestInterceptionTrigger.ts と client 変数定義、Processor の scope）を必須で読む判断をした。これにより「$context に何が渡るか」「httpContext が workflow 変数として露出するか」が明確になった。
- **エミュレーション可能性の強調**: setSaveRequestConfig による triggerWorkflows 付与が UI 側クライアントの単なるリクエスト装飾であることを発見した時点で、「API 呼び出し側が同一フォーマットを再現可能」と判断。信頼できる区別フラグはサーバー側で追加されていないと結論。
- **JS ノードでも不可**: sandbox が args のみであることを確認したため、「高度なノードを使えば...」という誤った期待を排除。
- **ログ記録方針厳守**: 各ファイルの主要記述・コードを「要約禁止」で長めに引用。CLAUDE.md の「調査結果やログに .../ja も併記」も意識して ja 版パスを記載。
- **調査範囲**: pre-action に集中。post-action や collection は比較・文脈のために最小限読了。実際のデータベース操作や他のプラグイン深掘りは本質問のスコープ外と判断してスキップ。

## 問題・疑問点

- 現在の NocoBase 実装では、Pre Action Event ワークフロー内で操作元（フォーム UI vs 任意 API）を標準的な変数・API で区別する手段が提供されていない。設計で「操作元による分岐」をしたい場合、以下のいずれかが必要になる:
  - カスタムヘッダを UI フォームアクション時のみ付与する拡張（core または自前 action ハンドラ変更）。
  - trigger 実装をフォークして追加コンテキスト（例: `source: 'ui-form' | 'api'`）を注入。
  - httpContext を workflow 変数として公開するような汎用機構の要望/コントリビュート。
- ドキュメントの "External Invocation" が API トリガーを公式サポートとして記載しているため、「UI のみ想定」という前提は崩れやすい。
- 調査中に見つかった httpContext は response 書き戻し専用に近く、読み取り用途のドキュメント例がほとんどない（response-message ノード経由で間接的に触る程度）。
- 未調査: 実際の koa context が action 実行時に持つ追加プロパティ（例: context.action 内部の source 情報や client 識別ヘッダの有無）について、NocoBase core の resourcer/action レイヤーまで深く掘る時間的制約があった。必要なら追加 report で掘る価値あり。

## 完了日時
2026-06-12T18:15:15+09:00
**ログファイル**: `reports/0004_workflow-pre-action-event-source/log.md`
