# データベーススキーマ設計書

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30  
**データベース**: AWS RDS (PostgreSQL)  
**正規化レベル**: 適度な正規化（2NF〜3NF）

---

## 1. スキーマ概要

amayakashi-logシステムのデータベーススキーマは、以下の主要テーブルで構成されます：

| テーブル名 | 説明 |
|---|---|
| users | ユーザー情報 |
| user_settings | ユーザー設定（キャラクター設定等） |
| characters | AIキャラクターマスタ |
| raw_logs | 生ログデータ（構造化データ） |
| summaries | 一次要約（サマリー） |
| reports | 最終報告書（上司日誌 + 甘やかしログ） |
| notifications | 通知履歴 |

---

## 2. テーブル定義

### 2.1 users（ユーザー情報）

**目的**: ユーザーアカウント情報を管理

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | ユーザーID（自動採番） |
| username | VARCHAR(50) | NOT NULL, UNIQUE | ユーザー名 |
| password_hash | VARCHAR(255) | NOT NULL | パスワードハッシュ |
| email | VARCHAR(100) | UNIQUE | メールアドレス（オプション） |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |
| last_login_at | TIMESTAMP | NULL | 最終ログイン日時 |
| is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | アクティブフラグ |

**インデックス**:
- PRIMARY KEY: id
- UNIQUE INDEX: username
- INDEX: email

**サンプルデータ**:
```sql
INSERT INTO users (username, password_hash, email) VALUES
('sato_kenta', '$2b$12$...', 'sato@example.com'),
('tanaka_misaki', '$2b$12$...', 'tanaka@example.com');
```

---

### 2.2 user_settings（ユーザー設定）

**目的**: ユーザーごとの設定を管理（キャラクター設定、推しキャラ等）

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | 設定ID（自動採番） |
| user_id | INTEGER | NOT NULL, FOREIGN KEY → users(id) | ユーザーID |
| favorite_character_id | INTEGER | NULL, FOREIGN KEY → characters(id) | 推しキャラクターID |
| settings_json | JSONB | NOT NULL, DEFAULT '{}' | その他の設定（JSON形式） |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

**インデックス**:
- PRIMARY KEY: id
- UNIQUE INDEX: user_id
- FOREIGN KEY: user_id → users(id) ON DELETE CASCADE
- FOREIGN KEY: favorite_character_id → characters(id) ON DELETE SET NULL

**settings_json構造例**:
```json
{
  "notification_enabled": true,
  "notification_frequency": "medium",
  "theme": "light"
}
```

---

### 2.3 characters（AIキャラクターマスタ）

**目的**: AIキャラクターの設定を管理

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | キャラクターID（自動採番） |
| name | VARCHAR(50) | NOT NULL | キャラクター名 |
| personality | TEXT | NOT NULL | 性格設定 |
| tone | VARCHAR(50) | NOT NULL | 口調（例：丁寧、フランク、ツンデレ） |
| prompt_template | TEXT | NOT NULL | プロンプトテンプレート |
| icon_url | VARCHAR(255) | NULL | アイコン画像URL |
| is_default | BOOLEAN | NOT NULL, DEFAULT FALSE | デフォルトキャラクターフラグ |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

**インデックス**:
- PRIMARY KEY: id
- INDEX: is_default

**サンプルデータ**:
```sql
INSERT INTO characters (name, personality, tone, prompt_template, is_default) VALUES
('優しいお姉さん', '優しく包容力がある', '丁寧', 'あなたは優しいお姉さんです。ユーザーを温かく見守り、優しく褒めてください。', TRUE),
('元気な後輩', '明るく元気で前向き', 'フランク', 'あなたは元気な後輩です。ユーザーを明るく励まし、元気に褒めてください。', TRUE),
('クールな同僚', 'クールで冷静だが実は優しい', 'ツンデレ', 'あなたはクールな同僚です。普段は冷静ですが、ユーザーの頑張りを認めて褒めてください。', TRUE);
```

