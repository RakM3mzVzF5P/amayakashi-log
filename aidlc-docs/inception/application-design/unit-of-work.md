# Unit of Work定義書

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30  
**分解戦略**: 機能別分割（エンドツーエンド優先）  
**デプロイメントモデル**: 分離デプロイメント（クライアント・サーバー別々）  
**ディレクトリ構造**: モノレポ

---

## 1. Unit分解概要

amayakashi-logシステムを**8つのUnit of Work**に分解します。エンドツーエンド優先のアプローチにより、最小限の機能で動作するシステムを早期に構築し、段階的に機能を拡張します。

### 分解方針
- **機能別分割**: 各機能を独立したUnitとして開発
- **エンドツーエンド優先**: Unit 1で最小限のE2E機能を実装
- **段階的拡張**: Unit 2以降で機能を段階的に追加
- **モック優先**: AI統合は最初モック、後で実AI統合
- **分離デプロイメント**: クライアントとサーバーを別々にデプロイ

---

## 2. Unit一覧

| Unit ID | Unit名 | 目的 | 優先度 |
|---|---|---|---|
| Unit 1 | 基盤・E2E最小実装 | 共有コンポーネント、データベース、最小限のE2E機能 | 最高 |
| Unit 2 | クライアント - ログ収集 | マルチモーダルログ収集機能 | 高 |
| Unit 3 | クライアント - UI | デスクトップウィジェット、サマリーエディター、設定画面 | 高 |
| Unit 4 | サーバー - 認証・ユーザー管理 | ユーザー認証、セッション管理、設定管理 | 高 |
| Unit 5 | サーバー - ログ処理 | ログ受信、処理、保存 | 高 |
| Unit 6 | サーバー - AI統合（実装） | AWS Bedrock統合、実AI生成 | 中 |
| Unit 7 | サーバー - Web UI | Web管理画面、報告書表示 | 中 |
| Unit 8 | 統合テスト | E2Eテスト、統合テスト | 中 |

---

## 3. Unit詳細定義

### Unit 1: 基盤・E2E最小実装

#### 目的
システムの基盤となる共有コンポーネント、データベーススキーマ、最小限のエンドツーエンド機能を実装し、早期に動作するシステムを構築します。

#### 含まれるコンポーネント

**共有コンポーネント**:
- ValidationUtil
- SerializationUtil
- EncryptionUtil
- LogData, ReportData, UserData, CharacterData（データモデル）

**データベース**:
- 全7テーブルのスキーマ作成（users, user_settings, characters, raw_logs, summaries, reports, notifications）
- 初期データ投入（デフォルトキャラクター3名）
- マイグレーション管理

**クライアント側（最小限）**:
- LocalStorageManager
- SettingsManager
- RESTClient（基本機能のみ）
- 簡易WidgetUI（記録開始/停止ボタンのみ）
- 簡易LogCollectorOrchestrator（1つのCollectorのみ統合）
- TypingCollector（最もシンプルなCollector）

**サーバー側（最小限）**:
- AuthenticationAPI（ログイン/ログアウトのみ）
- LogSubmissionAPI（ログ受信のみ）
- UserService（認証のみ）
- LogService（ログ保存のみ）
- AIService（モック実装 - 固定文字列を返す）
- UserRepository
- LogRepository
- CharacterRepository

#### 責務
- 共有コンポーネントの実装と他Unitでの再利用基盤確立
- データベーススキーマの作成と初期データ投入
- 最小限のE2E機能（ログイン → タイピングログ収集 → サーバー送信 → モックAI生成 → 保存）の実装
- 他Unitの開発基盤の確立

#### 境界
- AI統合は含まない（モック実装のみ）
- 複雑なUI機能は含まない
- マルチモーダルログ収集は含まない（タイピングのみ）
- Web UI は含まない

#### 成果物
- 共有ライブラリ（`shared/`）
- データベースマイグレーションスクリプト（`database/migrations/`）
- 最小限のクライアントアプリ（`client/`）
- 最小限のサーバーアプリ（`server/`）
- E2E動作確認手順書

---

### Unit 2: クライアント - ログ収集

#### 目的
マルチモーダルログ収集機能を実装し、カメラ、スクリーンショット、マイク、クリップボード、ウィンドウ監視、稼働監視を追加します。

#### 含まれるコンポーネント

