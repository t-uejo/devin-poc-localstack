# AGENTS.md

`localstack-terraform-test`サブプロジェクト用の AGENTS.md のドキュメント。

共通コマンド（Prerequisites、Setup、基本的な Terraform/LocalStack コマンド）などはリポジトリルートの[AGENTS.md](../AGENTS.md)を参照。

## プロジェクト構造

```
localstack-terraform-test/
├── .gitignore
├── AGENTS.md                    # サブプロジェクト固有のルール・運用規約
├── docker-compose.yml           # LocalStackコンテナ定義
├── docs/                        # 設計書・ドキュメント格納
└── terraform/                   # Terraform設定ファイル
    ├── .terraform.lock.hcl      # Terraform依存関係ロックファイル
    └── provider.tf              # AWSプロバイダー設定
```

## テスト仕様

### 期待されるテスト結果

```shell
tests/localstack.tftest.hcl... in progress
  run "check_s3_bucket_name"... pass
  run "check_lambda_function"... pass
  run "check_name_of_filename_written_to_dynamodb"... pass
tests/localstack.tftest.hcl... tearing down
tests/localstack.tftest.hcl... pass

Success! 3 passed, 0 failed.
```

## デバッグ手順

### LocalStack 認証設定

AWS クラウドをエミュレートするローカル実行コンテナ内で AWS CLI コマンドを実行できるように、以下の環境変数を設定する。

ここでは direnv を使用し、プロジェクトディレクトリに入ると自動的に環境変数が設定されるようにする。

1. `.envrc` ファイルを作成

   ```shell
   cat > .envrc << 'EOF'
   export AWS_ACCESS_KEY_ID=test
   export AWS_SECRET_ACCESS_KEY=test
   export AWS_SESSION_TOKEN=test
   export AWS_REGION=ap-northeast-1
   export AWS_ENDPOINT_URL=http://localhost:4566
   EOF
   ```

2. direnv を許可

   ```shell
   direnv allow
   ```

これ以降、このディレクトリに入ると自動的に LocalStack 用の環境変数が設定される。ディレクトリから出ると環境変数は自動的にアンロードされる。

### 検証用コマンド

#### S3 へファイルをアップロード

README.md ファイルを test-file.txt として 3://my-test-bucket へアップロードします。

```shell
aws --endpoint-url=http://localhost:4566 s3 cp README.md s3://my-test-bucket/test-file.txt
```

#### ステートマシンの確認

```shell
aws --endpoint-url=http://localhost:4566 stepfunctions list-state-machines
```

#### DynamoDB への書き込み確認

```shell
aws --endpoint-url=http://localhost:4566 dynamodb scan --table-name Files
```
