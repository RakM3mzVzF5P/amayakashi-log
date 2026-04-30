# Application Design統合ドキュメント

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30  
**フェーズ**: INCEPTION - Application Design

---

## 目次

1. [設計概要](#1-設計概要)
2. [システムアーキテクチャ](#2-システムアーキテクチャ)
3. [コンポーネント設計](#3-コンポーネント設計)
4. [サービス層設計](#4-サービス層設計)
5. [データベーススキーマ設計](#5-データベーススキーマ設計)
6. [コンポーネント依存関係](#6-コンポーネント依存関係)
7. [設計判断の根拠](#7-設計判断の根拠)
8. [次のステップ](#8-次のステップ)

---

## 1. 設計概要

### 1.1 設計目的
amayakashi-logシステムの主要コンポーネント、サービス層、データベーススキーマ、コンポーネント依存関係を定義し、CONSTRUCTION Phaseでの実装の基礎を確立します。

### 1.2 設計スコープ
- **3層アーキテクチャ**: クライアント（PyQt6）、サーバー（Flask）、データベース（PostgreSQL）
- **マルチモーダルAI統合**: AWS Bedrock（Claude 4）、OpenAI Whisper
- **複雑なデータフロー**: ログ収集 → 処理 → AI生成 → 保存 → 表示
- **プライバシー制御**: ブラックリスト、選択的ログ取得

### 1.3 設計方針
- **コンポーネント組織化**: モジュール別（各機能を独立したモジュール）
- **ログ収集粒度**: 個別コンポーネント（各機能独立）
- **プライバシー制御**: 専用のPrivacyManager（中央集権的管理）
- **AI生成配置**: すべてサーバー側
- **リアルタイム通知**: サーバー側生成、WebSocket配信
- **DB正規化**: 適度な正規化（2NF〜3NF）
- **サービス粒度**: 粗粒度サービス（LogService, AIService, UserService）
- **コンポーネント間通信**: 直接呼び出し
- **エラーハンドリング**: 各コンポーネントで個別処理
- **設定管理**: ハイブリッド（ローカル+DB）

---

## 2. システムアーキテクチャ

### 2.1 全体アーキテクチャ

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT TIER (PyQt6)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ UI Layer                                              │  │
│  │  - WidgetUI, SummaryEditorUI, SettingsUI            │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Business Logic Layer                                  │  │
│  │  - Log Collectors, Privacy Manager, Log Processor    │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Communication Layer                                   │  │
│  │  - REST Client, WebSocket Client                     │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                    SERVER TIER (Flask)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Web UI Layer                                          │  │
│  │  - Jinja2 Templates, Static Assets                   │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ API Layer                                             │  │
│  │  - REST Endpoints, WebSocket Handlers                │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Business Logic Layer                                  │  │
│  │  - Services (Log, AI, User)                          │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Data Access Layer                                     │  │
│  │  - Repository Pattern, ORM (SQLAlchemy)              │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────┐
│                    DATA TIER                                 │
│  - AWS RDS (PostgreSQL)                                     │
│  - AWS Bedrock (Claude 4, Whisper)                          │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 技術スタック

| カテゴリ | 技術 |
|---|---|
| クライアント言語 | Python 3.x |
| クライアントGUI | PyQt6 |
| サーバー言語 | Python 3.x |
| サーバーフレームワーク | Flask + Jinja2 |
| データベース | AWS RDS (PostgreSQL) |
| ORM | SQLAlchemy |
| AI | AWS Bedrock (Claude 4), OpenAI Whisper |
| OS API | pynput, pywin32, Pillow, OpenCV, pyaudio |
| 通信 | REST API (requests), WebSocket (Flask-SocketIO, websocket-client) |

---

## 3. コンポーネント設計

### 3.1 コンポーネント数サマリー

| カテゴリ | コンポーネント数 |
|---|---|
| クライアント - UI Layer | 4 |
| クライアント - Business Logic Layer | 8 |
| クライアント - Communication Layer | 2 |
| サーバー - Web UI Layer | 2 |
| サーバー - API Layer | 6 |
| サーバー - Business Logic Layer | 3 |
| サーバー - Data Access Layer | 3 |
| 共有 - データモデル | 4 |
| 共有 - ユーティリティ | 3 |
| 外部サービス | 3 |
| **合計** | **38** |

### 3.2 主要コンポーネント

#### クライアント側
- **WidgetUI**: デスクトップウィジェット
- **SummaryEditorUI**: サマリーエディター
- **SettingsUI**: 設定画面
- **LogCollectorOrchestrator**: ログ収集統括
- **CameraCollector, ScreenshotCollector, MicrophoneCollector, TypingCollector, ClipboardCollector, WindowCollector**: 各ログ収集コンポーネント
- **ActivityMonitor**: 稼働状況監視
- **PrivacyManager**: プライバシー制御
- **LogProcessor**: ログ処理
- **RESTClient, WebSocketClient**: 通信コンポーネント

#### サーバー側
- **AuthenticationAPI, LogSubmissionAPI, ReportAPI, CharacterAPI, SettingsAPI**: APIエンドポイント
- **WebSocketHandler**: WebSocket管理
- **LogService, AIService, UserService**: ビジネスロジックサービス
- **LogRepository, UserRepository, CharacterRepository**: データアクセス

**詳細**: `components.md`を参照

---

## 4. サービス層設計

### 4.1 サービス定義

#### LogService
**責務**: ログ処理と報告書管理
- ログ送信の受付と処理
- 一次要約の生成（AIServiceに委譲）
- サマリー確定の処理
- 最終生成の実行（AIServiceに委譲）
- 報告書の取得と管理

#### AIService
**責務**: AI生成処理
- AWS Bedrock接続の管理
- 一次要約の生成
- 最終生成の実行（上司日誌 + 甘やかしログ）
- リアルタイム通知メッセージの生成
- プロンプト管理

#### UserService
**責務**: ユーザー管理と認証
- ユーザー認証（ユーザー名/パスワード）
- セッション管理
- ユーザー設定の取得と更新
- キャラクター設定の取得と更新

### 4.2 サービス間インタラクション

**ログ送信〜一次要約生成**:
```
Client → LogSubmissionAPI → LogService → AIService → Bedrock
                                ↓
                          LogRepository
```

**サマリー確定〜最終生成**:
```
Client → LogSubmissionAPI → LogService → AIService → Bedrock
                                ↓            ↓
                          LogRepository  CharacterRepository
                                ↓
                          WebSocketHandler → Client
```

**詳細**: `services.md`を参照

---

## 5. データベーススキーマ設計

### 5.1 テーブル一覧

| テーブル名 | 説明 |
|---|---|
| users | ユーザー情報 |
| user_settings | ユーザー設定（キャラクター設定等） |
| characters | AIキャラクターマスタ |
| raw_logs | 生ログデータ（構造化データ） |
| summaries | 一次要約（サマリー） |
| reports | 最終報告書（上司日誌 + 甘やかしログ） |
| notifications | 通知履歴 |

### 5.2 リレーションシップ

```
users (1) ─────< (N) user_settings
  │
  │ (1)
  │
  ├─────< (N) raw_logs
  │         │ (1)
  │         │
  │         └─────< (N) summaries
  │                   │ (1)
  │                   │
  │                   └─────< (N) reports
  │
  └─────< (N) notifications

characters (1) ─────< (N) user_settings (favorite_character_id)
  │
  └─────< (N) notifications
```

### 5.3 正規化レベル
適度な正規化（2NF〜3NF）を採用し、データ整合性とクエリパフォーマンスのバランスを取ります。

**詳細**: `database-schema.md`を参照

---

## 6. コンポーネント依存関係

### 6.1 クライアント側依存関係

```
UI Layer
  ↓
Business Logic Layer
  ↓
Communication Layer
  ↓
Server
```

### 6.2 サーバー側依存関係

```
Web UI Layer / API Layer
  ↓
Business Logic Layer (Services)
  ↓
Data Access Layer (Repositories)
  ↓
Database
```

### 6.3 データフロー

**ログ収集フロー**:
```
ユーザー操作 → OS API → Collector → PrivacyManager → LocalStorageManager
```

**一次要約生成フロー**:
```
LogProcessor → RESTClient → LogSubmissionAPI → LogService → AIService → Bedrock
```

**最終生成フロー**:
```
RESTClient → LogSubmissionAPI → LogService → AIService → Bedrock → LogRepository → WebSocketHandler → Client
```

**詳細**: `component-dependency.md`を参照

---

## 7. 設計判断の根拠

### 7.1 モジュール別組織化
**判断**: クライアント側のコンポーネントをモジュール別に組織化

**根拠**:
- 各機能（カメラ、スクショ、マイク等）を独立したモジュールとして管理
- 再利用性とテスト容易性を向上
- 新しいログ収集機能の追加が容易

---

### 7.2 個別コンポーネント
**判断**: ログ収集機能を個別のコンポーネントとして設計

**根拠**:
- 各機能の独立性を保つ
- プライバシー設定で個別に有効/無効を切り替え可能
- 保守性とテスト容易性を向上

---

### 7.3 中央集権的プライバシー管理
**判断**: PrivacyManagerがすべてのプライバシー制御を管理

**根拠**:
- プライバシー制御の一貫性を保証
- ブラックリスト判定を一元管理
- 各Collectorがプライバシー設定を個別に管理する必要がない

---

### 7.4 すべてサーバー側でAI生成
**判断**: 一次要約と最終生成をすべてサーバー側で実行

**根拠**:
- AWS認証情報（アクセスキー/シークレットキー）をサーバー側のみで管理
- セキュリティリスクを最小化
- クライアント側の処理負荷を軽減

---

### 7.5 サーバー側生成、WebSocket配信
**判断**: リアルタイム通知をサーバー側で生成し、WebSocketで配信

**根拠**:
- AI生成処理をサーバー側に集約
- リアルタイム性を確保（WebSocket）
- クライアント側の処理負荷を軽減

---

### 7.6 適度な正規化（2NF〜3NF）
**判断**: データベーススキーマを適度に正規化

**根拠**:
- データ整合性を保ちつつ、クエリパフォーマンスを確保
- 過度な正規化による複雑なJOINを避ける
- 読み取りパフォーマンスを重視

---

### 7.7 粗粒度サービス
**判断**: サーバー側のサービスを粗粒度で設計（LogService, AIService, UserService）

**根拠**:
- サービス数を抑え、管理を簡素化
- 各サービスが明確な責務を持つ
- 将来的なマイクロサービス化の可能性を残す

---

### 7.8 直接呼び出し
**判断**: クライアント内のコンポーネント間通信を直接呼び出しで実装

**根拠**:
- シンプルで理解しやすい
- 同期処理に適している
- イベントバスやメディエーターパターンのオーバーヘッドを避ける

---

### 7.9 各コンポーネントで個別処理
**判断**: エラーハンドリングを各コンポーネントで個別に処理

**根拠**:
- 各コンポーネントが独自のエラー処理ロジックを持つ
- エラーの可視性を向上
- デバッグ容易性を向上

---

### 7.10 ハイブリッド設定管理
**判断**: プライバシー設定はローカル、キャラクター設定はDBで管理

**根拠**:
- プライバシー設定はクライアント側で即座にアクセス可能
- キャラクター設定はサーバー側で一元管理
- オフライン動作とマルチデバイス対応のバランスを取る

---

## 8. 次のステップ

### 8.1 Units Generation
Application Design完了後、Units Generationステージに進みます。

**Units Generationの目的**:
- システムを複数の実装単位（Units）に分割
- 各Unitの実装順序を決定
- 各Unitの依存関係を明確化

**予想されるUnits**:
1. **Unit 1: クライアント - ログ収集基盤**
   - LogCollectorOrchestrator
   - 各Collector（Camera, Screenshot, Typing, etc.）
   - PrivacyManager
   - LocalStorageManager

2. **Unit 2: クライアント - UI基盤**
   - WidgetUI
   - SummaryEditorUI
   - SettingsUI
   - NotificationManager

3. **Unit 3: クライアント - 通信基盤**
   - RESTClient
   - WebSocketClient

4. **Unit 4: サーバー - 認証とユーザー管理**
   - AuthenticationAPI
   - UserService
   - UserRepository
   - CharacterRepository

5. **Unit 5: サーバー - ログ処理とAI生成**
   - LogSubmissionAPI
   - LogService
   - AIService
   - LogRepository

6. **Unit 6: サーバー - Web UI**
   - TemplateRenderer
   - ReportAPI
   - CharacterAPI
   - SettingsAPI

7. **Unit 7: データベース初期化**
   - スキーマ作成
   - 初期データ投入

8. **Unit 8: 統合テスト**
   - エンドツーエンドテスト
   - パフォーマンステスト

---

### 8.2 CONSTRUCTION Phase
Units Generation完了後、CONSTRUCTION Phaseに進みます。

**CONSTRUCTION Phaseの主要ステージ**:
- Functional Design（各Unit）
- NFR Requirements（各Unit）
- NFR Design（各Unit）
- Infrastructure Design（各Unit）
- Code Generation（各Unit）
- Build and Test

---

## 9. 成果物サマリー

### 9.1 作成された設計ドキュメント
1. **components.md** - コンポーネント一覧と責務定義（38コンポーネント）
2. **component-methods.md** - コンポーネントメソッド定義（主要コンポーネント）
3. **services.md** - サービス層設計（3サービス）
4. **database-schema.md** - データベーススキーマ設計（7テーブル）
5. **component-dependency.md** - コンポーネント依存関係とデータフロー
6. **application-design.md** - 統合設計ドキュメント（本ドキュメント）

### 9.2 設計の完全性
- ✅ すべてのコンポーネントが明確な責務を持つ
- ✅ コンポーネント間の依存関係が明確
- ✅ データフローが一貫している
- ✅ プライバシー制御が適切に設計されている
- ✅ AI統合が適切に設計されている
- ✅ データベーススキーマが要件を満たしている

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