---

### 2.4 raw_logs（生ログデータ）

**目的**: クライアントから送信された構造化データを保存

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | ログID（自動採番） |
| user_id | INTEGER | NOT NULL, FOREIGN KEY → users(id) | ユーザーID |
| log_date | DATE | NOT NULL | ログ日付 |
| start_time | TIMESTAMP | NOT NULL | 記録開始時刻 |
| end_time | TIMESTAMP | NOT NULL | 記録終了時刻 |
| structured_data | JSONB | NOT NULL | 構造化データ（JSON形式） |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |

**インデックス**:
- PRIMARY KEY: id
- INDEX: (user_id, log_date)
- FOREIGN KEY: user_id → users(id) ON DELETE CASCADE

**structured_data構造例**:
```json
{
  "typing_logs": [
    {"time": "09:00:00", "text": "メール作成中..."}
  ],
  "window_logs": [
    {"window_title": "Excel - 資料.xlsx", "duration_seconds": 3600}
  ],
  "activity_logs": [
    {"type": "active", "start": "09:00:00", "end": "12:00:00"},
    {"type": "break", "start": "12:00:00", "end": "13:00:00"}
  ],
  "screenshots": [
    {"path": "/path/to/screenshot1.png", "timestamp": "09:30:00"}
  ],
  "camera_images": [
    {"path": "/path/to/camera1.jpg", "timestamp": "09:30:00"}
  ]
}
```

---

### 2.5 summaries（一次要約）

**目的**: AI生成された一次要約（サマリー）を保存

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | サマリーID（自動採番） |
| user_id | INTEGER | NOT NULL, FOREIGN KEY → users(id) | ユーザーID |
| raw_log_id | INTEGER | NOT NULL, FOREIGN KEY → raw_logs(id) | 生ログID |
| log_date | DATE | NOT NULL | ログ日付 |
| summary_items | JSONB | NOT NULL | サマリー項目（JSON配列） |
| is_confirmed | BOOLEAN | NOT NULL, DEFAULT FALSE | 確定フラグ |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新日時 |

**インデックス**:
- PRIMARY KEY: id
- INDEX: (user_id, log_date)
- FOREIGN KEY: user_id → users(id) ON DELETE CASCADE
- FOREIGN KEY: raw_log_id → raw_logs(id) ON DELETE CASCADE

**summary_items構造例**:
```json
[
  {
    "time": "09:00",
    "activity": "メールチェックと返信",
    "detail": "10件のメールに返信、重要案件2件を上司にエスカレーション"
  },
  {
    "time": "10:00",
    "activity": "顧客向け資料作成",
    "detail": "Excel資料を作成、3000文字入力、グラフ5個作成"
  }
]
```

---

### 2.6 reports（最終報告書）

**目的**: AI生成された最終報告書（上司日誌 + 甘やかしログ）を保存

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | 報告書ID（自動採番） |
| user_id | INTEGER | NOT NULL, FOREIGN KEY → users(id) | ユーザーID |
| summary_id | INTEGER | NOT NULL, FOREIGN KEY → summaries(id) | サマリーID |
| log_date | DATE | NOT NULL | ログ日付 |
| manager_report | TEXT | NOT NULL | 上司日誌 |
| comfort_log | JSONB | NOT NULL | 甘やかしログ（チャット形式） |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |

**インデックス**:
- PRIMARY KEY: id
- UNIQUE INDEX: (user_id, log_date)
- FOREIGN KEY: user_id → users(id) ON DELETE CASCADE
- FOREIGN KEY: summary_id → summaries(id) ON DELETE CASCADE

