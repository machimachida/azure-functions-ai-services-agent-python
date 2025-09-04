# macOS ローカル開発環境セットアップガイド

このドキュメントでは、Azure Functions AI Services Agent PythonプロジェクトをmacOSでローカル開発するための環境セットアップ方法を、プログラミング初心者の方にも分かりやすく説明します。

## 📋 必要なソフトウェア

### 1. Homebrew（パッケージマネージャー）

macOSでソフトウェアを簡単にインストールするために、まずHomebrewをインストールします。

参考: https://brew.sh/ja/

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

インストール後、ターミナルを再起動して確認：
```bash
brew --version
```

### 2. Python（必須）

このプロジェクトはPythonで開発されています。Python 3.11以降が必要です。

#### pyenvのインストール

Pythonのバージョン管理には`pyenv`を使用することを推奨します。

```bash
# Homebrewを使用してpyenvをインストール
brew install pyenv
```

#### pyenvの設定

`.zshrc`（またはお使いのシェルの設定ファイル）に以下を追加：

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

シェルを再起動またはsourceコマンドを実行：
```bash
source ~/.zshrc
```

#### Pythonのインストール

```bash
# 利用可能なPythonバージョンを確認
pyenv install --list | grep 3.11

# Python 3.11.xの最新版をインストール
pyenv install 3.11

# プロジェクトでPython 3.11を使用するように設定
pyenv local 3.11
```

#### Pythonインストール確認

```bash
python --version
```

### 3. Azure Functions Core Tools（必須）

Azure Functionsをローカルで実行するために必要です。

```bash
# Homebrewを使用（推奨）
brew tap azure/functions
brew install azure-functions-core-tools@4
```

#### インストール確認
```bash
func --version
```

### 4. Node.js（Azure Functions Core Tools用）

Azure Functions Core Toolsを動作させるために必要です。

#### すでにインストールされているのか確認

node.jsがバージョン18以上ならOKです。

```bash
node --version
npm --version
```

#### インストール

```bash
# Homebrewを使用
brew install node
```

#### インストール確認

```bash
node --version
npm --version
```

### 5. Azurite（ローカルストレージエミュレーター）

ローカル開発環境でAzure Storageをエミュレートします。

```bash
npm install -g azurite
```

#### インストール確認
```bash
azurite --version
```

### 7. Git（バージョン管理）

```bash
brew install git
```

#### インストール確認
```bash
git --version
```

## 🚀 プロジェクトセットアップ

### 1. リポジトリのクローン

```bash
git clone https://github.com/machimachida/azure-functions-ai-services-agent-python.git
cd azure-functions-ai-services-agent-python
```
### 2. Python仮想環境のセットアップ

仮想環境を作成することで、プロジェクトごとに独立したPython環境を作ることができます。
開発中は常にこの仮想環境を有効にしてください。

```bash
# appディレクトリに移動
cd app

# 仮想環境の作成（pyenvでインストールしたPythonを使用）
python -m venv .venv

# 仮想環境の有効化
source .venv/bin/activate
```

### 3. 依存関係のインストール

```bash
# Pythonパッケージのインストール
pip install -r requirements.txt
```

### 4. ローカル設定ファイルの確認

町田から以下のようなファイルをもらって`app/local.settings.json`に置いてください。

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "STORAGE_CONNECTION__queueServiceUri": "https://<storageaccount>.queue.core.windows.net",
    "PROJECT_ENDPOINT": "<AI Projectのエンドポイント>",
    "AzureWebJobsStorage": "UseDevelopmentStorage=true"
  }
}
```

## 🔧 開発環境の起動

### 1. Azuriteの起動

新しいターミナルウィンドウを開いて：

```bash
azurite --silent --location ~/azurite --debug ~/azurite/debug.log
```

### 2. Azure Functionsの起動

appディレクトリから：

```bash
cd app
func start
```

成功すると、以下のようなメッセージが表示されます：
```
Functions:
        GetWeather: [queueTrigger]
        prompt: [httpTrigger] http://localhost:7071/api/prompt
```

## 🧪 動作確認

### HTTPリクエストのテスト

#### VS Code REST Client拡張機能を使用する場合：
1. VS Codeに[REST Client拡張機能](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)をインストール
2. `app/test.http`ファイルを開く
3. リクエストの上にある「Send Request」をクリック

#### curlコマンドを使用する場合：
```bash
curl -X POST http://localhost:7071/api/prompt \
  -H "Content-Type: application/json" \
  -d '{"Prompt": "What is the weather in Tokyo?"}'
```

## 🛠️ 開発のワークフロー

### 1. コードの編集

`app/function_app.py`を書き換えればテストできます。
このコードサンプルは、お天気ツールを呼び出すエージェントです。
お天気ツールは、エージェントが指定した場所を受け取り、ダミーのお天気情報をエージェントに返します。

- 関数`initialize_client()`の中にあるAzureFunctionToolは、エージェントに渡すToolUseの説明文です。これを工夫することで、ToolUseの選択精度が高まります。
  - nameとdescriptionとparametersをいじってください。
  - input_queueとoutput_queueはいじらなくて良いです。
- 関数`initialize_client()`の中にある`project_client.agents.create_agent`の中にある`instructions`はエージェントに入力するシステムプロンプトです。書き換えることで、エージェントの動きが変わります。
- 関数`process_queue_message`は、エージェントに渡すツールの一例です。この例では、場所をlocationとして受け取り、ダミーのお天気情報をValueに入れて返します。

ライブラリをインストールする際は`app/requirements.txt`にライブラリの名前を追加します。
その後、以下のコマンドを入力します。

```bash
cd app  # 今appディレクトリにいない場合
pip install -r requirements.txt
```

### 2. テスト

test.httpを書き換えてリクエストを送ることでテストできます。
`Prompt`の内容を変更して様々なシナリオを試してください。
