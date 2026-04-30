# コンポーネント依存関係設計書

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30  
**通信パターン**: 直接呼び出し

---

## 1. 依存関係概要

このドキュメントでは、amayakashi-logシステムのコンポーネント間の依存関係、通信パターン、データフローを定義します。

---

## 2. クライアント側コンポーネント依存関係

### 2.1 依存関係マトリックス（クライアント）

| コンポーネント | 依存先 |
|---|---|
| **WidgetUI** | LogCollectorOrchestrator, NotificationManager |
| **SummaryEditorUI** | RESTClient |
| **SettingsUI** | SettingsManager, PrivacyManager |
| **NotificationManager** | WebSocketClient |
| **LogCollectorOrchestrator** | CameraCollector, ScreenshotCollector, MicrophoneCollector, TypingCollector, ClipboardCollector, WindowCollector, ActivityMonitor, PrivacyManager, LocalStorageManager |
| **CameraCollector** | LocalStorageManager, PrivacyManager, OpenCV |
| **ScreenshotCollector** | LocalStorageManager, PrivacyManager, Pillow, pywin32 |
| **MicrophoneCollector** | LocalStorageManager, PrivacyManager, pyaudio, Whisper |
| **TypingCollector** | LocalStorageManager, PrivacyManager, pynput |
| **ClipboardCollector** | LocalStorageManager, PrivacyManager, pyperclip |
| **WindowCollector** | LocalStorageManager, pywin32 |
| **ActivityMonitor** | LocalStorageManager, pynput |
| **PrivacyManager** | SettingsManager, pywin32 |
| **LogProcessor** | LocalStorageManager |
| **LocalStorageManager** | - |
| **SettingsManager** | LocalStorageManager |
| **RESTClient** | requests |
| **WebSocketClient** | websocket-client, NotificationManager |

---

### 2.2 レイヤー別依存関係図（クライアント）

```
┌─────────────────────────────────────────────────────────────┐
│                        UI Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  WidgetUI    │  │SummaryEditor │  │  SettingsUI  │     │
│  │              │  │      UI      │  │              │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
└─────────┼──────────────────┼──────────────────┼──────────────┘
          │                  │                  │
          ↓                  ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│                   Business Logic Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │LogCollector  │  │LogProcessor  │  │Privacy       │     │
│  │Orchestrator  │  │              │  │Manager       │     │
│  └──────┬───────┘  └──────────────┘  └──────┬───────┘     │
│         │                                     │              │
│         ├─────────────────────────────────────┤              │
│         │                                     │              │
│  ┌──────┴───────┐  ┌──────────────┐  ┌──────┴───────┐     │
│  │Camera        │  │Screenshot    │  │Microphone    │     │
│  │Collector     │  │Collector     │  │Collector     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │Typing        │  │Clipboard     │  │Window        │     │
│  │Collector     │  │Collector     │  │Collector     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │Activity      │  │Settings      │  │LocalStorage  │     │
│  │Monitor       │  │Manager       │  │Manager       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
          │                  │
          ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│                   Communication Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │RESTClient    │  │WebSocket     │  │Notification  │     │
│  │              │  │Client        │  │Manager       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. サーバー側コンポーネント依存関係

### 3.1 依存関係マトリックス（サーバー）

| コンポーネント | 依存先 |
|---|---|
| **TemplateRenderer** | Jinja2 |
| **StaticAssetManager** | - |
| **AuthenticationAPI** | UserService |
| **LogSubmissionAPI** | LogService |
| **ReportAPI** | LogService |
| **CharacterAPI** | UserService |
| **SettingsAPI** | UserService |
| **WebSocketHandler** | Flask-SocketIO, AIService |
| **LogService** | AIService, LogRepository, WebSocketHandler |
| **AIService** | boto3, CharacterRepository |
| **UserService** | UserRepository, CharacterRepository |
| **LogRepository** | SQLAlchemy, PostgreSQL |
| **UserRepository** | SQLAlchemy, PostgreSQL |
| **CharacterRepository** | SQLAlchemy, PostgreSQL |

---

### 3.2 レイヤー別依存関係図（サーバー）

```
┌─────────────────────────────────────────────────────────────┐
│                      Web UI Layer                            │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │Template      │  │StaticAsset   │                        │
│  │Renderer      │  │Manager       │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘
          │
          ↓
┌─────────────────────────────────────────────────────────────┐
│                        API Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │Authentication│  │LogSubmission │  │Report        │     │
│  │API           │  │API           │  │API           │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐     │
│  │Character     │  │Settings      │  │WebSocket     │     │
│  │API           │  │API           │  │Handler       │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
└─────────┼──────────────────┼──────────────────┼──────────────┘
          │                  │                  │
          ↓                  ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│                   Business Logic Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │LogService    │  │AIService     │  │UserService   │     │
