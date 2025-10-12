# AGENTS.md

`AGENTS.md` は、プロジェクトに関わる AI エージェント（例：Devin や Cursor など）の**設定手順や開発ルールをまとめたドキュメント**です。

開発環境のセットアップ方法、利用するコマンド、コード規約、推奨ワークフローなどを記載し、AI エージェントとチーム全体で統一した運用を行うために用います。

## Project Structure

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

### Terraform File Naming Conventions

- main.tf - メインのリソース定義
- vars.tf - 変数定義
- outputs.tf - 出力値定義
- provider.tf - プロバイダー設定
- versions.tf - バージョン制約

## Terraform Test

### 1. Run Local Stack Container

クローンしたリポジトリで、bash シェルで以下のコマンドを入力して、LocalStack Docker をデタッチモードで実行開始します。

```shell
docker compose up -d
```

LocalStack コンテナが起動して実行されるまで待機します。

### 2. Terraform Initialization

クローンしたリポジトリから以下のコマンドを入力して Terraform を初期化します。

```shell
terraform init
```

### 3. Run Terraform Test

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

### 4. Resource Cleanup

LocalStack コンテナを破棄するために以下のコマンドを入力します。

```shell
docker compose down
```

## Debugging with AWS CLI

### 1. Run Local Stack Container

クローンしたリポジトリで、bash シェルで以下のコマンドを入力して、LocalStack Docker をデタッチモードで実行開始します。

```shell
docker-compose up -d
```

LocalStack コンテナが起動して実行されるまで待機します。

### 2. Authentication

AWS クラウドをエミュレートするローカル実行コンテナ内で AWS CLI コマンドを実行できるように、以下の環境変数をエクスポートします。

```shell
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_SESSION_TOKEN=test
export AWS_REGION=ap-northeast-1
```

### 3. Create Resources Locally

ローカルで実行中のコンテナ内でリソースを作成します。

```shell
terraform init
terraform plan
terraform apply -auto-approve
```

最後に、デプロイされたリソースに対して AWS CLI コマンドを実行できます。例えば、ステートマシンが作成されたことを確認することができます。

```shell
aws --endpoint-url http://localhost:4566 stepfunctions list-state-machines
```

### 4. Destroy the resources

```shell
terraform destroy -auto-approve
```

LocalStack コンテナを破棄するために以下のコマンドを入力します。

```shell
docker compose down
```
