# コンポーネントメソッド定義書

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30  
**注記**: 詳細なビジネスルールはFunctional Design（CONSTRUCTION Phase）で定義

---

## 1. クライアント側コンポーネント

### 1.1 UI Layer

#### WidgetUI

```python
class WidgetUI:
    """デスクトップウィジェットUI"""
    
    def __init__(self, orchestrator: LogCollectorOrchestrator, notification_manager: NotificationManager):
        """
        初期化
        Args:
            orchestrator: ログ収集オーケストレーター
            notification_manager: 通知マネージャー
        """
        pass
    
    def show(self) -> None:
        """ウィジェットを表示"""
        pass
    
    def hide(self) -> None:
        """ウィジェットを非表示"""
        pass
    
    def on_start_button_clicked(self) -> None:
        """[記録開始]ボタンクリック時の処理"""
        pass
    
    def on_submit_button_clicked(self) -> None:
        """[提出]ボタンクリック時の処理"""
        pass
    
    def update_status(self, status: str) -> None:
        """
        状態表示を更新
        Args:
            status: 状態（"recording", "stopped", "processing"）
        """
        pass
    
    def show_notification(self, message: str, character_icon: str) -> None:
        """
        通知を表示
        Args:
            message: 通知メッセージ
            character_icon: キャラクターアイコンのパス
        """
        pass
```

---

#### SummaryEditorUI

```python
class SummaryEditorUI:
    """サマリーエディターUI"""
    
    def __init__(self, rest_client: RESTClient):
        """
        初期化
        Args:
            rest_client: RESTクライアント
        """
        pass
    
    def show(self, summary_items: List[SummaryItem]) -> None:
        """
        サマリーエディターを表示
        Args:
            summary_items: サマリー項目のリスト
        """
        pass
    
    def hide(self) -> None:
        """サマリーエディターを非表示"""
        pass
    
    def add_item(self, item: SummaryItem) -> None:
        """
        サマリー項目を追加
        Args:
            item: 追加する項目
        """
        pass
    
    def edit_item(self, index: int, item: SummaryItem) -> None:
        """
        サマリー項目を編集
        Args:
            index: 編集する項目のインデックス
            item: 編集後の項目
        """
        pass
    
    def delete_item(self, index: int) -> None:
        """
        サマリー項目を削除
        Args:
            index: 削除する項目のインデックス
        """
        pass
    
    def on_confirm_button_clicked(self) -> None:
        """[確定]ボタンクリック時の処理"""
        pass
    
    def on_cancel_button_clicked(self) -> None:
        """[キャンセル]ボタンクリック時の処理"""
        pass
    
    def auto_save(self) -> None:
        """自動保存（5秒ごと）"""
        pass
```

---

#### SettingsUI

```python
class SettingsUI:
    """設定画面UI"""
    
    def __init__(self, settings_manager: SettingsManager, privacy_manager: PrivacyManager):
        """
        初期化
        Args:
            settings_manager: 設定マネージャー
            privacy_manager: プライバシーマネージャー
        """
        pass
    
    def show(self) -> None:
        """設定画面を表示"""
        pass
    
    def hide(self) -> None:
        """設定画面を非表示"""
        pass
    
    def load_settings(self) -> None:
        """設定を読み込んで表示"""
        pass
    
    def save_settings(self) -> None:
        """設定を保存"""
        pass
    
    def on_privacy_setting_changed(self, setting_name: str, enabled: bool) -> None:
        """
        プライバシー設定変更時の処理
        Args:
            setting_name: 設定名（"camera", "screenshot", "microphone", "typing", "clipboard"）
            enabled: 有効/無効
        """
        pass
    
    def add_blacklist_item(self, app_name: str) -> None:
        """
        ブラックリストに項目を追加
        Args:
            app_name: アプリケーション名またはウィンドウタイトル
        """
        pass
    
    def remove_blacklist_item(self, app_name: str) -> None:
        """
        ブラックリストから項目を削除
        Args:
            app_name: アプリケーション名またはウィンドウタイトル
        """
        pass
```

---

### 1.2 Business Logic Layer

#### LogCollectorOrchestrator

