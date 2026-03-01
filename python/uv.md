# uv - 高速なPythonパッケージ・プロジェクトマネージャー

## 概要

**uv**は、Rust製の高速なPythonパッケージおよびプロジェクトマネージャーです。pip、pip-tools、poetry、pipenvなどの従来のPythonツールチェーンの代替として設計されています。

## インストール

### Linux / macOS

curlを使用する場合:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

wgetを使用する場合:

```bash
wget -qO- https://astral.sh/uv/install.sh | sh
```

### Windows

PowerShellで実行:

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### パッケージマネージャー経由

Homebrew（macOS/Linux）:

```bash
brew install uv
```

Cargo（Rustツールチェーン）:

```bash
cargo install uv
```

### インストール確認

インストールが成功したか確認します:

```bash
uv --version
```

正しくインストールされていれば、バージョン情報が表示されます（例: `uv 0.1.0`）。

## プロジェクトの初期化

### 新規プロジェクトの作成

新しいPythonプロジェクトを作成するには、`uv init`コマンドを使用します:

```bash
uv init my-project
cd my-project
```

または、カレントディレクトリでプロジェクトを初期化:

```bash
mkdir my-project
cd my-project
uv init
```

### 生成されるファイル構造

`uv init`を実行すると、以下のファイルとディレクトリが生成されます:

```txt
my-project/
├── .python-version    # 使用するPythonバージョンの指定
├── pyproject.toml     # プロジェクト設定と依存関係
├── README.md          # プロジェクトの説明
└── src/               # ソースコードディレクトリ
    └── my_project/
        └── __init__.py
```

### pyproject.tomlの基本構造

生成される`pyproject.toml`は、プロジェクトのメタデータと依存関係を管理します:

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.8"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

## 依存関係管理

### パッケージの追加

プロジェクトに新しい依存関係を追加するには、`uv add`コマンドを使用します:

```bash
# 単一パッケージを追加
uv add requests

# バージョンを指定して追加
uv add "numpy>=1.20.0"

# 複数パッケージを同時に追加
uv add requests pandas numpy
```

### 開発用依存関係の追加

テストやリンターなど、開発時のみ必要なパッケージは`--dev`フラグを使用します:

```bash
# 開発用依存関係として追加
uv add --dev pytest
uv add --dev ruff
```

これらは`pyproject.toml`の`[project.optional-dependencies]`セクションに記録されます。

### パッケージの削除

不要になった依存関係を削除するには、`uv remove`コマンドを使用します:

```bash
# パッケージを削除
uv remove requests

# 開発用依存関係を削除
uv remove --dev pytest
```

### 依存関係のインストール

既存のプロジェクトで依存関係をインストールする場合:

```bash
# すべての依存関係をインストール
uv sync

# 開発用依存関係を含めてインストール
uv sync --all-groups
```

### ロックファイル（uv.lock）

uvは`uv.lock`ファイルを使用して、依存関係の正確なバージョンを記録します:

- **目的**: 依存関係の再現性を保証（異なる環境で同じバージョンをインストール）
- **役割**: 直接的な依存関係だけでなく、推移的依存関係（依存先の依存）もロック
- **Git管理**: `uv.lock`はリポジトリにコミットすることを推奨
- **更新**: `uv add`や`uv remove`を実行すると自動的に更新される

### 依存関係の更新

パッケージを最新バージョンに更新するには:

```bash
# すべての依存関係を更新
uv lock --upgrade

# 特定のパッケージのみ更新
uv lock --upgrade-package requests

# ロックファイルに基づいてインストール
uv sync
```

## 仮想環境管理

### uvの自動仮想環境

uvは**自動的に仮想環境を作成・管理**します。従来の`python -m venv`や`virtualenv`を明示的に実行する必要はありません。

### 仮想環境の場所

uvが作成する仮想環境は、プロジェクトディレクトリ直下の`.venv/`に配置されます:

```txt
my-project/
├── .venv/               # 自動作成される仮想環境
│   ├── bin/
│   ├── lib/
│   └── ...
├── pyproject.toml
└── src/
```

### コマンドの実行

仮想環境内でPythonコマンドやスクリプトを実行するには、`uv run`を使用します:

```bash
# Pythonスクリプトを実行
uv run python script.py

# モジュールを実行
uv run python -m pytest

# インストールされたコマンドラインツールを実行
uv run ruff check .

## ruffで自動修正
uv run ruff check --fix .

## フォーマットを実行
uv run ruff format .
```

`uv run`を使用すると、自動的に適切な仮想環境が有効化され、コマンドが実行されます。

> [!TIP]
> ruffを使用する場合は、以下のように`pyproject.toml`ファイルに追記します。

```toml
[tool.ruff]
# 対象とするPythonバージョン
target-version = "py312"

# 1行の最大文字数
line-length = 88

# チェック対象から除外するディレクトリ
exclude = [
    ".venv",
    "__pycache__",
    "migrations",
]

[tool.ruff.lint]
# 有効にするルール
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # Pyflakes
    "I",    # isort
    "B",    # flake8-bugbear
]

# 無視するルール
ignore = []

[tool.ruff.format]
# クォートスタイル
quote-style = "double"

# インデントスタイル
indent-style = "space"
```

