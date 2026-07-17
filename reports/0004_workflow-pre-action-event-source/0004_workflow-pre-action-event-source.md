# ワークフロー Pre Action Event の起動元区別
## 知りたいこと

ワークフローにおいてPre Action Eventで起動するとき、NocoBaseのフォーム操作から起動したのか、API Callから起動したのかなどを区別できますか？

## 目的

操作元を特定できるのなら、それを設計に反映したい。

## 調査サマリー

NocoBase の Pre Action Event (Before Action Event / 操作前イベント、プラグイン: `@nocobase/plugin-workflow-request-interceptor`) において、ワークフローがトリガーされた時点で「NocoBase のフォーム UI 操作から起動されたのか、直接の API Call から起動されたのか」を区別できるかどうかを、公式ドキュメント（en/ja）と実装ソースを調査した結果、以下の事実が判明した。

- **公式ドキュメントで明記**: pre-action.md の "External Invocation"（日本語版「外部からの呼び出し」）セクションで、HTTP API 呼び出し（`?triggerWorkflows=...` パラメータ使用、または global モード時は直接コレクション action 呼び出し）によるトリガーが**公式にサポート**されている。UI フォーム submit と API の両方が同一の Pre Action Event を起動しうる。
- **ワークフローに渡されるデータ（$context で利用可能な変数）**:
  - Operator (user), Operator role identifier (roleName)
  - Action parameters: filterByTk (ID), values (Submitted data object) など
  - これらは RequestInterceptionTrigger.ts の middleware で `this.workflow.trigger(workflow, { user, roleName, params: { filterByTk, filter, values } }, { httpContext: context })` として渡される。
  - クライアント側の RequestInterceptionTrigger.tsx の `useVariables` で定義され、Condition ノードなどで参照可能。origin / source / trigger origin を表すフィールドは一切存在しない。
- **httpContext の扱い**:
  - Koa context 全体が processor.options.httpContext に渡される（post-action や custom-action も同様）。
  - しかし、Processor.ts の getScope で $context / $system / $env 等に注入されておらず、標準 variable picker からアクセスできない。
  - Response Message ノードなど一部 instruction だけが httpContext を使ってレスポンス書き戻しを行う。
- **JS ノードでもアクセス不可**:
  - ScriptInstruction + Vm.js のサンドボックスは、ノードで明示指定した arguments（getParsedValue で解決された $context 由来の値）のみを受け取る。processor や httpContext、raw headers は渡されない。
- **UI バインドの仕組み**:
  - フォームボタンへのバインド（FormSubmitActionModel / UpdateRecordActionModel）は、内部で `setSaveRequestConfig({ params: { triggerWorkflows } })` を呼び、実際の collection action (e.g. `/api/posts:create`) リクエストに `triggerWorkflows` パラメータを付与するだけ。
  - そのため、同一の `triggerWorkflows=xxx` を付けて curl などから直接呼び出せば、サーバー側（RequestInterceptionTrigger の localWorkflows マッチング含む）で**完全に同じデータ**が届く。区別のためのサーバーサイドフラグや秘密のヘッダ等は実装されていない。
- **結論**: 現在の標準機能では、Pre Action Event 起動時の操作元をワークフロー内で判別することはできない。ログファイルに各ファイルの詳細なコード/記述を記録済み。

**根拠ドキュメント/ソース**（詳細は log.md を参照）:
- `repository/docs/docs/en/workflow/triggers/pre-action.md` (および ja 版)
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/server/RequestInterceptionTrigger.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow-request-interceptor/src/client/RequestInterceptionTrigger.tsx`
- `repository/packages/plugins/@nocobase/plugin-workflow/src/server/Processor.ts`
- `repository/packages/plugins/@nocobase/plugin-workflow/src/client/flows/triggerWorkflows.tsx`

未解決の疑問点:
- カスタム拡張で httpContext や action context を露出させる方法の有無（高度な開発者向け）。
- 将来的に workflow 変数として request メタデータを標準提供する計画の有無（ドキュメント・CHANGELOG では言及なし）。

---

## 完了サマリー

- **完了日時**: 2026-06-12T18:15:15+09:00
- **ログファイル**: `reports/0004_workflow-pre-action-event-source/log.md`
- **備考**: 調査対象は主に Pre Action Event（request-interception トリガー）。Post Action なども類似構造であることを確認したが、質問の焦点に合わせた。git commit は未実行（スキル注意事項遵守）。
