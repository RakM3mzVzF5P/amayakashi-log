# サービス層設計書

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30  
**設計方針**: 粗粒度サービス（LogService, AIService, UserService）

---

## 1. サービス層概要

サーバー側のビジネスロジックを3つの粗粒度サービスに分割：
- **LogService**: ログ処理と報告書管理
- **AIService**: AI生成処理
- **UserService**: ユーザー管理と認証

各サービスは明確な責務を持ち、他のサービスやリポジトリと協調して動作します。

---

## 2. サービス定義

### 2.1 LogService

#### 責務
- ログ送信の受付と処理
- 一次要約の生成（AIServiceに委譲）
- サマリー確定の処理
- 最終生成の実行（AIServiceに委譲）
- 報告書の取得と管理
- WebSocket通知の配信

#### 依存関係
- **AIService**: AI生成処理を委譲
- **LogRepository**: ログデータの永続化
- **WebSocketHandler**: リアルタイム通知の配信

#### 主要メソッド
```python
def process_log_submission(user_id: int, structured_data: Dict[str, Any]) -> List[SummaryItem]
def process_summary_confirmation(user_id: int, summary: List[SummaryItem]) -> bool
def get_report(user_id: int, date: datetime) -> Dict[str, Any]
def get_report_list(user_id: int) -> List[Dict[str, Any]]
```

#### ワークフロー

**ログ送信処理**:
```
1. クライアントから整形データを受信
2. LogRepositoryに生ログを保存
3. AIServiceに一次要約生成を依頼
4. 一次要約をクライアントに返却
```

**サマリー確定処理**:
```
1. クライアントから修正済みサマリーを受信
2. LogRepositoryにサマリーを保存
3. AIServiceに最終生成を依頼
4. 最終報告書（上司日誌 + 甘やかしログ）をLogRepositoryに保存
5. WebSocketHandlerを介してクライアントに処理完了通知を配信
```

---

### 2.2 AIService

#### 責務
- AWS Bedrock接続の管理
- 一次要約の生成
- 最終生成の実行（上司日誌 + 甘やかしログ）
- リアルタイム通知メッセージの生成
- プロンプト管理
- マルチモーダルデータ（テキスト、画像）の処理

#### 依存関係
- **boto3**: AWS SDK（Bedrock接続）
- **CharacterRepository**: キャラクター設定の取得

#### 主要メソッド
```python
def generate_primary_summary(structured_data: Dict[str, Any]) -> List[SummaryItem]
def generate_final_report(user_id: int, summary: List[SummaryItem]) -> Dict[str, Any]
def generate_notification_message(user_id: int, context: Dict[str, Any]) -> str
def call_bedrock(prompt: str, images: List[str] = None) -> str
```

#### AI生成フロー

**一次要約生成**:
```
1. 構造化データを受け取る
2. プロンプトを構築:
   - タイピングログ、ウィンドウログ、稼働ログをテキスト化
   - スクリーンショット、カメラ画像を添付
3. Bedrockを呼び出し（Claude 4 - マルチモーダル）
4. AI生成結果を箇条書きサマリーに変換
5. サマリー項目のリストを返却
```

**最終生成**:
```
1. 修正済みサマリーを受け取る
2. ユーザーのキャラクター設定を取得（CharacterRepository）
3. 上司日誌用プロンプトを構築
4. Bedrockを呼び出し → 上司日誌を生成
5. 甘やかしログ用プロンプトを構築:
   - 3名のキャラクター設定を含める
   - サマリーから具体的なエビデンスを抽出
6. Bedrockを呼び出し → 甘やかしログ（チャット形式）を生成
7. 上司日誌 + 甘やかしログを返却
```

**リアルタイム通知メッセージ生成**:
```
1. コンテキスト（直前のログ、状態変化）を受け取る
2. ユーザーのキャラクター設定を取得
3. 通知用プロンプトを構築
4. Bedrockを呼び出し → パーソナライズされたメッセージを生成
5. メッセージを返却
```

#### プロンプト管理

**一次要約プロンプトテンプレート**:
```
あなたは業務ログ解析の専門家です。以下のログデータから、今日の業務の骨子を10点ほどの箇条書きにまとめてください。

【タイピングログ】
{typing_logs}

【ウィンドウログ】
{window_logs}

【稼働ログ】
{activity_logs}

【スクリーンショット】
{screenshots}

【カメラ画像】
{camera_images}

各項目は「時刻 + 業務内容 + 詳細」の形式で記述してください。
例：「14:00 顧客への資料作成（3000文字入力）」
```

