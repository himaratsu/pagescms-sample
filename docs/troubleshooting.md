---
layout: default
title: トラブルシューティング
nav_order: 5
description: "よくある問題と解決方法"
---

# トラブルシューティング
{: .no_toc }

遭遇する可能性のあるよくある問題と課題の解決策を見つけます。
{: .fs-6 .fw-300 }

## 目次
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## インストール問題

### Node.jsバージョンの問題

**問題**: Node.jsバージョンに関連するインストール中のエラー。

**解決策**:
1. Node.jsバージョンを確認:
   ```bash
   node --version
   ```
2. Node.js 14以上を使用していることを確認
3. nvmを使用してNode.jsバージョンを管理:
   ```bash
   # nvm をインストール（まだインストールしていない場合）
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
   
   # Node.js 18 をインストールして使用
   nvm install 18
   nvm use 18
   ```

### パッケージインストールの失敗

**問題**: `npm install` または `yarn install` が権限エラーで失敗する。

**解決策**:
1. npm/yarnキャッシュをクリア:
   ```bash
   npm cache clean --force
   # または
   yarn cache clean
   ```

2. `node_modules` とpackage-lockファイルを削除:
   ```bash
   rm -rf node_modules package-lock.json
   npm install
   ```

3. macOS/Linuxでの権限問題:
   ```bash
   sudo chown -R $(whoami) ~/.npm
   ```

## データベース接続問題

### PostgreSQL接続失敗

**問題**: PostgreSQLデータベースに接続できない。

**症状**:
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**解決策**:

1. **PostgreSQLが実行されているか確認**:
   ```bash
   # Ubuntu/Debian
   sudo systemctl status postgresql
   
   # macOS
   brew services list | grep postgresql
   ```

2. **PostgreSQLサービスを開始**:
   ```bash
   # Ubuntu/Debian
   sudo systemctl start postgresql
   
   # macOS
   brew services start postgresql
   ```

3. **接続設定を確認**:
   ```bash
   psql -h localhost -p 5432 -U your_username -d your_database
   ```

4. **ファイアウォール設定を確認**（リモートデータベースを使用している場合）:
   ```bash
   telnet your_db_host 5432
   ```

### データベースマイグレーションエラー

**問題**: 起動時にデータベースマイグレーションが失敗する。

**解決策**:
1. マイグレーションファイルの構文エラーを確認
2. データベースユーザーが適切な権限を持っていることを確認:
   ```sql
   GRANT ALL PRIVILEGES ON DATABASE your_db TO your_user;
   GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO your_user;
   ```
3. マイグレーションを手動で実行:
   ```bash
   npm run migrate
   # または
   yarn migrate
   ```

## API問題

### 認証エラー

**問題**: 401 Unauthorizedレスポンスが発生する。

**よくある原因と解決策**:

1. **無効なAPIキー**:
   - 環境変数のAPIキーを確認
   - 余分なスペースや文字がないか確認
   - キーが期限切れでないか確認

2. **不正なヘッダー形式**:
   ```javascript
   // 間違い
   headers: { 'Authorization': 'your_api_key' }
   
   // 正しい
   headers: { 'Authorization': 'Bearer your_api_key' }
   ```

3. **Content-Typeの欠如**:
   ```javascript
   headers: {
     'Authorization': 'Bearer your_api_key',
     'Content-Type': 'application/json'  // これを忘れないで！
   }
   ```

### レート制限超過

**問題**: 429 Too Many Requestsエラーが発生する。

**解決策**:
1. 指数バックオフを実装:
   ```javascript
   async function retryWithBackoff(fn, maxRetries = 3) {
     for (let i = 0; i < maxRetries; i++) {
       try {
         return await fn();
       } catch (error) {
         if (error.status === 429 && i < maxRetries - 1) {
           const delay = Math.pow(2, i) * 1000; // 1秒、2秒、4秒
           await new Promise(resolve => setTimeout(resolve, delay));
           continue;
         }
         throw error;
       }
     }
   }
   ```