**クライアント側**:
- CameraCollector
- ScreenshotCollector
- MicrophoneCollector（Whisper統合含む）
- ClipboardCollector
- WindowCollector
- ActivityMonitor
- LogCollectorOrchestrator（完全版 - 全Collectorを統合）
- PrivacyManager（簡易版 - 基本的な有効/無効のみ）
- LogProcessor

#### 責務
- マルチモーダルログ収集機能の実装
- 各Collectorの実装とテスト
- LogCollectorOrchestratorによる統合管理
- 簡易的なプライバシー制御
- 生ログの集計・整形

#### 境界
- UI機能は含まない（Unit 3で実装）
- 高度なプライバシー制御（ブラックリスト）は含まない
- サーバー側機能は含まない

#### 依存関係
- Unit 1（共有コンポーネント、LocalStorageManager、SettingsManager）

#### 成果物
- ログ収集モジュール（`client/collectors/`）
- ログ処理モジュール（`client/processors/`）
- プライバシー管理モジュール（`client/privacy/`）
- 単体テスト

---

### Unit 3: クライアント - UI

#### 目的
ユーザーインターフェースを実装し、デスクトップウィジェット、サマリーエディター、設定画面、通知機能を提供します。

#### 含まれるコンポーネント

**クライアント側**:
- WidgetUI（完全版）
- SummaryEditorUI
- SettingsUI
- NotificationManager
- WebSocketClient
- PrivacyManager（完全版 - ブラックリスト機能追加）

#### 責務
- デスクトップウィジェットの実装
- サマリーエディターの実装
- 設定画面の実装
- デスクトップ通知の実装
- WebSocket通信の実装
- 完全なプライバシー制御（ブラックリスト）の実装

#### 境界
- ログ収集機能は含まない（Unit 2で実装済み）
- サーバー側機能は含まない

#### 依存関係
- Unit 1（共有コンポーネント、RESTClient、SettingsManager）
- Unit 2（LogCollectorOrchestrator、PrivacyManager基本版）

#### 成果物
- UIモジュール（`client/ui/`）
- 通信モジュール（`client/communication/`）
- プライバシー管理モジュール完全版（`client/privacy/`）
- 単体テスト

---

### Unit 4: サーバー - 認証・ユーザー管理

#### 目的
ユーザー認証、セッション管理、ユーザー設定管理、キャラクター設定管理を実装します。

#### 含まれるコンポーネント

**サーバー側**:
- AuthenticationAPI（完全版）
- CharacterAPI
- SettingsAPI
- UserService（完全版）

#### 責務
- ユーザー認証（ユーザー名/パスワード）
- セッション管理
- ユーザー設定の取得/更新
- キャラクター設定の取得/更新
- ユーザー作成/削除

#### 境界
- ログ処理機能は含まない（Unit 5で実装）
- AI生成機能は含まない（Unit 6で実装）
- Web UI は含まない（Unit 7で実装）

#### 依存関係
- Unit 1（共有コンポーネント、UserRepository、CharacterRepository）

#### 成果物
- 認証APIモジュール（`server/api/auth/`）
- ユーザー管理APIモジュール（`server/api/users/`）
- ユーザーサービスモジュール（`server/services/user/`）
- 単体テスト

---

### Unit 5: サーバー - ログ処理

#### 目的
ログ受信、処理、保存、報告書取得機能を実装します。AI生成はモックのまま維持します。

#### 含まれるコンポーネント

**サーバー側**:
- LogSubmissionAPI（完全版）
- ReportAPI
- LogService（完全版 - AI生成はモック）
- WebSocketHandler

#### 責務
- ログ受信と保存
- 一次要約生成（モックAI）
- サマリー確定と最終生成（モックAI）
- 報告書取得
- WebSocket通知配信

#### 境界
- 実AI生成は含まない（Unit 6で実装）
- Web UI は含まない（Unit 7で実装）

#### 依存関係
- Unit 1（共有コンポーネント、LogRepository、AIService（モック版））
- Unit 4（UserService - ユーザー認証）

#### 成果物
- ログAPIモジュール（`server/api/logs/`）
- ログサービスモジュール（`server/services/log/`）
- WebSocketモジュール（`server/websocket/`）
- 単体テスト

---

### Unit 6: サーバー - AI統合（実装）

#### 目的
AWS Bedrock統合を実装し、モックAI生成を実AI生成に置き換えます。

#### 含まれるコンポーネント

**サーバー側**:
- AIService（実装版 - AWS Bedrock統合）

#### 責務
- AWS Bedrock接続の実装
- 一次要約生成（実AI）
- 最終生成（上司日誌 + 甘やかしログ）（実AI）
- リアルタイム通知メッセージ生成（実AI）
- プロンプト管理