**上司日誌プロンプトテンプレート**:
```
あなたは業務報告書作成の専門家です。以下のサマリーから、上司に提出する業務日誌を作成してください。

【サマリー】
{summary}

業務日誌は以下の形式で記述してください：
- 簡潔で読みやすい文章
- 成果を強調
- 具体的な数値を含める
```

**甘やかしログプロンプトテンプレート**:
```
あなたは以下の3名のAIキャラクターです。ユーザーの業務ログを見て、チャット形式でユーザーを褒めちぎってください。

【キャラクター設定】
{character_1}
{character_2}
{character_3}

【サマリー】
{summary}

チャット形式で、3名が順番に発言してください。各発言は具体的なログデータに基づき、ユーザーを全力で褒めてください。
```

---

### 2.3 UserService

#### 責務
- ユーザー認証（ユーザー名/パスワード）
- セッション管理
- ユーザー設定の取得と更新
- キャラクター設定の取得と更新
- ユーザー作成と削除

#### 依存関係
- **UserRepository**: ユーザーデータの永続化
- **CharacterRepository**: キャラクターデータの永続化

#### 主要メソッド
```python
def authenticate(username: str, password: str) -> Optional[int]
def get_user_settings(user_id: int) -> Dict[str, Any]
def update_user_settings(user_id: int, settings: Dict[str, Any]) -> bool
def get_characters(user_id: int) -> List[Dict[str, Any]]
def update_character(user_id: int, character_id: int, character_data: Dict[str, Any]) -> bool
```

#### 認証フロー
```
1. ユーザー名とパスワードを受け取る
2. UserRepositoryからユーザー情報を取得
3. パスワードをハッシュ化して比較
4. 認証成功ならユーザーIDを返却、失敗ならNone
5. セッションを作成（Flask session）
```

#### 設定管理フロー

**ユーザー設定取得**:
```
1. ユーザーIDを受け取る
2. UserRepositoryからユーザー設定を取得
3. 設定データを返却
```

**ユーザー設定更新**:
```
1. ユーザーIDと設定データを受け取る
2. 設定データを検証
3. UserRepositoryに設定を保存
4. 成功/失敗を返却
```

**キャラクター設定取得**:
```
1. ユーザーIDを受け取る
2. CharacterRepositoryからキャラクター一覧を取得
3. ユーザーのカスタマイズ設定を適用
4. キャラクター一覧を返却
```

**キャラクター設定更新**:
```
1. ユーザーID、キャラクターID、設定データを受け取る
2. 設定データを検証
3. CharacterRepositoryに設定を保存
4. 成功/失敗を返却
```

---

## 3. サービス間のインタラクション

### 3.1 ログ送信〜一次要約生成

```
Client → LogSubmissionAPI → LogService → AIService → Bedrock
                                ↓
                          LogRepository
                                ↓
Client ← LogSubmissionAPI ← LogService
```

**シーケンス**:
1. クライアントが整形データを送信（POST /api/logs/submit）
2. LogSubmissionAPIがLogServiceを呼び出し
3. LogServiceがLogRepositoryに生ログを保存
4. LogServiceがAIServiceに一次要約生成を依頼
5. AIServiceがBedrockを呼び出し
6. AIServiceが一次要約を返却
7. LogServiceが一次要約をクライアントに返却

---

### 3.2 サマリー確定〜最終生成

```
Client → LogSubmissionAPI → LogService → AIService → Bedrock
                                ↓            ↓
                          LogRepository  CharacterRepository
                                ↓
                          WebSocketHandler
                                ↓
                              Client
```

**シーケンス**:
1. クライアントが修正済みサマリーを送信（POST /api/logs/confirm）
2. LogSubmissionAPIがLogServiceを呼び出し
3. LogServiceがLogRepositoryにサマリーを保存
4. LogServiceがAIServiceに最終生成を依頼
5. AIServiceがCharacterRepositoryからキャラクター設定を取得
6. AIServiceがBedrockを呼び出し（上司日誌生成）
7. AIServiceがBedrockを呼び出し（甘やかしログ生成）
8. AIServiceが最終報告書を返却
9. LogServiceがLogRepositoryに最終報告書を保存
10. LogServiceがWebSocketHandlerを介してクライアントに処理完了通知を配信

---

### 3.3 リアルタイム通知生成

```
Client → WebSocketHandler → AIService → Bedrock
                                ↓
                          CharacterRepository
                                ↓
Client ← WebSocketHandler
```

**シーケンス**:
1. クライアントがWebSocket経由でコンテキストを送信
2. WebSocketHandlerがAIServiceに通知メッセージ生成を依頼
3. AIServiceがCharacterRepositoryからキャラクター設定を取得
4. AIServiceがBedrockを呼び出し
5. AIServiceが通知メッセージを返却
6. WebSocketHandlerがクライアントに通知を配信

---

### 3.4 ユーザー認証