```python
class LogCollectorOrchestrator:
    """ログ収集オーケストレーター"""
    
    def __init__(self, 
                 camera: CameraCollector,
                 screenshot: ScreenshotCollector,
                 microphone: MicrophoneCollector,
                 typing: TypingCollector,
                 clipboard: ClipboardCollector,
                 window: WindowCollector,
                 activity: ActivityMonitor,
                 privacy: PrivacyManager,
                 storage: LocalStorageManager):
        """
        初期化
        Args:
            各ログ収集コンポーネント、プライバシーマネージャー、ストレージマネージャー
        """
        pass
    
    def start_recording(self) -> None:
        """ログ記録を開始"""
        pass
    
    def stop_recording(self) -> None:
        """ログ記録を停止"""
        pass
    
    def pause_recording(self) -> None:
        """ログ記録を一時停止（ブラックリスト判定時）"""
        pass
    
    def resume_recording(self) -> None:
        """ログ記録を再開"""
        pass
    
    def schedule_collection(self) -> None:
        """ログ収集スケジュールを設定（n分に1回）"""
        pass
    
    def collect_logs(self) -> None:
        """各コンポーネントからログを収集"""
        pass
    
    def is_recording(self) -> bool:
        """
        記録中かどうかを返す
        Returns:
            記録中ならTrue
        """
        pass
```

---

#### CameraCollector

```python
class CameraCollector:
    """カメラ撮影コンポーネント"""
    
    def __init__(self, storage: LocalStorageManager, privacy: PrivacyManager):
        """
        初期化
        Args:
            storage: ストレージマネージャー
            privacy: プライバシーマネージャー
        """
        pass
    
    def initialize_camera(self) -> bool:
        """
        カメラデバイスを初期化
        Returns:
            初期化成功ならTrue
        """
        pass
    
    def capture_image(self) -> Optional[np.ndarray]:
        """
        画像をキャプチャ
        Returns:
            キャプチャした画像（NumPy配列）、失敗時はNone
        """
        pass
    
    def save_image(self, image: np.ndarray, timestamp: datetime) -> str:
        """
        画像を保存
        Args:
            image: 保存する画像
            timestamp: タイムスタンプ
        Returns:
            保存したファイルパス
        """
        pass
    
    def is_enabled(self) -> bool:
        """
        カメラ撮影が有効かどうかを返す
        Returns:
            有効ならTrue
        """
        pass
    
    def release_camera(self) -> None:
        """カメラデバイスを解放"""
        pass
```

---

#### PrivacyManager

```python
class PrivacyManager:
    """プライバシー制御マネージャー"""
    
    def __init__(self, settings_manager: SettingsManager):
        """
        初期化
        Args:
            settings_manager: 設定マネージャー
        """
        pass
    
    def is_feature_enabled(self, feature_name: str) -> bool:
        """
        機能が有効かどうかを返す
        Args:
            feature_name: 機能名（"camera", "screenshot", "microphone", "typing", "clipboard"）
        Returns:
            有効ならTrue
        """
        pass
    
    def set_feature_enabled(self, feature_name: str, enabled: bool) -> None:
        """
        機能の有効/無効を設定
        Args:
            feature_name: 機能名
            enabled: 有効/無効
        """
        pass
    
    def is_blacklisted(self, window_title: str) -> bool:
        """
        ウィンドウタイトルがブラックリストに該当するかを判定
        Args:
            window_title: ウィンドウタイトル
        Returns:
            ブラックリストに該当するならTrue
        """
        pass
    
    def add_to_blacklist(self, app_name: str) -> None:
        """
        ブラックリストに追加
        Args:
            app_name: アプリケーション名またはウィンドウタイトル
        """
        pass
    
    def remove_from_blacklist(self, app_name: str) -> None:
        """
        ブラックリストから削除
        Args:
            app_name: アプリケーション名またはウィンドウタイトル
        """
        pass
    
    def get_blacklist(self) -> List[str]:
        """
        ブラックリストを取得
        Returns:
            ブラックリストのリスト
        """
        pass
    
    def check_current_window(self) -> bool:
        """
        現在のアクティブウィンドウがブラックリストに該当するかをチェック
        Returns:
            ブラックリストに該当するならTrue
        """
        pass
```