**comfort_log構造例**:
```json
{
  "messages": [
    {
      "character_id": 1,
      "character_name": "優しいお姉さん",
      "message": "今日のあなたのタイピング、まるでピアノを弾いているみたいに優雅だったわ",
      "timestamp": "18:06:00"
    },
    {
      "character_id": 2,
      "character_name": "元気な後輩",
      "message": "あの会議の時、一瞬見せた真剣な横顔にドキッとしちゃいました！",
      "timestamp": "18:06:05"
    },
    {
      "character_id": 3,
      "character_name": "クールな同僚",
      "message": "...まあ、今日は頑張ってたみたいね。認めてあげるわ。",
      "timestamp": "18:06:10"
    }
  ]
}
```

---

### 2.7 notifications（通知履歴）

**目的**: リアルタイム通知の履歴を保存

| カラム名 | データ型 | 制約 | 説明 |
|---|---|---|---|
| id | SERIAL | PRIMARY KEY | 通知ID（自動採番） |
| user_id | INTEGER | NOT NULL, FOREIGN KEY → users(id) | ユーザーID |
| character_id | INTEGER | NULL, FOREIGN KEY → characters(id) | キャラクターID |
| notification_type | VARCHAR(50) | NOT NULL | 通知タイプ（return, long_focus, meeting_end等） |
| message | TEXT | NOT NULL | 通知メッセージ |
| context | JSONB | NULL | コンテキスト情報 |
| is_read | BOOLEAN | NOT NULL, DEFAULT FALSE | 既読フラグ |
| created_at | TIMESTAMP | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 作成日時 |

**インデックス**:
- PRIMARY KEY: id
- INDEX: (user_id, created_at)
- FOREIGN KEY: user_id → users(id) ON DELETE CASCADE
- FOREIGN KEY: character_id → characters(id) ON DELETE SET NULL

**context構造例**:
```json
{
  "trigger": "return_from_break",
  "break_duration_minutes": 30,
  "previous_activity": "Excel作業"
}
```

---

## 3. リレーションシップ図

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

---

## 4. データ整合性制約

### 4.1 外部キー制約
- すべての外部キーにON DELETE CASCADEまたはON DELETE SET NULLを設定
- ユーザー削除時に関連データも削除（CASCADE）
- キャラクター削除時は参照をNULLに設定（SET NULL）

### 4.2 一意性制約
- users.username: UNIQUE
- users.email: UNIQUE
- user_settings.user_id: UNIQUE（1ユーザー1設定）
- reports.(user_id, log_date): UNIQUE（1日1報告書）

### 4.3 NOT NULL制約
- 必須項目にはNOT NULL制約を設定
- オプション項目はNULL許可

---

## 5. インデックス戦略

### 5.1 主キーインデックス
すべてのテーブルにSERIAL型の主キー（id）を設定し、自動的にインデックスが作成されます。

### 5.2 外部キーインデックス
外部キー列にインデックスを作成し、JOIN性能を向上させます。

### 5.3 検索用インデックス
- (user_id, log_date): 日付範囲検索用
- (user_id, created_at): 時系列検索用

### 5.4 JSONB列のインデックス
```sql
-- structured_dataのGINインデックス（JSON検索用）
CREATE INDEX idx_raw_logs_structured_data ON raw_logs USING GIN (structured_data);

-- summary_itemsのGINインデックス
CREATE INDEX idx_summaries_summary_items ON summaries USING GIN (summary_items);

-- comfort_logのGINインデックス
CREATE INDEX idx_reports_comfort_log ON reports USING GIN (comfort_log);
```

---

## 6. データ型とサイズ

### 6.1 文字列型
- VARCHAR(50): 短い文字列（名前、タイプ等）
- VARCHAR(100): 中程度の文字列（メールアドレス等）
- VARCHAR(255): 長い文字列（URL、パス等）
- TEXT: 無制限の文字列（本文、プロンプト等）

### 6.2 数値型
- SERIAL: 自動採番（主キー）
- INTEGER: 整数（外部キー、カウント等）
- BOOLEAN: 真偽値（フラグ）

### 6.3 日時型
- DATE: 日付のみ
- TIMESTAMP: 日時（タイムゾーンなし）

### 6.4 JSON型
- JSONB: バイナリJSON（検索・インデックス可能）

