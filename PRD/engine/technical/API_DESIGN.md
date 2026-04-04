# API設計 (API Design)

本ドキュメントでは、Isidoricaの内部APIの設計を定義する。

**ステータス**: ドラフト（Phase 4で実装予定）

---

## 1. 概要

### 1.1 目的

*   執筆支援ツール（Web検索ページ、Webエディタ、VSCode拡張、CLI）が使用するAPI
*   出典の検索、登録、衝突チェックを提供

### 1.2 技術スタック

| 項目 | 技術 |
| :--- | :--- |
| **ランタイム** | Cloudflare Workers |
| **データストア** | Cloudflare KV（bibliography.json のキャッシュ） |
| **認証** | 読み取りは公開、書き込みはGitHub OAuth |

---

## 2. エンドポイント一覧

| メソッド | パス | 目的 |
| :--- | :--- | :--- |
| `GET` | `/api/sources` | 既存出典を検索 |
| `GET` | `/api/sources/ids` | IDリストを取得（衝突チェック用） |
| `GET` | `/api/sources/:id` | 特定の出典を取得 |
| `POST` | `/api/sources` | 新規出典を追加 |
| `GET` | `/api/external/search` | 外部API（Google Books, CiNii, CrossRef）を検索 |

---

## 3. エンドポイント詳細

### 3.1 GET /api/sources

既存出典を検索する。

#### リクエスト

```
GET /api/sources?q=ハイデガー&limit=20
```

| パラメータ | 必須 | 説明 |
| :--- | :--- | :--- |
| `q` | ✓ | 検索クエリ（著者名、タイトル等） |
| `limit` | - | 最大件数（デフォルト: 20） |

#### レスポンス

```json
{
  "results": [
    {
      "id": "heidegger-being-time-1962",
      "title": "Being and Time",
      "author": [{"family": "Heidegger", "given": "Martin"}],
      "year": 1962,
      "language": "en",
      "status": "curated"
    }
  ],
  "total": 1
}
```

### 3.2 GET /api/sources/ids

IDリストを取得する（衝突チェック用）。

#### リクエスト

```
GET /api/sources/ids
```

#### レスポンス

```json
{
  "ids": [
    "heidegger-being-time-1962",
    "tanaka-game-theory-2020",
    "wang-economics-2021"
  ],
  "count": 3
}
```

*   **用途**: 執筆支援ツールが起動時にロードし、メモリに保持
*   **軽量**: IDのみを返す（全データではない）

### 3.3 GET /api/sources/:id

特定の出典を取得する。

#### リクエスト

```
GET /api/sources/heidegger-being-time-1962
```

#### レスポンス

```json
{
  "id": "heidegger-being-time-1962",
  "title": "Being and Time",
  "author": [{"family": "Heidegger", "given": "Martin"}],
  "year": 1962,
  "language": "en",
  "status": "curated",
  "translations": {
    "ja": "heidegger-being-time-ja-2003"
  }
}
```

### 3.4 POST /api/sources

新規出典を追加する。

#### リクエスト

```
POST /api/sources
Content-Type: application/json
Authorization: Bearer {github_token}
```

```json
{
  "title": "ゲーム理論入門",
  "author": [{"family": "田中", "given": "太郎"}],
  "year": 2020,
  "language": "ja",
  "ISBN": "978-4-1234-5678-9"
}
```

#### レスポンス

```json
{
  "id": "tanaka-game-theory-2020",
  "status": "auto",
  "created": true
}
```

*   **ID自動生成**: サーバーサイドで生成
*   **衝突チェック**: サーバーサイドで実施、衝突時はイニシャル/連番を追加
*   **認証必須**: GitHub OAuthトークン

### 3.5 GET /api/external/search

外部API（Google Books, CiNii, CrossRef）を検索する。

#### リクエスト

```
GET /api/external/search?q=Heidegger+Being+and+Time&provider=all
```

| パラメータ | 必須 | 説明 |
| :--- | :--- | :--- |
| `q` | ✓ | 検索クエリ |
| `provider` | - | `google`, `cinii`, `crossref`, `all`（デフォルト: all） |

#### レスポンス

```json
{
  "results": [
    {
      "provider": "google",
      "title": "Being and Time",
      "author": [{"family": "Heidegger", "given": "Martin"}],
      "year": 1962,
      "ISBN": "978-0-06-090832-6"
    }
  ],
  "total": 1
}
```

*   **プロキシ**: Cloudflare Workersが外部APIにリクエスト
*   **レート制限管理**: サーバーサイドで各APIのレート制限を管理

---

## 4. 認証

### 4.1 読み取りAPI

*   `GET /api/sources`
*   `GET /api/sources/ids`
*   `GET /api/sources/:id`
*   `GET /api/external/search`

→ **認証不要**（公開API）

### 4.2 書き込みAPI

*   `POST /api/sources`

→ **GitHub OAuth必須**

### 4.3 認証フロー

```
ユーザー: GitHub でログイン
    ↓
Isidorica: OAuth認証、アクセストークン取得
    ↓
ユーザー: 出典を追加
    ↓
執筆支援ツール: POST /api/sources（トークン付き）
    ↓
API: トークン検証 → リポジトリにコミット権限があるか確認
    ↓
API: bibliography.json を更新（PRを作成 or 直接コミット）
```

---

## 5. データ同期

### 5.1 bibliography.json と KV の同期

| イベント | 処理 |
| :--- | :--- |
| **ビルド時** | bibliography.json を KV に同期 |
| **POST /api/sources** | KV を更新 + GitHubにPR作成 |
| **PRマージ時** | GitHub Webhook → KV を更新 |

### 5.2 競合の回避

*   **POST /api/sources** はPRを作成（直接マージしない）
*   PRレビュー → マージ → KV更新
*   同時作業による競合はPRレベルで解決

---

## 6. エラーハンドリング

| ステータス | 意味 |
| :--- | :--- |
| `200` | 成功 |
| `400` | リクエスト不正（バリデーションエラー） |
| `401` | 認証必要 |
| `403` | 権限なし |
| `404` | 出典が見つからない |
| `409` | ID衝突（自動解決できなかった場合） |
| `429` | レート制限超過 |
| `500` | サーバーエラー |

---

## 7. 将来の拡張

*   `PATCH /api/sources/:id` - 出典の更新
*   `DELETE /api/sources/:id` - 出典の削除（deprecated化）
*   `GET /api/sources/:id/translations` - 翻訳関係の取得
*   `POST /api/sources/:id/translations` - 翻訳関係の設定