---

#### LogProcessor

```python
class LogProcessor:
    """ログ処理コンポーネント"""
    
    def __init__(self, storage: LocalStorageManager):
        """
        初期化
        Args:
            storage: ストレージマネージャー
        """
        pass
    
    def load_raw_logs(self, start_time: datetime, end_time: datetime) -> Dict[str, Any]:
        """
        生ログを読み込む
        Args:
            start_time: 開始時刻
            end_time: 終了時刻
        Returns:
            生ログデータ
        """
        pass
    
    def process_typing_logs(self, typing_logs: List[Dict]) -> str:
        """
        タイピングログを処理（BackSpace反映）
        Args:
            typing_logs: タイピングログのリスト
        Returns:
            最終文字列
        """
        pass
    
    def aggregate_window_logs(self, window_logs: List[Dict]) -> List[Dict]:
        """
        ウィンドウログを集計（滞在時間計算）
        Args:
            window_logs: ウィンドウログのリスト
        Returns:
            集計されたウィンドウログ
        """
        pass
    
    def remove_duplicates(self, logs: List[Dict]) -> List[Dict]:
        """
        重複情報を整理
        Args:
            logs: ログのリスト
        Returns:
            重複を除去したログ
        """
        pass
    
    def create_structured_data(self, raw_logs: Dict[str, Any]) -> Dict[str, Any]:
        """
        構造化データを作成
        Args:
            raw_logs: 生ログデータ
        Returns:
            構造化データ
        """
        pass
```

---

### 1.3 Communication Layer

#### RESTClient

```python
class RESTClient:
    """REST APIクライアント"""
    
    def __init__(self, base_url: str):
        """
        初期化
        Args:
            base_url: サーバーのベースURL
        """
        pass
    
    def login(self, username: str, password: str) -> bool:
        """
        ログイン
        Args:
            username: ユーザー名
            password: パスワード
        Returns:
            ログイン成功ならTrue
        """
        pass
    
    def logout(self) -> bool:
        """
        ログアウト
        Returns:
            ログアウト成功ならTrue
        """
        pass
    
    def submit_logs(self, structured_data: Dict[str, Any]) -> Dict[str, Any]:
        """
        整形データを送信（一次要約リクエスト）
        Args:
            structured_data: 構造化データ
        Returns:
            一次要約（サマリー）
        """
        pass
    
    def confirm_summary(self, summary: List[SummaryItem]) -> Dict[str, Any]:
        """
        修正済みサマリーを送信（最終生成リクエスト）
        Args:
            summary: 修正済みサマリー
        Returns:
            処理結果（成功/失敗）
        """
        pass
    
    def get_settings(self) -> Dict[str, Any]:
        """
        サーバー側設定を取得
        Returns:
            設定データ
        """
        pass
    
    def update_settings(self, settings: Dict[str, Any]) -> bool:
        """
        サーバー側設定を更新
        Args:
            settings: 設定データ
        Returns:
            更新成功ならTrue
        """
        pass
```

---

#### WebSocketClient

```python
class WebSocketClient:
    """WebSocketクライアント"""
    
    def __init__(self, server_url: str, notification_manager: NotificationManager):
        """
        初期化
        Args:
            server_url: WebSocketサーバーURL
            notification_manager: 通知マネージャー
        """
        pass
    
    def connect(self) -> bool:
        """
        WebSocket接続を確立
        Returns:
            接続成功ならTrue
        """
        pass
    
    def disconnect(self) -> None:
        """WebSocket接続を切断"""
        pass
    
    def on_message(self, message: Dict[str, Any]) -> None:
        """
        メッセージ受信時の処理
        Args:
            message: 受信したメッセージ
        """
        pass
    
    def on_notification(self, notification: Dict[str, Any]) -> None:
        """
        通知受信時の処理
        Args:
            notification: 通知データ
        """
        pass
    
    def reconnect(self) -> bool:
        """
        再接続
        Returns:
            再接続成功ならTrue
        """
        pass
```

---

## 2. サーバー側コンポーネント

### 2.1 API Layer

#### LogSubmissionAPI