│  │              │  │              │  │              │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
└─────────┼──────────────────┼──────────────────┼──────────────┘
          │                  │                  │
          ↓                  ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│                    Data Access Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │Log           │  │User          │  │Character     │     │
│  │Repository    │  │Repository    │  │Repository    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
└─────────┼──────────────────┼──────────────────┼──────────────┘
          │                  │                  │
          ↓                  ↓                  ↓
┌─────────────────────────────────────────────────────────────┐
│                        Data Tier                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │           AWS RDS (PostgreSQL)                       │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. クライアント・サーバー間通信

### 4.1 REST API通信

```
Client                                Server
  │                                     │
  │  POST /api/login                    │
  ├────────────────────────────────────>│
  │                                     │ AuthenticationAPI
  │                                     │      ↓
  │                                     │ UserService
  │                                     │      ↓
  │                                     │ UserRepository
  │                                     │
  │  200 OK (session cookie)            │
  │<────────────────────────────────────┤
  │                                     │
  │  POST /api/logs/submit              │
  ├────────────────────────────────────>│
  │  (structured_data)                  │ LogSubmissionAPI
  │                                     │      ↓
  │                                     │ LogService
  │                                     │      ↓
  │                                     │ AIService → Bedrock
  │                                     │      ↓
  │                                     │ LogRepository
  │                                     │
  │  200 OK (summary)                   │
  │<────────────────────────────────────┤
  │                                     │
  │  POST /api/logs/confirm             │
  ├────────────────────────────────────>│
  │  (summary)                          │ LogSubmissionAPI
  │                                     │      ↓
  │                                     │ LogService (async)
  │                                     │      ↓
  │  200 OK                             │ AIService → Bedrock
  │<────────────────────────────────────┤      ↓
  │                                     │ LogRepository
  │                                     │      ↓
  │                                     │ WebSocketHandler
  │                                     │
  │  WebSocket: report_ready            │
  │<════════════════════════════════════┤
  │                                     │
```

---

### 4.2 WebSocket通信

```
Client                                Server
  │                                     │
  │  WebSocket Connect                  │
  ├════════════════════════════════════>│
  │                                     │ WebSocketHandler
  │                                     │
  │  Connected                          │
  │<════════════════════════════════════┤
  │                                     │
  │                                     │ (状態変化検知)
  │                                     │      ↓
  │                                     │ AIService → Bedrock
  │                                     │      ↓
  │  Notification (message)             │ WebSocketHandler
  │<════════════════════════════════════┤
  │                                     │
  │  (表示)                             │
  │                                     │
```

---

## 5. データフロー

### 5.1 ログ収集フロー

```
1. ユーザー操作
   ↓
2. OS API (pynput, pywin32, OpenCV, etc.)
   ↓
3. 各Collector (Camera, Screenshot, Typing, etc.)
   ↓
4. PrivacyManager (ブラックリスト判定)
   ↓
5. LocalStorageManager (生ログ保存)
   ↓
6. LogCollectorOrchestrator (統括)
```

---

### 5.2 一次要約生成フロー

```
1. ユーザーが[提出]ボタンをクリック
   ↓
2. LogProcessor (生ログ読み込み・整形)
   ↓
3. RESTClient (整形データ送信)
   ↓
4. LogSubmissionAPI (受信)
   ↓
5. LogService (処理)
   ↓
6. LogRepository (生ログ保存)
   ↓
7. AIService (一次要約生成)
   ↓
8. AWS Bedrock (Claude 4)
   ↓
9. AIService (結果パース)
   ↓
10. LogService (サマリー返却)
   ↓
11. RESTClient (サマリー受信)
   ↓
12. SummaryEditorUI (サマリー表示)
```

---

### 5.3 最終生成フロー

```
1. ユーザーが[確定]ボタンをクリック
   ↓
2. RESTClient (修正済みサマリー送信)
   ↓
3. LogSubmissionAPI (受信)
   ↓
4. LogService (処理開始)
   ↓
5. LogRepository (サマリー保存)
   ↓
6. LogService (非同期処理開始)
   ↓
7. RESTClient (200 OK受信)
   ↓
8. AIService (最終生成実行)
   ├─> CharacterRepository (キャラクター設定取得)
   ├─> AWS Bedrock (上司日誌生成)
   └─> AWS Bedrock (甘やかしログ生成)
   ↓
9. LogRepository (最終報告書保存)
   ↓
10. WebSocketHandler (処理完了通知配信)
   ↓
11. WebSocketClient (通知受信)
   ↓
12. NotificationManager (通知表示)
```

