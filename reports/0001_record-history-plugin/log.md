# Record Historyプラグイン調査ログ

## 調査日時
2026-06-05

## 調査ファイル一覧

### 英語ドキュメント
- `/Users/yuta/space/docs/nocobase/repository/docs/docs/en/record-history/index.md` - Record History機能の詳細ドキュメント
- `/Users/yuta/space/docs/nocobase/repository/docs/docs/en/plugins/@nocobase/plugin-record-history/index.md` - プラグインメタ情報
- `/Users/yuta/space/docs/nocobase/repository/docs/docs/en/guide/index.md` - 全体ガイド（Record Historyの位置付け確認）
- `/Users/yuta/space/docs/nocobase/repository/docs/docs/en/log-and-monitor/logger/overview.md` - ログ・監視機能の概要（他の機能との比較）

### 日本語ドキュメント
- `/Users/yuta/space/docs/nocobase/repository/docs/docs/ja/record-history/index.md` - Record History機能の日本語版ドキュメント
- `/Users/yuta/space/docs/nocobase/repository/docs/docs/ja/plugins/@nocobase/plugin-record-history/index.md` - プラグインメタ情報（日本語）

### 他の言語での同等ファイル
- ポルトガル語、中国語、スペイン語、韓国語、フランス語、ドイツ語、ロシア語用のプラグインメタファイルが存在

## 調査内容

### 1. Record Historyプラグインとは

Record Historyプラグインはnocobaseに内蔵されたデータ変更追跡プラグインで、以下の特徴を持つ：

#### プラグイン基本情報
- **パッケージ名**: `@nocobase/plugin-record-history`
- **表示名**: `Record history`
- **対応バージョン**: 2.x
- **ライセンス**: 有料機能（isFree: false）
- **デフォルト状態**: 無効（defaultEnabled: false）
- **エディションレベル**: 2（推奨される機能レベル）
- **デフォルトで有効**: なし（builtIn: false）

#### 機能説明
公式ドキュメントの説明：「The **Record History** plugin tracks data changes by automatically saving snapshots and differences of **create**, **update**, and **delete** operations. It helps users quickly review data modifications and audit operation activities.」

日本語版の説明：「履歴記録**プラグイン**は、データの変更履歴を追跡し、新規作成、更新、削除といった操作のスナップショットと差分を自動的に保存します。これにより、ユーザーはデータの変更を素早く確認し、操作履歴を監査することができます。」

### 2. 使い方

#### ステップ1: コレクションとフィールドの追加
- Record Historyプラグインの設定ページにアクセス
- 追跡したいコレクション（collection）とフィールドを追加
- 記録効率を向上させ、データ冗長性を避けるため、必要なコレクションとフィールドのみを設定することが推奨される
- **追跡不要な典型的なフィールド**:
  - 一意のID（unique ID）
  - 作成日時（createdAt）
  - 更新日時（updatedAt）
  - 作成者（createdBy）
  - 更新者（updatedBy）

#### ステップ2: 履歴データスナップショットの同期
重要な注意事項：
- 履歴記録有効化前に作成されたデータについては、最初の更新によってスナップショットが生成されるまで、その後の変更のみが記録される
- つまり、**履歴記録有効化以前の初回更新と削除は履歴に記録されない**
- 既存データの履歴を保持したい場合は、一度スナップショットの同期を実行可能
- スナップショットサイズの計算式：記録数 × 記録対象フィールド数
- 大規模データセットの場合は、データ範囲で絞り込み、重要なデータのみを同期することを推奨

同期プロセス：
1. 「Sync Historical Snapshots」ボタンをクリック
2. 同期対象のフィールドとデータ範囲を設定
3. 同期タスクがバックグラウンドキューに入れられ実行される
4. リスト更新で完了状況を確認可能

#### ステップ3: Record Historyブロックの使用
ユーザーインターフェース上での利用：

**ブロック追加方法**:
1. Record History Blockを選択
2. コレクションを選択
3. ページに履歴ブロックが追加される

