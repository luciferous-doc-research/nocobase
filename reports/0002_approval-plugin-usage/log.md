# 調査ログ - 承認プラグインについて詳しく知りたい

## 調査ファイル一覧

英語版ドキュメント：
- `./repository/docs/docs/en/workflow/triggers/approval.md` - 承認トリガーについて
- `./repository/docs/docs/en/workflow/nodes/approval.md` - 承認ノードについて
- `./repository/docs/docs/en/workflow/index.md` - ワークフロー全体の概要
- `./repository/docs/docs/en/workflow/getting-started.md` - ワークフロー入門

日本語版ドキュメント：
- `./repository/docs/docs/ja/workflow/triggers/approval.md` - 承認トリガーについて
- `./repository/docs/docs/ja/workflow/nodes/approval.md` - 承認ノードについて

## 調査アプローチ

1. まず、承認プラグイン関連のドキュメントを検索するために `find` と `grep` を実行
   - キーワード：「approval」「approve」
   - 結果：20 個のマッチングファイルを発見

2. 中でも最も関連性が高いファイルを優先的に調査：
   - `workflow/triggers/approval.md` - 承認トリガーの詳細説明
   - `workflow/nodes/approval.md` - 承認ノードの詳細説明

3. ワークフロー全体の理解のため、以下のドキュメントを補助的に確認：
   - `workflow/index.md` - ワークフローシステム全体の概要
   - `workflow/getting-started.md` - ワークフロー実装の基本手順

4. 日本語版ドキュメントでも同じ内容を確認（ファイルパスの併記が必要なため）

## 調査結果

### 1. 承認プラグイン（Approval Plugin）について

**パッケージ情報：**
- パッケージ名：`@nocobase/plugin-workflow-approval`
- NocoBase v2.0+ で利用可能
- ワークフローシステムに統合された機能

**何ができるのか（用途）：**

承認プラグインは、人手による承認・却下・差し戻しなどの決定を要するビジネスプロセスを管理するためのシステムです。以下のようなシナリオに対応：
- 休暇申請
- 経費精算承認
- 原材料購入承認
- その他のオフィスオートメーション関連の承認プロセス

英語ドキュメント (`./repository/docs/docs/en/workflow/triggers/approval.md`, line 9) に記載：
> "Approval is a process type specifically designed for human-initiated and human-processed tasks to decide the status of relevant data."

日本語ドキュメント (`./repository/docs/docs/ja/workflow/triggers/approval.md`, line 12-13) に記載：
> 「承認」は、人の手によって開始され、人の手によって処理されることで、関連データのステータスを決定するプロセスの一種です。これは通常、オフィスオートメーションやその他の手動による意思決定業務のプロセス管理に利用されます。

### 2. どう使えばいいのか（具体的な使い方）

#### 2.1 ワークフローの作成方法

以下の 3 ステップで承認ワークフローを作成します（英語版 `./repository/docs/docs/en/workflow/triggers/approval.md`, line 13-32）：

1. **ワークフロー管理ページにアクセス**
   - プラグイン設定メニュー → ワークフロー管理
   
2. **新しいワークフローを作成**
   - 「Add New」ボタンをクリック
   - 「Approval」タイプを選択
   
3. **トリガーを設定**
   - トリガーカードをクリック
   - 承認対象となるコレクションを選択
   - 承認の初期フォームを設定

#### 2.2 承認ノードの設定

英語版 `./repository/docs/docs/en/workflow/nodes/approval.md` の詳細説明より、以下を設定可能：

**承認モード（Pass Mode）：**
- 直通モード（Pass-through Mode）：承認可/不可で分岐、シンプルなプロセス向け
- 分岐モード（Branch Mode）：承認/却下/差し戻しで異なる処理分岐を実行、複雑なロジック向け

（英語版 line 25-47 に詳細な説明）
（日本語版 `./repository/docs/docs/ja/workflow/nodes/approval.md`, line 29-43 に日本語説明）

**承認者の設定：**
- 静的選択：ユーザーリストから直接選択
- 動的選択：変数またはクエリ条件に基づいて動的に指定

（英語版 line 49-57）
（日本語版 line 45-55）

**承認方式（Agreement Mode）：**
複数の承認者がいる場合の決定方法 3 種類（英語版 line 64-68）：
- **Anyone**：誰か 1 人が承認すればOK。全員が却下してはじめて却下
- **Countersign**：全員が承認してはじめてOK。誰か 1 人が却下すれば却下
- **Vote**：設定された比率以上の承認で OK

（日本語版 line 63-65：「または承認（或签）」「全員承認（会签）」「投票」）

**処理順序（Processing Order）：**
- **Parallel**：順序関係なく誰でも処理可能
- **Sequential**：設定順に順番に処理

（英語版 line 71-78, 日本語版 line 72-76）

**承認者インターフェース設定：**
承認者が見るUIを自由にカスタマイズ可能：
- 詳細ブロック：申請内容の表示
- アクションフォーム：承認/却下/差し戻しなどのボタン
- フォーム要素：承認時に変更可能なフィールドを設定

（英語版 line 88-146, 日本語版 line 84-134）

### 3. 承認依頼を投げる方法（申請方法）

#### 3.1 UI 経由での申請

**方法1：データ作成フォームから**

英語版 `./repository/docs/docs/en/workflow/triggers/approval.md` の line 99-107 に記載：

