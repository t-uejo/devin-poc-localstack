# AGENTS.md

`AGENTS.md` は、プロジェクトに関わる AI エージェント（例：Devin や Cursor など）の**設定手順や開発ルールをまとめたドキュメント**です。

開発環境のセットアップ方法、利用するコマンド、コード規約、推奨ワークフローなどを記載し、AI エージェントとチーム全体で統一した運用を行うために用います。

## プロジェクト構造

```
devin-poc-localstack/
├── .gitignore
├── .terraform.lock.hcl          //Terraformの依存関係ロックファイル
├── AGENTS.md
├── README.md
├── docker-compose.yml           //LocalStackコンテナの設定
├── docs/                        //ドキュメントディレクトリ
└── provider.tf                  //AWSプロバイダーの設定
```

## ファイル命名規則

### Terraform ファイル

Terraform プロジェクトでは、機能別にファイルを分割することで保守性と可読性を向上させます。以下の命名規則に従ってファイルを整理してください。

#### 基本ファイル構成

| ファイル名     | 用途               | 説明                                                 |
| -------------- | ------------------ | ---------------------------------------------------- |
| `main.tf`      | メインリソース定義 | 主要な AWS リソース（S3、Lambda、DynamoDB 等）の定義 |
| `variables.tf` | 変数定義           | 入力変数の宣言とデフォルト値の設定                   |
| `outputs.tf`   | 出力値定義         | 他のモジュールやルートモジュールで参照する値の定義   |
| `provider.tf`  | プロバイダー設定   | AWS プロバイダーの設定と認証情報                     |
| `versions.tf`  | バージョン制約     | Terraform とプロバイダーのバージョン制約             |

## Terraform Test

### 1. LocalStack コンテナの実行

クローンしたリポジトリで、bash シェルで以下のコマンドを入力して、LocalStack Docker をデタッチモードで実行開始します。

```shell
docker compose up -d
```

LocalStack コンテナが起動して実行されるまで待機します。

### 2. Terraform の初期化

クローンしたリポジトリから以下のコマンドを入力して Terraform を初期化します。

```shell
terraform init
```

### 3. Terraform テストの実行

Terraform Test を実行するために以下のコマンドを入力します。

```shell
terraform test
```

すべてのテストが正常に通過したことを確認します。

出力は以下のようになります：

```shell
tests/localstack.tftest.hcl... in progress
  run "check_s3_bucket_name"... pass
  run "check_lambda_function"... pass
  run "check_name_of_filename_written_to_dynamodb"... pass
tests/localstack.tftest.hcl... tearing down
tests/localstack.tftest.hcl... pass

Success! 3 passed, 0 failed.
```

### 4. リソースのクリーンアップ

LocalStack コンテナを破棄するために以下のコマンドを入力します。

```shell
docker compose down
```

## AWS CLI でのデバッグ

### 1. LocalStack コンテナの実行

クローンしたリポジトリで、bash シェルで以下のコマンドを入力して、LocalStack Docker をデタッチモードで実行開始します。

```shell
docker compose up -d
```

LocalStack コンテナが起動して実行されるまで待機します。

### 2. 認証

AWS クラウドをエミュレートするローカル実行コンテナ内で AWS CLI コマンドを実行できるように、以下の環境変数をエクスポートします。

```shell
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_SESSION_TOKEN=test
export AWS_REGION=ap-northeast-1
```

### 3. ローカルでのリソース作成

ローカルで実行中のコンテナ内でリソースを作成します。

```shell
terraform init
terraform plan
terraform apply -auto-approve
```

### 4. デプロイされたリソースでの AWS CLI コマンド実行

以下のコマンドでデバッグを行い、設計書の仕様を満たせているか確認してください。

#### 4-1. S3 へファイルをアップロード

`README.md`ファイルを`test-file.txt`として`3://my-test-bucket`へアップロード。

```
aws --endpoint-url=http://localhost:4566 s3 cp README.md s3://my-test-bucket/test-file.txt
```

#### 4-2. ステートマシンが作成されたことを確認

```shell
aws --endpoint-url http://localhost:4566 stepfunctions list-state-machines
```

#### 4-3. dynamodb にファイルが書き込まれた確認

```
aws --endpoint-url=http://localhost:4566 dynamodb scan --table-name Files
```

### 5. リソースの破棄

```shell
terraform destroy -auto-approve
```

LocalStack コンテナを破棄するために以下のコマンドを入力します。

```shell
docker compose down
```
