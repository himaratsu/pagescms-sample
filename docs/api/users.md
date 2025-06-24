---
layout: default
title: ユーザーAPI
parent: APIリファレンス
nav_order: 1
description: "ユーザー管理に関するAPIエンドポイント"
---

# ユーザーAPI
{: .no_toc }

ユーザーの作成、取得、更新、削除に関するAPIエンドポイントです。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 認証

すべてのユーザーAPIエンドポイントはAPIキーによる認証が必要です。

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

## エンドポイント一覧

### 全ユーザー取得

すべてのユーザーのリストを取得します。

```http
GET /users
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `page` | integer | いいえ | ページ番号（デフォルト: 1） |
| `limit` | integer | いいえ | ページあたりの結果数（デフォルト: 20、最大: 100） |
| `sort` | string | いいえ | ソートフィールド（name, email, created_at） |
| `order` | string | いいえ | ソート順（asc, desc） |
| `search` | string | いいえ | 名前またはメールアドレスで検索 |

**リクエスト例:**

```bash
curl -X GET "https://api.example.com/v1/users?page=1&limit=10&search=yamada" \
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
      "status": "active",
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

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `id` | string | はい | ユーザーID |

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
    "status": "active",
    "profile": {
      "avatar": "https://example.com/avatars/yamada.jpg",
      "bio": "ソフトウェア開発者",
      "location": "東京, 日本"
    },
    "preferences": {
      "language": "ja",
      "timezone": "Asia/Tokyo",
      "notifications": true
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

| フィールド | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `name` | string | はい | ユーザー名（2-50文字） |
| `email` | string | はい | メールアドレス（有効な形式） |
| `password` | string | はい | パスワード（8文字以上） |
| `profile` | object | いいえ | プロフィール情報 |

**リクエスト例:**

```bash
curl -X POST "https://api.example.com/v1/users" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "佐藤花子",
    "email": "sato@example.com",
    "password": "secure_password_123",
    "profile": {
      "bio": "Webデザイナー",
      "location": "大阪, 日本"
    }
  }'
```

**レスポンス例:**

```json
{
  "data": {
    "id": "user_456",
    "name": "佐藤花子",
    "email": "sato@example.com",
    "status": "active",
    "created_at": "2023-06-24T15:30:00Z",
    "updated_at": "2023-06-24T15:30:00Z"
  }
}
```

### ユーザー更新

既存のユーザー情報を更新します。

```http
PUT /users/{id}
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `id` | string | はい | ユーザーID |

**リクエストボディ:**

```json
{
  "name": "山田次郎",
  "profile": {
    "bio": "シニアソフトウェア開発者",
    "location": "東京, 日本"
  },
  "preferences": {
    "notifications": false
  }
}
```

### ユーザー削除

ユーザーを削除します。

```http
DELETE /users/{id}
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `id` | string | はい | ユーザーID |

**レスポンス例:**

```json
{
  "message": "ユーザーが正常に削除されました"
}
```

## エラーレスポンス

ユーザーAPI固有のエラーコード:

| エラーコード | HTTPステータス | 説明 |
|:------------|:-------------|:------------|
| USER_NOT_FOUND | 404 | ユーザーが見つかりません |
| EMAIL_ALREADY_EXISTS | 400 | メールアドレスが既に使用されています |
| INVALID_PASSWORD | 400 | パスワードが要件を満たしていません |
| USER_INACTIVE | 403 | ユーザーアカウントが無効です |

**エラーレスポンス例:**

```json
{
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "このメールアドレスは既に使用されています",
    "details": {
      "field": "email",
      "value": "existing@example.com"
    }
  }
}
```