#### 境界
- ログ処理機能は含まない（Unit 5で実装済み）
- Web UI は含まない（Unit 7で実装）

#### 依存関係
- Unit 1（共有コンポーネント、CharacterRepository）
- Unit 5（LogService - AI生成呼び出し元）

#### 成果物
- AIサービスモジュール実装版（`server/services/ai/`）
- プロンプトテンプレート（`server/prompts/`）
- 単体テスト

---

### Unit 7: サーバー - Web UI

#### 目的
Web管理画面を実装し、報告書表示、カレンダーアーカイブ、キャラクター管理機能を提供します。

#### 含まれるコンポーネント

**サーバー側**:
- TemplateRenderer
- StaticAssetManager
- Jinja2テンプレート（ログイン、マイページ、カレンダー、報告書表示、キャラクター管理）

#### 責務
- Web管理画面のレンダリング
- 報告書表示（上司日誌 + 甘やかしログ）
- カレンダーアーカイブ
- キャラクター管理画面
- 静的アセット配信

#### 境界
- API機能は含まない（Unit 4, 5で実装済み）
- AI生成機能は含まない（Unit 6で実装済み）

#### 依存関係
- Unit 1（共有コンポーネント）
- Unit 4（AuthenticationAPI、UserService）
- Unit 5（ReportAPI、LogService）

#### 成果物
- Webテンプレートモジュール（`server/templates/`）
- 静的アセット（`server/static/`）
- 単体テスト

---

### Unit 8: 統合テスト

#### 目的
すべてのUnitが統合された状態でのエンドツーエンドテスト、統合テスト、パフォーマンステストを実施します。

#### 含まれるコンポーネント

**テスト**:
- E2Eテストスイート
- 統合テストスイート
- パフォーマンステストスイート
- テストデータ生成ツール

#### 責務
- エンドツーエンドテスト（ログイン → ログ収集 → AI生成 → 報告書表示）
- 統合テスト（Unit間の連携テスト）
- パフォーマンステスト（負荷テスト、レスポンスタイムテスト）
- バグ修正と品質保証

#### 境界
- 新機能の実装は含まない
- 単体テストは各Unitで実装済み

#### 依存関係
- すべてのUnit（Unit 1〜7）

#### 成果物
- E2Eテストスイート（`tests/e2e/`）
- 統合テストスイート（`tests/integration/`）
- パフォーマンステストスイート（`tests/performance/`）
- テスト実行手順書

---

## 4. コード組織化戦略（ディレクトリ構造）

### 4.1 モノレポ構造