**レコード詳細ポップアップ内での利用**:
- 「Current Record」を選択することで、その特定レコードに限定した履歴表示ブロックを追加可能
- 詳細ページのポップアップ内でレコード固有の変更履歴を表示可能

**説明文テンプレートのカスタマイズ**:
1. ブロック設定で「Edit Template」をクリック
2. 以下の操作ごとに説明文テンプレートを設定可能：
   - **Create操作**: レコード作成時の説明文
   - **Update操作**: レコード更新時の説明文
     - 全フィールド共通テンプレート、または特定フィールド個別テンプレートの両方に対応
   - **Delete操作**: レコード削除時の説明文
3. 説明文設定時に変数を使用可能（例：$\{fieldName\}など）
4. テンプレート適用範囲の選択：
   - 「All record history blocks of the current collection」：該当コレクションのすべての履歴ブロックに適用
   - 「Only this record history block」：このブロックのみに適用

### 3. 他のログ・監視機能との関係と違い

#### Server Logs（サーバーログ）
- システムの実行時情報を記録
- コード実行チェーンの追跡
- エラーまたはランタイム例外の追跡
- ターミナルまたはファイルに出力
- システムエラーの診断・トラブルシューティング用

#### Request Logs（リクエストログ）
- HTTP API リクエスト・レスポンス詳細を記録
- リクエストID、APIパス、ヘッダー、レスポンスステータスコード、実行時間を中心
- ターミナルまたはファイルに出力
- API呼び出しの追跡と実行パフォーマンス分析用

#### Audit Logs（監査ログ）
- ユーザー（またはAPI）のシステムリソースへのアクション記録
- リソース種類、対象オブジェクト、操作種類、ユーザー情報、操作ステータスを中心
- リクエストパラメータとレスポンスをメタデータとして保存
- ファイルとデータベーステーブルの両形式で保存
- 注意：リクエストパラメータと応答はデータスナップショット**ではない**
- 何らかの操作が発生したかは判定できるが、修正前の正確なデータは保持しない
- データ変更後の復元やバージョン管理には使用不可

#### Record History（レコード履歴）- **本プラグイン**
- **データコンテンツの変更履歴を記録** → Audit Logsとの主な違い
- リソース種類、リソースオブジェクト、操作種類、変更フィールド、変更前後の値を追跡
- **データ比較と監査に有用**
- データベーステーブルに保存
- **変更前後の値を正確に保持する** → データ復元やバージョン管理に適している

### 4. 調査アプローチ

1. nocobaseドキュメント内で「record-history」キーワード検索
2. 主要ドキュメントファイル（プラグインメタ、詳細ドキュメント）を読み込み
3. 日本語ドキュメントも併行して確認（AI翻訳版であることを確認）
4. ガイド内でのRecord Historyの位置付けを確認
5. ログ・監視機能の概要ドキュメントでRecord Historyと他のログ機能との関係を確認

### 5. 発見事項と判断

#### 重要な発見
1. **有料機能**: Record Historyは有料エディション機能（editionLevel: 2）
2. **バージョン制限**: nocobase v2.x対応のみ
3. **自動スナップショット機構**: 最初の更新時に自動的にスナップショットが生成される仕組み
4. **明確な機能分化**: Audit Logsはアクション記録、Record Historyはデータ内容変更記録に特化
5. **テンプレート機能**: 単なるログ記録ではなく、ユーザーフレンドリーな説明文を自動生成可能

#### デザイン上の特徴
- プラグイン方式：コア機能ではなく追加インストール型
- 設定画面ベース：プログラミングなしで設定可能
- UI統合：Record Historyブロックとしてページに直接埋め込み可能
- カスタマイズ性：テンプレート変数による柔軟な説明文生成

### 6. 未確認の点・今後の確認項目

- 実装の詳細（プラグイン内部のコード構造）
- パフォーマンス考慮（スナップショット同期時の負荷）
- APIでのアクセス方法
- 他のプラグインとの相互作用
- デプロイメント・運用時の設定方法
