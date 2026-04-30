# 要件確認質問

以下の質問に回答して、要件を明確化してください。各質問の [Answer]: タグの後に、選択肢の文字（A、B、C等）を記入してください。

---

## Question 1: データベース接続方式
AWS RDS (PostgreSQL)への接続方式を教えてください。

A) クライアントとサーバーの両方が直接RDSに接続する
B) クライアントはサーバー経由でのみデータベースにアクセスする（推奨）
C) クライアントは読み取り専用でRDSに直接接続、書き込みはサーバー経由
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 2: AWS Bedrock認証方式
AWS Bedrockへのアクセス認証方式を教えてください。

A) サーバー側でIAMロールを使用（EC2/ECS等で実行）
B) サーバー側でアクセスキー/シークレットキーを使用
C) クライアント側でも直接Bedrockにアクセス
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 3: 生ログの保存場所
クライアント側で収集した生ログ（スクショ、音声等）の一時保存場所を教えてください。

A) ローカルディスクの専用フォルダ（例：C:\Users\{user}\AppData\Local\amayakashi-log\）
B) ローカルディスクのユーザー指定フォルダ
C) メモリ上のみ（ディスクに保存しない）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 4: カメラ・マイクのプライバシー設定
カメラとマイクの使用について、初期設定はどうしますか？

A) デフォルトで有効（ユーザーが設定で無効化可能）
B) デフォルトで無効（ユーザーが設定で有効化する必要がある）
C) 初回起動時にユーザーに選択させる
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 5: サーバーのデプロイ環境
Flaskサーバーのデプロイ先を教えてください。

A) ローカル開発環境のみ（本番デプロイは対象外）
B) AWS EC2インスタンス
C) AWS ECS/Fargate（コンテナ）
D) AWS Lambda + API Gateway（サーバーレス）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 6: クライアント・サーバー間通信
クライアントとサーバー間の通信プロトコルを教えてください。

A) HTTP/HTTPS REST API
B) WebSocket（リアルタイム通信）
C) gRPC
D) REST API + WebSocket（通知用）
X) Other (please describe after [Answer]: tag below)

[Answer]: D

---

## Question 7: AIキャラクターマスタの初期データ
AIアシスタントキャラクターの初期データはどうしますか？

A) システムに3名のデフォルトキャラクターを組み込む
B) ユーザーが初回起動時に自分で作成する
C) サンプルキャラクターを提供し、ユーザーがカスタマイズ可能
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 8: Whisper音声認識の実行場所
OpenAI Whisperによる音声認識はどこで実行しますか？

A) クライアント側でローカル実行（Whisperモデルをローカルにダウンロード）
B) サーバー側で実行（音声データをサーバーに送信）
C) OpenAI APIを使用（クラウド実行）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 9: ユーザー認証機能
複数ユーザーの利用を想定していますか？

A) 単一ユーザー専用（認証機能不要）
B) 複数ユーザー対応（ユーザー名/パスワード認証）
C) 複数ユーザー対応（Windows統合認証）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 10: 開発優先順位
最初のバージョン（MVP）で実装する機能の優先順位を教えてください。

A) 全機能を最初から実装する
B) コア機能のみ実装（ログ取得 + 業務日誌生成 + 基本的なAI褒め機能）
C) 段階的実装（Phase 1: ログ取得と日誌生成、Phase 2: AI褒め機能、Phase 3: リアルタイム通知）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question: Security Extensions
このプロジェクトにセキュリティ拡張ルールを適用しますか？

A) Yes — すべてのセキュリティルールをブロッキング制約として適用（本番グレードアプリケーション推奨）
B) No — すべてのセキュリティルールをスキップ（PoC、プロトタイプ、実験的プロジェクト向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question: Property-Based Testing Extension
このプロジェクトにプロパティベーステスト（PBT）ルールを適用しますか？

A) Yes — すべてのPBTルールをブロッキング制約として適用（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを持つプロジェクト推奨）
B) Partial — 純粋関数とシリアライゼーションのラウンドトリップのみPBTルールを適用（アルゴリズム複雑度が限定的なプロジェクト向け）
C) No — すべてのPBTルールをスキップ（シンプルなCRUDアプリケーション、UIのみのプロジェクト、重要なビジネスロジックのない薄い統合レイヤー向け）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

**回答方法**: 各質問の [Answer]: の後に、選択した選択肢の文字（A、B、C、D、X）を記入してください。選択肢に該当しない場合は X を選択し、その後に説明を記述してください。

すべての質問に回答したら、「完了しました」または「done」とお知らせください。