1. コレクションのデータ作成/編集フォームの「Submit」（または「Save」）ボタンに承認ワークフローをバインド
2. ユーザーがフォーム送信時に自動的に承認ワークフローがトリガー
3. 送信されたデータはコレクション に保存されると同時に、承認フロー内にスナップショット保存

日本語版 `./repository/docs/docs/ja/workflow/triggers/approval.md` の line 27-37 に同じ説明

**方法2：承認センター（Approval Center Block）から直接申請**

英語版 line 129-132 に記載：
> "After adding a form block, just like in a regular form configuration interface, you can add field components from the corresponding collection and arrange them as needed..."

日本語版 line 95 に記載：
> 「新しい承認を直接申請」

#### 3.2 HTTP API 経由での申請

英語版 `./repository/docs/docs/en/workflow/triggers/approval.md` の line 150-172 に詳細記載。

**パターン1：コレクションデータ作成APIから**

```bash
curl -X POST -H 'Authorization: Bearer <your token>' -H 'X-Role: <roleName>' -d \
  '{
    "title": "Hello, world!",
    "content": "This is a test post."
  }'
  "http://localhost:3000/api/posts:create?triggerWorkflows=workflowKey"
```

注記：
- `triggerWorkflows` に承認ワークフローのキー（workflow key）を指定
- 複数の場合はカンマ区切り
- 認証情報（Authorization ヘッダーまたは token パラメータ）必須
- ロール（X-Role ヘッダー）必須

（日本語版 line 113-134 に同等の説明）

**パターン2：承認センターAPI から（Approval Center 方式）**

```bash
curl -X POST -H 'Authorization: Bearer <your token>' -H 'X-Role: <roleName>' -d \
  '{
    "collectionName": "<collection name>",
    "workflowId": <workflow id>,
    "data": { "<field>": "<value>" },
    "status": <initial approval status>,
  }'
  "http://localhost:3000/api/approvals:create"
```

パラメータ：
- `collectionName`：対象コレクション名（必須）
- `workflowId`：承認ワークフローID（必須）
- `data`：コレクションのフィールドデータ（必須）
- `status`：初期ステータス（必須）
  - `0`：下書き（保存のみ）
  - `1`：承認申請（申請者が申請リクエストを送信）

（英語版 line 198-218, 日本語版 line 156-176）

### 4. 投げられた依頼はどこから見れるのか

#### 4.1 To-Do Center

英語版 `./repository/docs/docs/en/workflow/triggers/approval.md` の line 113-146 に記載。

**To-Do Center の場所：**
画面上部のツールバーにある To-Do Center にアクセス（line 115）
> "Approvals initiated by the current user and their pending tasks can be accessed through the To-Do Center in the top toolbar"

日本語版 line 83-87：
> To-Doセンターは、ユーザーがTo-Doタスクを確認・処理するための統一された入り口を提供します。現在のユーザーが申請した承認や保留中のタスクは、上部のツールバーにあるTo-Doセンターからアクセスできます。

**To-Do Center の主要セクション：**

1. **My Submissions（申請済み）**
   - 自分が申請した承認一覧を表示
   - 各申請の進捗状況を確認可能
   - （英語版 line 121-126, 日本語版 line 89-97）

2. **My To-dos（自分のTo-Do）**
   - 自分に割り当てられた承認タスク（承認が必要な案件）を表示
   - To-Do リスト形式と詳細ビューを提供
   - （英語版 line 135-146, 日本語版 line 99-107）

#### 4.2 カード設定による表示

v2.0+ より、To-Do Center 内のカード表示をカスタマイズ可能（英語版 line 68-78, 日本語版も同様）：

**My application Card（申請済みカード）：**
- 業務コレクションのフィールドを自由に選択して表示
- 承認関連情報も表示可能
- カスタマイズしたカードが To-Do Center リストに表示

**My Approvals Card（承認タスクカード）：**
- 承認ノード単位でカード設定が可能
- 承認者が見るカードレイアウトを自由にカスタマイズ

### 5. 承認者側の操作API（参考）

英語版 `./repository/docs/docs/en/workflow/triggers/approval.md` の line 253-344 に記載。

承認者は以下の API で承認処理を実行：

**承認/却下：**
```bash
curl -X POST -H 'Authorization: Bearer <your token>' -d \
  '{
    "status": 2,
    "comment": "Looks good to me.",
    "data": { "<field to modify>": "<value>" }
  }'
  "http://localhost:3000/api/approvalRecords:submit/<record id>"
```

**差し戻し：**
```bash
curl -X POST -H 'Authorization: Bearer <your token>' -d \
  '{
    "returnToNodeKey": "<node key>",
  }'
  "http://localhost:3000/api/approvalRecords:return/<record id>"
```

**委任：**
```bash
curl -X POST -H 'Authorization: Bearer <your token>' -d \
  '{
    "assignee": <user id>,
  }'
  "http://localhost:3000/api/approvalRecords:delegate/<record id>"
```

（日本語版 line 211-301 に同等の記載）

## 会話内容

この調査は、ユーザーが以下を知りたいとの要望により実施：
1. 承認プラグインについて詳しく知りたい
2. 具体的な使い方
3. 承認依頼の投げ方
4. 投げられた依頼の確認方法

すべてのドキュメントを読み込み、体系的に整理してまとめました。

## 問題・疑問点

特になし。ドキュメントが十分詳細で、質問の内容すべてについて答えが提示されていました。