2. レート制限ヘッダーを確認:
   ```javascript
   console.log('残りレート制限:', response.headers['x-ratelimit-remaining']);
   console.log('レート制限リセット:', response.headers['x-ratelimit-reset']);
   ```

## パフォーマンス問題

### APIレスポンスの遅延

**問題**: APIリクエストの完了に時間がかかりすぎる。

**診断手順**:

1. **ネットワーク遅延を確認**:
   ```bash
   ping api.example.com
   curl -w "@curl-format.txt" -o /dev/null -s "https://api.example.com/endpoint"
   ```

2. **データベースクエリを監視**:
   ```bash
   # PostgreSQLで遅いクエリログを有効にする
   # postgresql.confに追加:
   log_min_duration_statement = 1000  # 1秒以上かかるクエリをログ出力
   ```

3. **アプリケーションをプロファイル**:
   ```javascript
   console.time('api-call');
   const result = await apiCall();
   console.timeEnd('api-call');
   ```

**解決策**:
- 頻繁にクエリされるフィールドにデータベースインデックスを追加
- 繰り返しリクエストにキャッシュを実装
- コネクションプールを使用
- データベースクエリを最適化

### メモリリーク

**問題**: アプリケーションのメモリ使用量が増加し続ける。

**診断ツール**:
```bash
# メモリ使用量を監視
htop
# または
top -p $(pgrep node)

# Node.jsメモリプロファイリング
node --inspect your-app.js
```

**よくある原因**:
- 閉じられていないデータベース接続
- 削除されていないイベントリスナー
- ガベージコレクションされない大きなオブジェクト
- 循環参照

## 環境問題

### 環境変数が読み込まれない

**問題**: アプリケーションで環境変数が未定義になる。

**解決策**:

1. **.envファイルの場所を確認**（プロジェクトルートにある必要があります）

2. **.envファイルの形式を確認**:
   ```bash
   # 正しい
   API_KEY=your_key_here
   DATABASE_URL=postgresql://user:pass@host:port/db
   
   # 間違い（=の周りにスペースなし）
   API_KEY = your_key_here
   DATABASE_URL = postgresql://user:pass@host:port/db
   ```

3. **dotenvが早期に読み込まれていることを確認**:
   ```javascript
   // メインファイルの最初に
   require('dotenv').config();
   ```

4. **変数名のタイポを確認**:
   ```javascript
   // これらが完全に一致することを確認
   const apiKey = process.env.API_KEY; // process.env.API_keyではない
   ```

### ポートが既に使用中

**問題**: ポートが既に使用されているためサーバーを開始できない。

**解決策**:

1. **ポートを使用しているものを特定**:
   ```bash
   # Linux/macOS
   lsof -i :3000
   
   # Windows
   netstat -ano | findstr :3000
   ```

2. **プロセスを強制終了**:
   ```bash
   # Linux/macOS
   kill -9 $(lsof -t -i:3000)
   
   # Windows
   taskkill /PID <PID> /F
   ```

3. **異なるポートを使用**:
   ```bash
   PORT=3001 npm start
   ```

## ヘルプの取得

まだ問題が解決しない場合:

1. **FAQ確認**: [FAQページ]({{ site.baseurl }}/docs/faq/)
2. **既存の課題を検索**: [GitHub Issues](https://github.com/himaratsu/pagescms-sample/issues)
3. **新しい課題を作成**、以下を含める:
   - 問題の詳細な説明
   - 再現手順
   - 環境情報（OS、Node.jsバージョンなど）
   - エラーメッセージとログ

### デバッグ情報の収集

課題を報告する前に、この情報を収集してください:

```bash
# システム情報
node --version
npm --version
cat /etc/os-release  # Linux
sw_vers             # macOS

# アプリケーションログ
tail -f logs/app.log

# 環境変数（機密データを削除）
env | grep -E "(NODE_|API_|DATABASE_)" | sed 's/=.*/=***/'
```