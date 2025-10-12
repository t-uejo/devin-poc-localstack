# AGENTS.md (Repository Root)

## Purpose

リポジトリ全体の運用規約・共通ルールを定義する。Devin 等のエージェントは、本ドキュメントを最優先の共通原則として参照すること。

サブプロジェクト固有の詳細は、各フォルダ直下の `AGENTS.md` を優先すること。競合時はサブプロジェクトのルールを優先すること。

## Repository Layout

以下に示すように本リポジトリはモノレポ構成である。

```
devin-poc-localstack/
├── README.md                    # リポジトリ全体説明
├── AGENTS.md                    # AIエージェント用リポジトリ全体運用規約・共通ルール
└── localstack-terraform-test/   # サブプロジェクトのルート
    ├── README.md                # プロジェクト説明
    ├── AGENTS.md                # サブプロジェクト固有のルール・運用規約
    ├── docker-compose.yml       # LocalStack コンテナ定義
    ├── docs/                    # 設計書ファイルを格納
    └── terraform/               # Terraform 設定ファイル
        └── provider.tf          # プロバイダー設定
```

## Prerequisites

- Terraform v1.6+（tfenv 推奨、`terraform test` コマンド対応）
- `terraform-docs` 最新安定版
- `tflint` 最新安定版（Terraform 静的解析）
- `checkov` 最新安定版（セキュリティ・コンプライアンススキャン）
- Python 3.11+
- Docker
- LocalStack
- （必要に応じて）tflocal, awscli-local

## Setup

以下は Devin 仮想マシンを想定した手順である。Devin 仮想マシンには `Docker` や `pip` は既にインストール済みである。

### Linux (Ubuntu/Debian)

```bash
# Terraform
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
tfenv install
tfenv use

# terraform-docs
curl -sSLo terraform-docs.tar.gz https://github.com/terraform-docs/terraform-docs/releases/download/v0.19.0/terraform-docs-v0.19.0-linux-amd64.tar.gz

tar -xzf terraform-docs.tar.gz && sudo mv terraform-docs /usr/local/bin/ && rm terraform-docs.tar.gz

# tflint
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# checkov
pip install checkov
```

## Commands (common)

### LocalStack

```bash
# LocalStack コンテナの実行
docker compose up -d

# リソースのクリーンアップ
docker compose down
```

### Terraform

```bash
# 初期化
terraform init

# 検証
terraform fmt -check && terraform validate

# リソースのプラン
terraform plan

# リソース作成
terraform apply -auto-approve

# テスト
terraform test
```

### 静的解析

```bash
# tflint による静的解析
tflint --init && tflint

# checkov によるセキュリティスキャン
checkov -d . --quiet
```

### terraform-docs

```bash
# ドキュメント生成
terraform-docs markdown . > README.md
```

## Quality Gates

- `terraform fmt` が差分ゼロであること
- `tflint`/`checkov`/`terraform validate` がエラー無し

## Conventions

### ファイル命名規則

| ファイル名            | 用途                                               |
| --------------------- | -------------------------------------------------- |
| `main.tf`             | メインリソース定義                                 |
| `variables.tf`        | 入力変数の宣言とデフォルト値                       |
| `outputs.tf`          | 出力値定義                                         |
| `provider.tf`         | プロバイダー設定                                   |
| `versions.tf`         | Terraform/プロバイダーのバージョン制約             |
| `.terraform.lock.hcl` | 依存バージョンのロックファイル（**Git 管理必須**） |

### 命名規則

<!-- TODO:モジュールなど命名規則を明確にする -->

- リソース/モジュール: 接頭辞を統一（例: `proj-<sys>-<comp>`）

## Security

- 認証情報はコミット禁止（`secrets/` は gitignore）
- IRSA / OIDC 等のベストプラクティスを優先

## For Agents (Devin 等)

- 変更提案は PR として明確な差分を提示
- 大量コード生成よりも最小差分・明確な根拠を重視
- 各サブプロジェクトのコンテキスト（目的・制約）を理解してから実装
- ドキュメント（README.md / docs）は変更と同時に更新
- Quality Gates をすべてクリアしてからコミット提案
