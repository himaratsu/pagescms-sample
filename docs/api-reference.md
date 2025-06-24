---
layout: default
title: APIリファレンス
nav_order: 3
description: "実例付きの完全なAPIドキュメント"
---

# APIリファレンス
{: .no_toc }

実例とレスポンス形式を含む、すべてのAPIエンドポイントの完全なリファレンスです。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 認証

すべてのAPIリクエストはAPIキーを使用した認証が必要です。リクエストヘッダーにAPIキーを含めてください:

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

## ベースURL

すべてのAPIエンドポイントは、ベースURLに対する相対URLです:

```
https://api.example.com/v1
```

## ユーザーAPI

### 全ユーザー取得

すべてのユーザーのリストを取得します。

```http
GET /users
```

**パラメータ:**

| パラメータ | タイプ | 説明 |
|:----------|:-----|:------------|
| `page` | integer | ページ番号（デフォルト: 1） |
| `limit` | integer | ページあたりの結果数（デフォルト: 20、最大: 100） |
| `sort` | string | ソートフィールド（name, email, created_at） |
| `order` | string | ソート順（asc, desc） |

**リクエスト例:**

```bash
curl -X GET "https://api.example.com/v1/users?page=1&limit=10" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**レスポンス例:**

```json
{
  "data": [
    {
      "id": "user_123",
      "name": "山田太郎",
      "email": "yamada@example.com",
      "created_at": "2023-06-20T10:30:00Z",
      "updated_at": "2023-06-21T14:15:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 150,
    "pages": 15
  }
}
```

### 単一ユーザー取得

IDを指定して特定のユーザーを取得します。

```http
GET /users/{id}
```

**パラメータ:**

| パラメータ | タイプ | 説明 |
|:----------|:-----|:------------|
| `id` | string | ユーザーID |

**リクエスト例:**

```bash
curl -X GET "https://api.example.com/v1/users/user_123" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**レスポンス例:**

```json
{
  "data": {
    "id": "user_123",
    "name": "山田太郎",
    "email": "yamada@example.com",
    "profile": {
      "avatar": "https://example.com/avatars/yamada.jpg",
      "bio": "ソフトウェア開発者"
    },
    "created_at": "2023-06-20T10:30:00Z",
    "updated_at": "2023-06-21T14:15:00Z"
  }
}
```

### ユーザー作成

新しいユーザーを作成します。

```http
POST /users
```

**リクエストボディ:**

```json
{
  "name": "佐藤花子",
  "email": "sato@example.com",
  "password": "secure_password_123"
}
```

**リクエスト例:**

```bash
curl -X POST "https://api.example.com/v1/users" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "佐藤花子",
    "email": "sato@example.com",
    "password": "secure_password_123"
  }'
```

## プロジェクトAPI

### 全プロジェクト取得

すべてのプロジェクトのリストを取得します。

```http
GET /projects
```

**レスポンス例:**

```json
{
  "data": [
    {
      "id": "proj_456",
      "name": "素晴らしいプロジェクト",
      "description": "プロジェクトの説明",
      "status": "active",
      "owner_id": "user_123",
      "created_at": "2023-06-20T10:30:00Z"
    }
  ]
}
```

## エラーレスポンス

APIは標準的なHTTPレスポンスコードを使用します:

| ステータスコード | 説明 |
|:------------|:------------|
| 200 | OK - リクエスト成功 |
| 201 | Created - リソース作成成功 |
| 400 | Bad Request - 無効なリクエストパラメータ |
| 401 | Unauthorized - 無効または欠如したAPIキー |
| 403 | Forbidden - 権限不足 |
| 404 | Not Found - リソースが見つからない |
| 429 | Too Many Requests - レート制限超過 |
| 500 | Internal Server Error - サーバーエラー |

**エラーレスポンス形式:**

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "リクエストパラメータが無効です",
    "details": {
      "field": "email",
      "issue": "このメールアドレスは既に使用されています"
    }
  }
}
```

## レート制限

APIリクエストは、APIキーあたり1時間に1000リクエストまでに制限されています。レート制限情報はレスポンスヘッダーに含まれています:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1623456789
```