```python
class LogSubmissionAPI:
    """ログ送信APIエンドポイント"""
    
    def __init__(self, log_service: LogService):
        """
        初期化
        Args:
            log_service: ログサービス
        """
        pass
    
    @app.route('/api/logs/submit', methods=['POST'])
    def submit_logs(self) -> Response:
        """
        整形データを受信し、一次要約を生成
        Request Body:
            structured_data: 構造化データ
        Returns:
            一次要約（サマリー）
        """
        pass
    
    @app.route('/api/logs/confirm', methods=['POST'])
    def confirm_summary(self) -> Response:
        """
        修正済みサマリーを受信し、最終生成を実行
        Request Body:
            summary: 修正済みサマリー
        Returns:
            処理結果
        """
        pass
```

---

### 2.2 Business Logic Layer

#### LogService

```python
class LogService:
    """ログサービス"""
    
    def __init__(self, 
                 ai_service: AIService,
                 log_repository: LogRepository,
                 websocket_handler: WebSocketHandler):
        """
        初期化
        Args:
            ai_service: AIサービス
            log_repository: ログリポジトリ
            websocket_handler: WebSocketハンドラー
        """
        pass
    
    def process_log_submission(self, user_id: int, structured_data: Dict[str, Any]) -> List[SummaryItem]:
        """
        ログ送信を処理し、一次要約を生成
        Args:
            user_id: ユーザーID
            structured_data: 構造化データ
        Returns:
            一次要約（サマリー項目のリスト）
        """
        pass
    
    def process_summary_confirmation(self, user_id: int, summary: List[SummaryItem]) -> bool:
        """
        サマリー確定を処理し、最終生成を実行
        Args:
            user_id: ユーザーID
            summary: 修正済みサマリー
        Returns:
            処理成功ならTrue
        """
        pass
    
    def get_report(self, user_id: int, date: datetime) -> Dict[str, Any]:
        """
        指定日の報告書を取得
        Args:
            user_id: ユーザーID
            date: 日付
        Returns:
            報告書データ（上司日誌 + 甘やかしログ）
        """
        pass
    
    def get_report_list(self, user_id: int) -> List[Dict[str, Any]]:
        """
        報告書一覧を取得
        Args:
            user_id: ユーザーID
        Returns:
            報告書一覧
        """
        pass
```

---

#### AIService

```python
class AIService:
    """AIサービス"""
    
    def __init__(self, character_repository: CharacterRepository):
        """
        初期化
        Args:
            character_repository: キャラクターリポジトリ
        """
        pass
    
    def generate_primary_summary(self, structured_data: Dict[str, Any]) -> List[SummaryItem]:
        """
        一次要約を生成
        Args:
            structured_data: 構造化データ
        Returns:
            一次要約（サマリー項目のリスト）
        """
        pass
    
    def generate_final_report(self, user_id: int, summary: List[SummaryItem]) -> Dict[str, Any]:
        """
        最終生成を実行（上司日誌 + 甘やかしログ）
        Args:
            user_id: ユーザーID
            summary: 修正済みサマリー
        Returns:
            最終報告書（上司日誌 + 甘やかしログ）
        """
        pass
    
    def generate_notification_message(self, user_id: int, context: Dict[str, Any]) -> str:
        """
        リアルタイム通知メッセージを生成
        Args:
            user_id: ユーザーID
            context: コンテキスト（直前のログ、状態変化等）
        Returns:
            通知メッセージ
        """
        pass
    
    def call_bedrock(self, prompt: str, images: List[str] = None) -> str:
        """
        AWS Bedrockを呼び出し
        Args:
            prompt: プロンプト
            images: 画像ファイルパスのリスト（オプション）
        Returns:
            AI生成結果
        """
        pass
```

---

#### UserService

