# GitHub Pages デプロイ手順

このドキュメントでは、EvidenceアプリをGitHub Pagesにデプロイする手順を説明します。

## 📋 前提条件

- GitHubアカウント
- リポジトリが既にGitHubにプッシュされている
- MotherDuckのアクセストークン

## 🚀 デプロイ手順

### 1. GitHub Pagesの設定

1. GitHubリポジトリページへアクセス
2. **Settings** > **Pages** を開く
3. **Source** を `GitHub Actions` に変更

### 2. Secretsの設定

1. **Settings** > **Secrets and variables** > **Actions** を開く
2. **New repository secret** をクリック
3. 以下のSecretを追加:

```
Name: MOTHERDUCK_TOKEN
Value: [あなたのMotherDuckトークン]
```

#### 環境変数の命名規則

```
EVIDENCE_SOURCE__[ソース名]__[オプション名]
```

**例:**
- MotherDuckの場合: `EVIDENCE_SOURCE__motherduck__token`
- PostgreSQLの場合:
  - `EVIDENCE_SOURCE__postgres__host`
  - `EVIDENCE_SOURCE__postgres__database`
  - `EVIDENCE_SOURCE__postgres__user`
  - `EVIDENCE_SOURCE__postgres__password`

### 3. デプロイ実行

以下のいずれかの方法でデプロイを実行:

#### 自動デプロイ (推奨)
`main`ブランチにプッシュすると自動的にデプロイされます:

```bash
git add .
git commit -m "Deploy to GitHub Pages"
git push origin main
```

#### 手動デプロイ
1. **Actions** タブを開く
2. **Deploy Evidence to GitHub Pages** ワークフローを選択
3. **Run workflow** をクリック

### 4. デプロイ確認

デプロイが完了したら、以下のURLでアクセスできます:

```
https://[username].github.io/motherduck-dbt-evidence/
```

## 📁 設定ファイル

以下のファイルが既に設定されています:

### `reports/evidence.config.yaml`
```yaml
deployment:
  basePath: /motherduck-dbt-evidence
```

### `reports/package.json`
```json
"scripts": {
  "build": "EVIDENCE_BUILD_DIR=./build/motherduck-dbt-evidence evidence build"
}
```

### `.github/workflows/deploy.yml`
GitHub Actionsのワークフロー定義ファイル

## 🔄 定期更新の設定 (オプション)

データを定期的に更新したい場合、`.github/workflows/deploy.yml`の以下の行のコメントを外してください:

```yaml
schedule:
  - cron: '0 */6 * * *'  # 6時間ごと
```

### cronスケジュール例

```yaml
# 毎時実行
- cron: '0 * * * *'

# 6時間ごと
- cron: '0 */6 * * *'

# 毎日午前9時(UTC)
- cron: '0 9 * * *'

# 平日の午前9時(UTC)
- cron: '0 9 * * 1-5'
```

## 🐛 トラブルシューティング

### ビルドが失敗する場合

1. **Secretsの確認**: 環境変数が正しく設定されているか確認
2. **ログの確認**: Actions タブでビルドログを確認
3. **ローカルビルドテスト**:
   ```bash
   cd reports
   npm run build
   ```

### ページが表示されない場合

1. **Pages設定の確認**: Sourceが`GitHub Actions`になっているか確認
2. **URL確認**: `https://[username].github.io/motherduck-dbt-evidence/` にアクセス
3. **キャッシュクリア**: ブラウザのキャッシュをクリア

### データソース接続エラー

1. **トークンの有効期限**: MotherDuckトークンの有効期限を確認
2. **権限の確認**: トークンに適切な権限があるか確認
3. **ネットワーク**: GitHub Actionsからアクセス可能か確認

## 📚 参考資料

- [Evidence公式ドキュメント - GitHub Pages](https://docs.evidence.dev/deployment/self-host/github-pages/)
- [GitHub Actions ドキュメント](https://docs.github.com/en/actions)
- [GitHub Pages ドキュメント](https://docs.github.com/en/pages)

## 🎯 次のステップ

- カスタムドメインの設定
- CI/CDパイプラインの最適化
- データ更新スケジュールの調整
- パフォーマンスモニタリング
