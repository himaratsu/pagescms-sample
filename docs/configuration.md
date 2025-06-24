---
layout: default
title: 設定ガイド
nav_order: 4
description: "セットアップと設定の手順"
---

# 設定ガイド
{: .no_toc }

さまざまな環境でアプリケーションを設定およびカスタマイズする方法を学びます。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 環境設定

### 開発環境

プロジェクトルートに `.env.development` ファイルを作成します:

```bash
# API設定
API_URL=http://localhost:8080/api/v1
API_KEY=dev_api_key_123
DEBUG=true

# データベース設定
DATABASE_URL=postgresql://user:password@localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379

# フィーチャーフラグ
ENABLE_ANALYTICS=false
ENABLE_CACHING=true
ENABLE_LOGGING=true
```

### 本番環境

本番環境では、これらの推奨設定を使用してください:

```bash
# API設定
API_URL=https://api.yourapp.com/v1
API_KEY=prod_api_key_secure_123
DEBUG=false

# データベース設定
DATABASE_URL=postgresql://user:password@prod-db:5432/myapp_prod
REDIS_URL=redis://prod-redis:6379

# フィーチャーフラグ
ENABLE_ANALYTICS=true
ENABLE_CACHING=true
ENABLE_LOGGING=true

# セキュリティ
SESSION_SECRET=your_super_secure_session_secret
JWT_SECRET=your_jwt_secret_key
CORS_ORIGIN=https://yourapp.com
```

## アプリケーション設定

### 基本設定

`config.json` ファイルを使用してアプリケーションを設定します:

```json
{
  "app": {
    "name": "私のアプリケーション",
    "version": "1.0.0",
    "port": 3000,
    "host": "localhost"
  },
  "database": {
    "type": "postgresql",
    "host": "localhost",
    "port": 5432,
    "database": "myapp",
    "username": "user",
    "password": "password",
    "ssl": false,
    "pool": {
      "min": 2,
      "max": 10
    }
  },
  "cache": {
    "type": "redis",
    "host": "localhost",
    "port": 6379,
    "ttl": 3600
  }
}
```

### 高度な設定

より高度な設定には、`config.advanced.json` を使用します:

```json
{
  "logging": {
    "level": "info",
    "format": "json",
    "file": "./logs/app.log",
    "maxFiles": 5,
    "maxSize": "10m"
  },
  "security": {
    "bcrypt": {
      "rounds": 12
    },
    "jwt": {
      "expiresIn": "7d",
      "algorithm": "HS256"
    },
    "cors": {
      "origin": ["http://localhost:3000"],
      "credentials": true
    }
  },
  "features": {
    "rateLimit": {
      "windowMs": 900000,
      "max": 100
    },
    "upload": {
      "maxFileSize": "10mb",
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}
```

## データベース設定

### PostgreSQLセットアップ

1. PostgreSQLをインストール:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install postgresql postgresql-contrib
   
   # macOS with Homebrew
   brew install postgresql
   ```

2. データベースとユーザーを作成:
   ```sql
   CREATE DATABASE myapp_dev;
   CREATE USER myapp_user WITH PASSWORD 'secure_password';
   GRANT ALL PRIVILEGES ON DATABASE myapp_dev TO myapp_user;
   ```

3. 環境変数を更新:
   ```bash
   DATABASE_URL=postgresql://myapp_user:secure_password@localhost:5432/myapp_dev
   ```

### Redisセットアップ

1. Redisをインストール:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install redis-server
   
   # macOS with Homebrew
   brew install redis
   ```

2. Redisサービスを開始:
   ```bash
   # Ubuntu/Debian
   sudo systemctl start redis-server
   
   # macOS
   brew services start redis
   ```

## Webサーバー設定

### Nginx設定

本番環境でのNginx設定例:

```nginx
server {
    listen 80;
    server_name yourapp.com www.yourapp.com;
    
    # HTTPからHTTPSへリダイレクト
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourapp.com www.yourapp.com;
    
    # SSL設定
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    # セキュリティヘッダー
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    
    # Node.jsアプリケーションへのプロキシ
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # 静的ファイル配信
    location /static/ {
        alias /path/to/static/files/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

## 監視設定

### アプリケーション監視

環境変数を使用してアプリケーション監視を設定:

```bash
# 監視
MONITORING_ENABLED=true
METRICS_PORT=9090
HEALTH_CHECK_ENDPOINT=/health

# APM（アプリケーションパフォーマンス監視）
APM_SERVICE_NAME=myapp
APM_SERVER_URL=https://apm.yourapp.com
APM_SECRET_TOKEN=your_apm_token
```

### ヘルスチェック設定

```json
{
  "healthCheck": {
    "timeout": 5000,
    "checks": [
      {
        "name": "database",
        "type": "postgresql",
        "critical": true
      },
      {
        "name": "redis",
        "type": "redis",
        "critical": false
      },
      {
        "name": "external-api",
        "type": "http",
        "url": "https://api.external.com/health",
        "critical": false
      }
    ]
  }
}
```

## 設定のトラブルシューティング

### よくある問題

1. **ポートが既に使用中**: 設定でポートを変更してください
2. **データベース接続失敗**: DATABASE_URLを確認してください
3. **Redis接続失敗**: Redisが動作していることを確認してください
4. **SSL証明書エラー**: 証明書のパスと権限を確認してください

### デバッグモード

詳細なログを出力するためにデバッグモードを有効にします:

```bash
DEBUG=true
LOG_LEVEL=debug
```

これにより、設定の読み込みとアプリケーションの起動に関する詳細情報が提供されます。