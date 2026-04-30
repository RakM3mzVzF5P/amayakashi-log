# Unit of Work依存関係

**プロジェクト**: amayakashi-log  
**作成日**: 2026-04-30

---

## 1. Unit依存関係マトリックス

| Unit | 依存先Unit | 依存理由 |
|---|---|---|
| Unit 1: 基盤・E2E最小実装 | なし | 基盤Unit |
| Unit 2: クライアント - ログ収集 | Unit 1 | 共有コンポーネント、LocalStorageManager、SettingsManager |
| Unit 3: クライアント - UI | Unit 1, Unit 2 | 共有コンポーネント、RESTClient、LogCollectorOrchestrator、PrivacyManager |
| Unit 4: サーバー - 認証・ユーザー管理 | Unit 1 | 共有コンポーネント、UserRepository、CharacterRepository |
| Unit 5: サーバー - ログ処理 | Unit 1, Unit 4 | 共有コンポーネント、LogRepository、AIService（モック）、UserService |
| Unit 6: サーバー - AI統合（実装） | Unit 1, Unit 5 | 共有コンポーネント、CharacterRepository、LogService |
| Unit 7: サーバー - Web UI | Unit 1, Unit 4, Unit 5 | 共有コンポーネント、AuthenticationAPI、ReportAPI、LogService |
| Unit 8: 統合テスト | Unit 1〜7 | すべてのUnit |

---

## 2. 依存関係図

```
Unit 1: 基盤・E2E最小実装
   ↓
   ├─→ Unit 2: クライアント - ログ収集
   │      ↓
   │   Unit 3: クライアント - UI
   │
   ├─→ Unit 4: サーバー - 認証・ユーザー管理
   │      ↓
   │   Unit 5: サーバー - ログ処理
   │      ↓
   │   Unit 6: サーバー - AI統合（実装）
   │
   └─→ Unit 7: サーバー - Web UI
          ↓
       Unit 8: 統合テスト
```

---

## 3. 実装順序の推奨

### Phase 1: 基盤構築（2週間）
1. **Unit 1: 基盤・E2E最小実装**
   - 共有コンポーネント、データベース、最小限のE2E機能

### Phase 2: コア機能実装（5週間）
2. **Unit 2: クライアント - ログ収集**（2週間）
   - マルチモーダルログ収集

3. **Unit 4: サーバー - 認証・ユーザー管理**（1週間）
   - ユーザー認証、設定管理

4. **Unit 5: サーバー - ログ処理**（2週間）
   - ログ処理、モックAI生成

### Phase 3: UI・AI統合（4週間）
5. **Unit 3: クライアント - UI**（2週間）
   - デスクトップウィジェット、サマリーエディター、設定画面

6. **Unit 6: サーバー - AI統合（実装）**（1週間）
   - AWS Bedrock統合

7. **Unit 7: サーバー - Web UI**（1週間）
   - Web管理画面

### Phase 4: テスト・品質保証（1週間）
8. **Unit 8: 統合テスト**（1週間）
   - E2Eテスト、統合テスト、パフォーマンステスト

**合計期間**: 約12週間（3ヶ月）

---

## 4. 並行開発の可能性

### 並行可能なUnit
- **Unit 2とUnit 4**: 依存関係がないため並行開発可能
- **Unit 3とUnit 6**: Unit 5完了後、並行開発可能
- **Unit 6とUnit 7**: Unit 5完了後、並行開発可能

### 並行開発時の注意点
- 共有コンポーネント（Unit 1）の変更は慎重に行う
- データベーススキーマの変更は全Unitに影響するため、事前調整が必要

---

## 5. 依存関係の理由

### Unit 1への依存
すべてのUnitがUnit 1に依存する理由：
- 共有コンポーネント（データモデル、ユーティリティ）の利用
- データベーススキーマの利用
- 基本的な通信コンポーネント（RESTClient等）の利用

### Unit 2への依存（Unit 3）
Unit 3がUnit 2に依存する理由：
- LogCollectorOrchestratorの利用（UI から制御）
- PrivacyManagerの拡張（ブラックリスト機能追加）

### Unit 4への依存（Unit 5, Unit 7）
Unit 5とUnit 7がUnit 4に依存する理由：
- ユーザー認証機能の利用
- セッション管理の利用

### Unit 5への依存（Unit 6, Unit 7）
Unit 6とUnit 7がUnit 5に依存する理由：
- LogServiceの利用（AI生成呼び出し、報告書取得）

---

## 6. 循環依存の回避

### 設計原則
- Unit間の依存関係は一方向のみ
- 循環依存は禁止
- 必要に応じてインターフェースを導入

### 検証結果
✅ 循環依存なし - すべてのUnit依存関係は一方向

---

**文書作成日**: 2026-04-30  
**作成者**: AI-DLC System  
**バージョン**: 1.0