### 環境の有効化（オプション）

必要に応じて、従来のように仮想環境を明示的に有効化することも可能です:

**Linux / macOS:**

```bash
source .venv/bin/activate
```

**Windows:**

```powershell
.venv\Scripts\activate
```

有効化後は、通常のPythonコマンドがそのまま使用できます:

```bash
python script.py
pytest
```

無効化するには:

```bash
deactivate
```

## 基本的なワークフロー例

新しいプロジェクトを開始してから開発するまでの典型的な流れ:

```bash
# 1. 新規プロジェクトを作成
uv init my-web-app
cd my-web-app

# 2. 必要な依存関係を追加
uv add flask requests
uv add --dev pytest black

# 3. Pythonファイルを作成
cat > src/my_web_app/app.py << 'EOF'
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
EOF

# 4. アプリケーションを実行
uv run python src/my_web_app/app.py

# 5. テストを実行
uv run pytest
```

## VS Codeとの連携

vscodeを使用している場合は、`ruff`と`Python`拡張機能をインストールし、プロジェクトディレクトリ内の`.vscode/settings.json`ファイルに以下のような設定を追記してください。

```json
{
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll.ruff": "explicit",
            "source.organizeImports.ruff": "explicit"
        },
        "editor.tabSize": 4,
    },
    "python.testing.pytestEnabled": true,
    "python.testing.pytestArgs": ["tests"],
}
```

## よく使うコマンド一覧

### プロジェクト管理

| コマンド | 説明 | 使用例 |
|---------|------|--------|
| `uv init` | 新規プロジェクトを初期化 | `uv init my-project` |
| `uv init --lib` | ライブラリプロジェクトを初期化 | `uv init --lib my-lib` |

### 依存関係管理

| コマンド | 説明 | 使用例 |
|---------|------|--------|
| `uv add` | パッケージを追加 | `uv add requests` |
| `uv add --dev` | 開発用依存関係を追加 | `uv add --dev pytest` |
| `uv remove` | パッケージを削除 | `uv remove requests` |
| `uv sync` | 依存関係をインストール | `uv sync` |
| `uv lock` | ロックファイルを更新 | `uv lock --upgrade` |

### Python実行

| コマンド | 説明 | 使用例 |
|---------|------|--------|
| `uv run` | 仮想環境内でコマンド実行 | `uv run python script.py` |
| `uv run --with` | 一時的にパッケージを追加して実行 | `uv run --with httpx python -c "import httpx"` |

### その他

| コマンド | 説明 | 使用例 |
|---------|------|--------|
| `uv pip install` | pipコマンドの代替 | `uv pip install requests` |
| `uv pip list` | インストール済みパッケージ一覧 | `uv pip list` |
| `uv python install` | Python自体をインストール | `uv python install 3.12` |
| `uv python list` | 利用可能なPythonバージョン一覧 | `uv python list` |

## トラブルシューティング

### uvコマンドが見つからない

**症状**: `uv: command not found`と表示される

**解決策**:

1. シェル設定ファイルを再読み込み:

   ```bash
   source ~/.bashrc  # または ~/.zshrc
   ```

2. パスが通っているか確認:

   ```bash
   echo $PATH | grep .cargo/bin
   ```

3. uvを再インストール

### 権限エラー

**症状**: パッケージのインストール時に権限エラーが発生

**解決策**:

- `sudo`を**使用しない**でください。uvは仮想環境内にインストールするため不要です
- ホームディレクトリの権限を確認:

  ```bash
  ls -la ~/.local/share/uv
  ```

### 既存プロジェクトでuvを使用する

**シナリオ**: requirements.txtやpoetryを使用している既存プロジェクト

**解決策**:

1. requirements.txtから移行:

   ```bash
   # uv initで初期化
   uv init
   
   # requirements.txtから依存関係をインポート
   uv pip install -r requirements.txt
   
   # pyproject.tomlに依存関係を記録（手動で追加、またはuv addで再追加）
   ```

2. poetryから移行:

   ```bash
   # pyproject.tomlはそのまま使用可能
   uv sync
   ```

   uvはpoetryの`pyproject.toml`形式と互換性があります。

### Python バージョンの指定

**シナリオ**: 特定のPythonバージョンを使用したい

**解決策**:

1. `.python-version`ファイルを編集:

   ```bash
   echo "3.11" > .python-version
   ```

2. または、プロジェクト初期化時に指定:

   ```bash
   uv init --python 3.11
   ```

3. uvでPythonをインストール:

   ```bash
   uv python install 3.11
   ```

### 依存関係の競合

**症状**: 依存関係の解決に失敗する

**解決策**:

1. ロックファイルを削除して再生成:

   ```bash
   rm uv.lock
   uv lock
   ```

2. 特定のバージョンを明示的に指定:

   ```bash
   uv add "package==1.2.3"
   ```

3. バージョン範囲を緩和:

   ```bash
   uv add "package>=1.0,<2.0"
   ```

## 参考リンク

- [uv公式ドキュメント](https://docs.astral.sh/uv/)
- [uv GitHubリポジトリ](https://github.com/astral-sh/uv)
- [Python Packaging User Guide](https://packaging.python.org/)

---

**最終更新**: 2026年3月