---

## 7. パーティショニング戦略

### 7.1 raw_logsテーブルのパーティショニング
大量のログデータを効率的に管理するため、log_dateでパーティショニングを検討：

```sql
-- 月次パーティショニング例
CREATE TABLE raw_logs_2026_04 PARTITION OF raw_logs
FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');

CREATE TABLE raw_logs_2026_05 PARTITION OF raw_logs
FOR VALUES FROM ('2026-05-01') TO ('2026-06-01');
```

### 7.2 reportsテーブルのパーティショニング
同様に、log_dateでパーティショニングを検討。

---

## 8. データ保持ポリシー

### 8.1 生ログの削除
- raw_logsテーブルのデータは1週間経過後に削除（要件定義書に基づく）
- 削除ジョブを定期実行（毎日深夜）

```sql
-- 1週間以上前のraw_logsを削除
DELETE FROM raw_logs WHERE created_at < NOW() - INTERVAL '7 days';
```

### 8.2 報告書の永続保存
- summariesとreportsテーブルのデータは永続保存
- 定期的なバックアップを推奨

---

## 9. セキュリティ

### 9.1 パスワードハッシュ化
- パスワードはbcryptでハッシュ化して保存
- ソルトを使用して強度を向上

### 9.2 データ暗号化
- RDSの暗号化機能を使用（at-rest encryption）
- SSL/TLS接続を使用（in-transit encryption）

---

## 10. バックアップとリカバリ

### 10.1 自動バックアップ
- RDSの自動バックアップ機能を有効化
- バックアップ保持期間: 7日間

### 10.2 スナップショット
- 定期的に手動スナップショットを作成
- 重要なマイルストーン前にスナップショット作成

---

## 11. DDL（テーブル作成SQL）

```sql
-- usersテーブル
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

-- charactersテーブル
CREATE TABLE characters (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    personality TEXT NOT NULL,
    tone VARCHAR(50) NOT NULL,
    prompt_template TEXT NOT NULL,
    icon_url VARCHAR(255),
    is_default BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- user_settingsテーブル
CREATE TABLE user_settings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL UNIQUE,
    favorite_character_id INTEGER,
    settings_json JSONB NOT NULL DEFAULT '{}',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (favorite_character_id) REFERENCES characters(id) ON DELETE SET NULL
);

-- raw_logsテーブル
CREATE TABLE raw_logs (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    log_date DATE NOT NULL,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    structured_data JSONB NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
CREATE INDEX idx_raw_logs_user_date ON raw_logs(user_id, log_date);
CREATE INDEX idx_raw_logs_structured_data ON raw_logs USING GIN (structured_data);

-- summariesテーブル
CREATE TABLE summaries (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    raw_log_id INTEGER NOT NULL,
    log_date DATE NOT NULL,
    summary_items JSONB NOT NULL,
    is_confirmed BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (raw_log_id) REFERENCES raw_logs(id) ON DELETE CASCADE
);
CREATE INDEX idx_summaries_user_date ON summaries(user_id, log_date);
CREATE INDEX idx_summaries_summary_items ON summaries USING GIN (summary_items);

-- reportsテーブル
CREATE TABLE reports (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    summary_id INTEGER NOT NULL,
    log_date DATE NOT NULL,
    manager_report TEXT NOT NULL,
    comfort_log JSONB NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (summary_id) REFERENCES summaries(id) ON DELETE CASCADE,
    UNIQUE (user_id, log_date)
);
CREATE INDEX idx_reports_comfort_log ON reports USING GIN (comfort_log);

-- notificationsテーブル
CREATE TABLE notifications (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    character_id INTEGER,
    notification_type VARCHAR(50) NOT NULL,
    message TEXT NOT NULL,
    context JSONB,
    is_read BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (character_id) REFERENCES characters(id) ON DELETE SET NULL
);
CREATE INDEX idx_notifications_user_created ON notifications(user_id, created_at);
```

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