```
amayakashi-log/                    # ワークスペースルート
├── shared/                        # 共有コンポーネント（Unit 1）
│   ├── models/                    # データモデル
│   │   ├── log_data.py
│   │   ├── report_data.py
│   │   ├── user_data.py
│   │   └── character_data.py
│   └── utils/                     # ユーティリティ
│       ├── validation.py
│       ├── serialization.py
│       └── encryption.py
│
├── database/                      # データベース（Unit 1）
│   ├── migrations/                # マイグレーションスクリプト
│   │   ├── 001_create_users.sql
│   │   ├── 002_create_characters.sql
│   │   └── ...
│   └── seeds/                     # 初期データ
│       └── default_characters.sql
│
├── client/                        # クライアント側
│   ├── collectors/                # ログ収集（Unit 2）
│   │   ├── camera_collector.py
│   │   ├── screenshot_collector.py
│   │   ├── microphone_collector.py
│   │   ├── typing_collector.py
│   │   ├── clipboard_collector.py
│   │   ├── window_collector.py
│   │   └── activity_monitor.py
│   ├── processors/                # ログ処理（Unit 2）
│   │   └── log_processor.py
│   ├── privacy/                   # プライバシー管理（Unit 2, 3）
│   │   └── privacy_manager.py
│   ├── ui/                        # UI（Unit 3）
│   │   ├── widget_ui.py
│   │   ├── summary_editor_ui.py
│   │   ├── settings_ui.py
│   │   └── notification_manager.py
│   ├── communication/             # 通信（Unit 1, 3）
│   │   ├── rest_client.py
│   │   └── websocket_client.py
│   ├── storage/                   # ストレージ（Unit 1）
│   │   ├── local_storage_manager.py
│   │   └── settings_manager.py
│   ├── orchestrator/              # オーケストレーター（Unit 1, 2）
│   │   └── log_collector_orchestrator.py
│   └── main.py                    # エントリーポイント
│
├── server/                        # サーバー側
│   ├── api/                       # API層
│   │   ├── auth/                  # 認証API（Unit 4）
│   │   │   └── authentication_api.py
│   │   ├── users/                 # ユーザー管理API（Unit 4）
│   │   │   ├── character_api.py
│   │   │   └── settings_api.py
│   │   └── logs/                  # ログAPI（Unit 5）
│   │       ├── log_submission_api.py
│   │       └── report_api.py
│   ├── services/                  # ビジネスロジック層
│   │   ├── user/                  # ユーザーサービス（Unit 4）
│   │   │   └── user_service.py
│   │   ├── log/                   # ログサービス（Unit 5）
│   │   │   └── log_service.py
│   │   └── ai/                    # AIサービス（Unit 1（モック）, Unit 6（実装））
│   │       └── ai_service.py
│   ├── repositories/              # データアクセス層（Unit 1）
│   │   ├── user_repository.py
│   │   ├── log_repository.py
│   │   └── character_repository.py
│   ├── websocket/                 # WebSocket（Unit 5）
│   │   └── websocket_handler.py
│   ├── templates/                 # Webテンプレート（Unit 7）
│   │   ├── login.html
│   │   ├── mypage.html
│   │   ├── calendar.html
│   │   └── report.html
│   ├── static/                    # 静的アセット（Unit 7）
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   ├── prompts/                   # プロンプトテンプレート（Unit 6）
│   │   ├── primary_summary.txt
│   │   ├── manager_report.txt
│   │   └── comfort_log.txt
│   └── main.py                    # エントリーポイント
│
├── tests/                         # テスト（Unit 8）
│   ├── unit/                      # 単体テスト（各Unit）
│   ├── integration/               # 統合テスト（Unit 8）
│   ├── e2e/                       # E2Eテスト（Unit 8）
│   └── performance/               # パフォーマンステスト（Unit 8）
│
├── aidlc-docs/                    # ドキュメント（AI-DLC生成）
│   └── ...
│
├── requirements.txt               # Python依存関係（クライアント・サーバー共通）
├── requirements-client.txt        # クライアント専用依存関係
├── requirements-server.txt        # サーバー専用依存関係
├── README.md                      # プロジェクトREADME
└── .gitignore                     # Git除外設定
```

### 4.2 デプロイメント構成

**クライアント側**:
- `client/`ディレクトリをPyInstallerでWindows実行ファイルにパッケージ化
- 配布: インストーラー（.exe）

**サーバー側**:
- `server/`ディレクトリをAWS EC2にデプロイ
- 配布: Dockerコンテナ（オプション）

---

## 5. Unit実装順序

### 推奨実装順序

1. **Unit 1: 基盤・E2E最小実装** （最優先）
   - 理由: すべてのUnitの基盤となる
   - 期間: 2週間

2. **Unit 2: クライアント - ログ収集** （高優先度）
   - 理由: コア機能の実装
   - 期間: 2週間

3. **Unit 4: サーバー - 認証・ユーザー管理** （高優先度）
   - 理由: Unit 5の前提条件
   - 期間: 1週間

4. **Unit 5: サーバー - ログ処理** （高優先度）
   - 理由: コア機能の実装
   - 期間: 2週間

5. **Unit 3: クライアント - UI** （中優先度）
   - 理由: ユーザー体験の向上
   - 期間: 2週間

6. **Unit 6: サーバー - AI統合（実装）** （中優先度）
   - 理由: モックから実AI生成への置き換え
   - 期間: 1週間

7. **Unit 7: サーバー - Web UI** （中優先度）
   - 理由: Web管理画面の実装
   - 期間: 1週間

8. **Unit 8: 統合テスト** （中優先度）
   - 理由: 品質保証
   - 期間: 1週間

**合計期間**: 約12週間（3ヶ月）

---

## 6. Unit間の連携

### 6.1 共有コンポーネントの利用
- Unit 1で実装された共有コンポーネント（`shared/`）は、すべてのUnitで利用可能
- データモデル、ユーティリティは共通インターフェースを提供

### 6.2 データベースの利用
- Unit 1で作成されたデータベーススキーマは、すべてのサーバー側Unitで利用可能
- マイグレーション管理により、スキーマ変更を追跡

### 6.3 モックからの移行
- Unit 1〜5ではAIServiceのモック実装を使用
- Unit 6で実AI統合を実装し、モックを置き換え
- インターフェースは変更しないため、他Unitの修正は不要

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
