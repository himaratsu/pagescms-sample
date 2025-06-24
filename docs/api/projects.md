---
layout: default
title: プロジェクトAPI
parent: APIリファレンス
nav_order: 2
description: "プロジェクト管理に関するAPIエンドポイント"
---

# プロジェクトAPI
{: .no_toc }

プロジェクトの作成、取得、更新、削除に関するAPIエンドポイントです。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 認証

すべてのプロジェクトAPIエンドポイントはAPIキーによる認証が必要です。

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

## エンドポイント一覧

### 全プロジェクト取得

ユーザーがアクセス可能なすべてのプロジェクトのリストを取得します。

```http
GET /projects
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `page` | integer | いいえ | ページ番号（デフォルト: 1） |
| `limit` | integer | いいえ | ページあたりの結果数（デフォルト: 20、最大: 100） |
| `status` | string | いいえ | プロジェクトステータス（active, inactive, archived） |
| `owner_id` | string | いいえ | オーナーのユーザーID |
| `search` | string | いいえ | プロジェクト名で検索 |

**リクエスト例:**

```bash
curl -X GET "https://api.example.com/v1/projects?status=active&limit=5" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**レスポンス例:**

```json
{
  "data": [
    {
      "id": "proj_123",
      "name": "Webアプリケーション開発",
      "description": "新しいWebアプリケーションの開発プロジェクト",
      "status": "active",
      "owner_id": "user_123",
      "team_members": ["user_123", "user_456", "user_789"],
      "created_at": "2023-06-20T10:30:00Z",
      "updated_at": "2023-06-21T14:15:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 5,
    "total": 12,
    "pages": 3
  }
}
```

### 単一プロジェクト取得

IDを指定して特定のプロジェクトを取得します。

```http
GET /projects/{id}
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `id` | string | はい | プロジェクトID |

**リクエスト例:**

```bash
curl -X GET "https://api.example.com/v1/projects/proj_123" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**レスポンス例:**

```json
{
  "data": {
    "id": "proj_123",
    "name": "Webアプリケーション開発",
    "description": "新しいWebアプリケーションの開発プロジェクト",
    "status": "active",
    "owner": {
      "id": "user_123",
      "name": "山田太郎",
      "email": "yamada@example.com"
    },
    "team_members": [
      {
        "id": "user_123",
        "name": "山田太郎",
        "role": "owner"
      },
      {
        "id": "user_456",
        "name": "佐藤花子",
        "role": "developer"
      }
    ],
    "settings": {
      "visibility": "private",
      "notifications": true,
      "auto_backup": true
    },
    "stats": {
      "tasks_total": 25,
      "tasks_completed": 18,
      "tasks_pending": 7
    },
    "created_at": "2023-06-20T10:30:00Z",
    "updated_at": "2023-06-21T14:15:00Z"
  }
}
```

### プロジェクト作成

新しいプロジェクトを作成します。

```http
POST /projects
```

**リクエストボディ:**

| フィールド | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `name` | string | はい | プロジェクト名（2-100文字） |
| `description` | string | いいえ | プロジェクトの説明 |
| `visibility` | string | いいえ | 可視性（public, private）デフォルト: private |
| `team_members` | array | いいえ | チームメンバーのユーザーIDの配列 |

**リクエスト例:**

```bash
curl -X POST "https://api.example.com/v1/projects" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "モバイルアプリ開発",
    "description": "新しいモバイルアプリケーションの開発",
    "visibility": "private",
    "team_members": ["user_456", "user_789"]
  }'
```

**レスポンス例:**

```json
{
  "data": {
    "id": "proj_456",
    "name": "モバイルアプリ開発",
    "description": "新しいモバイルアプリケーションの開発",
    "status": "active",
    "owner_id": "user_123",
    "created_at": "2023-06-24T15:30:00Z",
    "updated_at": "2023-06-24T15:30:00Z"
  }
}
```

### プロジェクト更新

既存のプロジェクト情報を更新します。

```http
PUT /projects/{id}
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `id` | string | はい | プロジェクトID |

**リクエストボディ:**

```json
{
  "name": "高度なWebアプリケーション開発",
  "description": "React と Node.js を使用した高度なWebアプリケーション",
  "status": "active",
  "settings": {
    "notifications": false,
    "auto_backup": true
  }
}
```

### プロジェクト削除

プロジェクトを削除します。

```http
DELETE /projects/{id}
```

**パラメータ:**

| パラメータ | タイプ | 必須 | 説明 |
|:----------|:-----|:-----|:------------|
| `id` | string | はい | プロジェクトID |

**レスポンス例:**

```json
{
  "message": "プロジェクトが正常に削除されました"
}
```

## チームメンバー管理

### メンバー追加

プロジェクトにチームメンバーを追加します。

```http
POST /projects/{id}/members
```

**リクエストボディ:**

```json
{
  "user_id": "user_999",
  "role": "developer"
}
```

### メンバー削除

プロジェクトからチームメンバーを削除します。

```http
DELETE /projects/{id}/members/{user_id}
```

## エラーレスポンス

プロジェクトAPI固有のエラーコード:

| エラーコード | HTTPステータス | 説明 |
|:------------|:-------------|:------------|
| PROJECT_NOT_FOUND | 404 | プロジェクトが見つかりません |
| PROJECT_ACCESS_DENIED | 403 | プロジェクトへのアクセスが拒否されました |
| INVALID_PROJECT_NAME | 400 | プロジェクト名が無効です |
| MEMBER_ALREADY_EXISTS | 400 | メンバーは既にプロジェクトに参加しています |
| MEMBER_NOT_FOUND | 404 | 指定されたメンバーが見つかりません |

**エラーレスポンス例:**

```json
{
  "error": {
    "code": "PROJECT_ACCESS_DENIED",
    "message": "このプロジェクトにアクセスする権限がありません",
    "details": {
      "project_id": "proj_123",
      "required_role": "member"
    }
  }
}
```