```
Client → AuthenticationAPI → UserService → UserRepository
                                ↓
Client ← AuthenticationAPI ← UserService
```

**シーケンス**:
1. クライアントがユーザー名とパスワードを送信（POST /api/login）
2. AuthenticationAPIがUserServiceを呼び出し
3. UserServiceがUserRepositoryからユーザー情報を取得
4. UserServiceがパスワードを検証
5. UserServiceが認証結果を返却
6. AuthenticationAPIがセッションを作成
7. AuthenticationAPIが認証結果をクライアントに返却

---

## 4. オーケストレーションパターン

### 4.1 同期処理パターン
**使用場面**: ログ送信、サマリー確定、認証等
**特徴**: クライアントがレスポンスを待つ

```python
# LogServiceの例
def process_log_submission(self, user_id: int, structured_data: Dict[str, Any]) -> List[SummaryItem]:
    # 1. ログを保存
    log_id = self.log_repository.save_log(user_id, structured_data)
    
    # 2. AI生成を依頼（同期）
    summary = self.ai_service.generate_primary_summary(structured_data)
    
    # 3. サマリーを返却
    return summary
```

---

### 4.2 非同期処理パターン
**使用場面**: 最終生成、リアルタイム通知
**特徴**: 処理完了をWebSocketで通知

```python
# LogServiceの例
def process_summary_confirmation(self, user_id: int, summary: List[SummaryItem]) -> bool:
    # 1. サマリーを保存
    self.log_repository.save_summary(user_id, summary)
    
    # 2. 非同期で最終生成を実行
    threading.Thread(target=self._generate_final_report_async, args=(user_id, summary)).start()
    
    # 3. 即座に成功を返却
    return True

def _generate_final_report_async(self, user_id: int, summary: List[SummaryItem]):
    # AI生成を実行
    report = self.ai_service.generate_final_report(user_id, summary)
    
    # 報告書を保存
    self.log_repository.save_report(user_id, datetime.now(), report)
    
    # WebSocketで通知
    self.websocket_handler.send_notification(user_id, {"type": "report_ready"})
```

---

### 4.3 トランザクション管理パターン
**使用場面**: データベース操作を伴う処理
**特徴**: ロールバック可能

```python
# LogServiceの例
def process_summary_confirmation(self, user_id: int, summary: List[SummaryItem]) -> bool:
    try:
        # トランザクション開始
        with self.log_repository.transaction():
            # サマリーを保存
            self.log_repository.save_summary(user_id, summary)
            
            # AI生成を実行
            report = self.ai_service.generate_final_report(user_id, summary)
            
            # 報告書を保存
            self.log_repository.save_report(user_id, datetime.now(), report)
            
            # コミット
            return True
    except Exception as e:
        # ロールバック
        logging.error(f"Error in process_summary_confirmation: {e}")
        return False
```

---

## 5. エラーハンドリング戦略

### 5.1 サービス層でのエラーハンドリング
各サービスは独自にエラーを処理し、適切なエラーメッセージを返却します。

```python
# AIServiceの例
def generate_primary_summary(self, structured_data: Dict[str, Any]) -> List[SummaryItem]:
    try:
        # Bedrock呼び出し
        result = self.call_bedrock(prompt, images)
        return self._parse_summary(result)
    except BotoClientError as e:
        logging.error(f"Bedrock API error: {e}")
        raise AIServiceException("AI生成に失敗しました。しばらくしてから再試行してください。")
    except Exception as e:
        logging.error(f"Unexpected error: {e}")
        raise AIServiceException("予期しないエラーが発生しました。")
```

### 5.2 API層でのエラーハンドリング
API層はサービス層からの例外をキャッチし、適切なHTTPステータスコードを返却します。

```python
# LogSubmissionAPIの例
@app.route('/api/logs/submit', methods=['POST'])
def submit_logs(self):
    try:
        data = request.json
        user_id = session['user_id']
        summary = self.log_service.process_log_submission(user_id, data)
        return jsonify({"summary": summary}), 200
    except AIServiceException as e:
        return jsonify({"error": str(e)}), 500
    except Exception as e:
        logging.error(f"Unexpected error: {e}")
        return jsonify({"error": "サーバーエラーが発生しました。"}), 500
```

---

## 6. パフォーマンス最適化

### 6.1 キャッシング
- キャラクター設定をメモリキャッシュ（頻繁にアクセスされるため）
- ユーザー設定をメモリキャッシュ

### 6.2 非同期処理
- 最終生成は非同期で実行（処理時間が長いため）
- WebSocketで処理完了を通知

### 6.3 バッチ処理
- 複数のBedrock呼び出しを並列実行（上司日誌と甘やかしログを同時生成）

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