---

### 5.4 リアルタイム通知フロー

```
1. ActivityMonitor (状態変化検知)
   ↓
2. WebSocketClient (コンテキスト送信)
   ↓
3. WebSocketHandler (受信)
   ↓
4. AIService (通知メッセージ生成)
   ├─> CharacterRepository (キャラクター設定取得)
   └─> AWS Bedrock (メッセージ生成)
   ↓
5. WebSocketHandler (通知配信)
   ↓
6. WebSocketClient (通知受信)
   ↓
7. NotificationManager (通知表示)
   ↓
8. WidgetUI (デスクトップ通知)
```

---

## 6. 通信パターン

### 6.1 同期通信（直接呼び出し）
クライアント内のコンポーネント間通信は、直接呼び出しで実装します。

**例**: WidgetUI → LogCollectorOrchestrator
```python
# WidgetUI
def on_start_button_clicked(self):
    self.orchestrator.start_recording()
```

---

### 6.2 非同期通信（WebSocket）
サーバーからクライアントへのリアルタイム通知は、WebSocketで実装します。

**例**: WebSocketHandler → WebSocketClient
```python
# WebSocketHandler (サーバー側)
def send_notification(self, user_id, notification):
    socketio.emit('notification', notification, room=user_id)

# WebSocketClient (クライアント側)
def on_message(self, message):
    if message['type'] == 'notification':
        self.notification_manager.show_notification(message['data'])
```

---

### 6.3 REST API通信
クライアント・サーバー間のデータ送受信は、REST APIで実装します。

**例**: RESTClient → LogSubmissionAPI
```python
# RESTClient (クライアント側)
def submit_logs(self, structured_data):
    response = requests.post(f"{self.base_url}/api/logs/submit", json=structured_data)
    return response.json()

# LogSubmissionAPI (サーバー側)
@app.route('/api/logs/submit', methods=['POST'])
def submit_logs():
    data = request.json
    summary = log_service.process_log_submission(session['user_id'], data)
    return jsonify({"summary": summary})
```

---

## 7. エラー伝播

### 7.1 クライアント側エラー伝播

```
Collector (例外発生)
   ↓
LogCollectorOrchestrator (キャッチ・ログ記録)
   ↓
WidgetUI (エラー通知表示)
```

**例**:
```python
# CameraCollector
def capture_image(self):
    try:
        # カメラキャプチャ処理
        pass
    except Exception as e:
        logging.error(f"Camera capture failed: {e}")
        raise CameraCollectorException("カメラキャプチャに失敗しました")

# LogCollectorOrchestrator
def collect_logs(self):
    try:
        self.camera.capture_image()
    except CameraCollectorException as e:
        logging.error(f"Log collection error: {e}")
        self.widget_ui.show_error_notification(str(e))
```

---

### 7.2 サーバー側エラー伝播

```
Repository (例外発生)
   ↓
Service (キャッチ・ログ記録・カスタム例外)
   ↓
API (キャッチ・HTTPステータスコード返却)
   ↓
RESTClient (エラーハンドリング)
   ↓
UI (エラー通知表示)
```

**例**:
```python
# LogRepository
def save_log(self, user_id, log_data):
    try:
        # データベース保存処理
        pass
    except SQLAlchemyError as e:
        logging.error(f"Database error: {e}")
        raise RepositoryException("データベースエラーが発生しました")

# LogService
def process_log_submission(self, user_id, structured_data):
    try:
        self.log_repository.save_log(user_id, structured_data)
    except RepositoryException as e:
        logging.error(f"Service error: {e}")
        raise LogServiceException("ログ処理に失敗しました")

# LogSubmissionAPI
@app.route('/api/logs/submit', methods=['POST'])
def submit_logs():
    try:
        summary = log_service.process_log_submission(session['user_id'], request.json)
        return jsonify({"summary": summary}), 200
    except LogServiceException as e:
        return jsonify({"error": str(e)}), 500
```

---

## 8. 依存関係の原則

### 8.1 レイヤー間依存の方向
- 上位レイヤーは下位レイヤーに依存する
- 下位レイヤーは上位レイヤーに依存しない

### 8.2 循環依存の禁止
- コンポーネント間の循環依存は禁止
- 必要に応じてインターフェースを導入

### 8.3 疎結合の原則
- コンポーネント間は疎結合を保つ
- 直接呼び出しでも、インターフェースを介して依存

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