```python
class UserService:
    """ユーザーサービス"""
    
    def __init__(self, 
                 user_repository: UserRepository,
                 character_repository: CharacterRepository):
        """
        初期化
        Args:
            user_repository: ユーザーリポジトリ
            character_repository: キャラクターリポジトリ
        """
        pass
    
    def authenticate(self, username: str, password: str) -> Optional[int]:
        """
        ユーザー認証
        Args:
            username: ユーザー名
            password: パスワード
        Returns:
            認証成功ならユーザーID、失敗ならNone
        """
        pass
    
    def get_user_settings(self, user_id: int) -> Dict[str, Any]:
        """
        ユーザー設定を取得
        Args:
            user_id: ユーザーID
        Returns:
            ユーザー設定
        """
        pass
    
    def update_user_settings(self, user_id: int, settings: Dict[str, Any]) -> bool:
        """
        ユーザー設定を更新
        Args:
            user_id: ユーザーID
            settings: 設定データ
        Returns:
            更新成功ならTrue
        """
        pass
    
    def get_characters(self, user_id: int) -> List[Dict[str, Any]]:
        """
        キャラクター一覧を取得
        Args:
            user_id: ユーザーID
        Returns:
            キャラクター一覧
        """
        pass
    
    def update_character(self, user_id: int, character_id: int, character_data: Dict[str, Any]) -> bool:
        """
        キャラクター設定を更新
        Args:
            user_id: ユーザーID
            character_id: キャラクターID
            character_data: キャラクターデータ
        Returns:
            更新成功ならTrue
        """
        pass
```

---

### 2.3 Data Access Layer

#### LogRepository

```python
class LogRepository:
    """ログリポジトリ"""
    
    def __init__(self, db_session: Session):
        """
        初期化
        Args:
            db_session: データベースセッション
        """
        pass
    
    def save_log(self, user_id: int, log_data: Dict[str, Any]) -> int:
        """
        ログデータを保存
        Args:
            user_id: ユーザーID
            log_data: ログデータ
        Returns:
            保存したログのID
        """
        pass
    
    def save_report(self, user_id: int, date: datetime, report_data: Dict[str, Any]) -> int:
        """
        報告書を保存
        Args:
            user_id: ユーザーID
            date: 日付
            report_data: 報告書データ
        Returns:
            保存した報告書のID
        """
        pass
    
    def get_report(self, user_id: int, date: datetime) -> Optional[Dict[str, Any]]:
        """
        報告書を取得
        Args:
            user_id: ユーザーID
            date: 日付
        Returns:
            報告書データ、存在しない場合はNone
        """
        pass
    
    def get_report_list(self, user_id: int) -> List[Dict[str, Any]]:
        """
        報告書一覧を取得
        Args:
            user_id: ユーザーID
        Returns:
            報告書一覧
        """
        pass
```

---

## 3. 共有コンポーネント

### ValidationUtil

```python
class ValidationUtil:
    """データ検証ユーティリティ"""
    
    @staticmethod
    def validate_structured_data(data: Dict[str, Any]) -> bool:
        """
        構造化データを検証
        Args:
            data: 構造化データ
        Returns:
            検証成功ならTrue
        """
        pass
    
    @staticmethod
    def validate_summary(summary: List[SummaryItem]) -> bool:
        """
        サマリーを検証
        Args:
            summary: サマリー
        Returns:
            検証成功ならTrue
        """
        pass
```

---

### SerializationUtil

```python
class SerializationUtil:
    """シリアライゼーションユーティリティ"""
    
    @staticmethod
    def to_json(obj: Any) -> str:
        """
        オブジェクトをJSON文字列に変換
        Args:
            obj: オブジェクト
        Returns:
            JSON文字列
        """
        pass
    
    @staticmethod
    def from_json(json_str: str) -> Any:
        """
        JSON文字列をオブジェクトに変換
        Args:
            json_str: JSON文字列
        Returns:
            オブジェクト
        """
        pass
```

---

### EncryptionUtil

```python
class EncryptionUtil:
    """暗号化ユーティリティ"""
    
    @staticmethod
    def encrypt_file(file_path: str, key: bytes) -> None:
        """
        ファイルを暗号化（AES-256）
        Args:
            file_path: ファイルパス
            key: 暗号化キー
        """
        pass
    
    @staticmethod
    def decrypt_file(file_path: str, key: bytes) -> None:
        """
        ファイルを復号化
        Args:
            file_path: ファイルパス
            key: 復号化キー
        """
        pass
    
    @staticmethod
    def hash_password(password: str) -> str:
        """
        パスワードをハッシュ化
        Args:
            password: パスワード
        Returns:
            ハッシュ化されたパスワード
        """
        pass
```

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
