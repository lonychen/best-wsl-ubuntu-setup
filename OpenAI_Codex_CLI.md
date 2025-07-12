# OpenAI Codex CLI 環境建置說明

## 初始化容器內專用工具

1. 安裝 Codex CLI 必備工具

   ```sh
   sudo apt-get update && sudo apt-get install -y --no-install-recommends \
     aggregate \
     ca-certificates \
     curl \
     dnsutils \
     fzf \
     gh \
     git \
     gnupg2 \
     iproute2 \
     ipset \
     iptables \
     yq \
     jq \
     less \
     man-db \
     procps \
     unzip \
     ripgrep \
     zsh \
     zstd
   ```

2. 安裝 Codex CLI

   ```sh
   npm i -g @openai/codex

   codex --version
   ```

   💡 目前最新版為 `0.5.0`

3. 取得你的 OpenAI API Key

   ```sh
   codex login
   ```

   他會要求你登入 ChatGPT 或 OpenAI 帳號，只要登入成功就可以得到一把 API Key。

4. 測試 `codex` 是否可以正常運作

   ```sh
   codex 'hi'
   ```

## 從原始碼建置 codex 工具

```sh
# 設定 Git 配置，避免 Windows CRLF 換行符號問題
# 在跨平台開發時，Windows 使用 CRLF(\r\n)，而 Linux/Mac 使用 LF(\n)
# 設定為 input 可確保檔案在儲存時統一使用 LF 換行符號
echo "📝 設定 Git 換行符號處理方式..."
git config --global core.autocrlf input

# 切換到使用者主目錄並建立專案資料夾
# ~ 代表使用者的主目錄 (home directory)
echo "📁 建立專案目錄結構..."
cd ~/
mkdir projects; cd projects

# 從 GitHub 克隆 OpenAI Codex 儲存庫
# 這會下載完整的專案原始碼到本地端
echo "⬇️ 下載 OpenAI Codex 原始碼..."
git clone https://github.com/openai/codex.git
cd codex
git checkout 7896b1089dbf702dd07929910504e9558a20d085
cd codex-cli

# 獲取並套用社群貢獻的 Pull Request 修正
# 這些是 Will 保哥提交的重要修正
echo "🔧 套用社群修正補丁..."
git fetch origin '+refs/pull/*/head:refs/remotes/origin/pr/*'
echo "   - 套用 PR #1004 修正..."
git cherry-pick origin/pr/1004
echo "   - 套用 PR #1121 修正..."
git cherry-pick origin/pr/1121
echo "   - 套用 PR #1122 修正..."
git cherry-pick origin/pr/1122
echo "   - 套用 PR #1125 修正..."
git cherry-pick origin/pr/1125
echo "   - 套用 PR #1130 修正..."
git cherry-pick origin/pr/1130
echo "   - 套用 PR #1134 修正..."
git cherry-pick origin/pr/1134
echo "   - 套用 PR #1143 修正..."
git cherry-pick origin/pr/1143

# 啟用 Node.js Corepack 套件管理器
# Corepack 允許使用不同的套件管理器 (如 pnpm, yarn) 而不需要全域安裝
echo "⚙️ 啟用 Node.js Corepack..."
corepack enable

# 設定 pnpm 套件管理器
echo "📦 設定 pnpm 套件管理器..."
yes | pnpm setup

# 套用 pnpm 套件管理器的環境變數
echo "📦 套用 pnpm 套件管理器的環境變數..."
source ~/.bashrc

# 安裝專案相依套件並進行建置
echo "📦 安裝專案相依套件..."
pnpm install

echo "🔨 編譯專案..."
pnpm build

# 僅限 Linux 系統：下載預建的沙盒執行檔案
# 這些檔案用於安全地執行程式碼，需要 gh (GitHub CLI) 和 zstd 工具
echo "🛡️ 安裝沙盒執行環境 (僅限 Linux) ..."
./scripts/install_native_deps.sh

# 顯示 CLI 工具的使用說明和選項
echo "📖 顯示使用說明..."
node ./dist/cli.js --help

# 直接執行本地建置的 CLI 工具
echo "🚀 測試執行 Codex CLI..."
node ./dist/cli.js

# 將 CLI 工具連結到全域環境，方便在任何地方使用
echo "🌐 安裝到全域環境..."
pnpm link --global

# 驗證安裝版本 (0.0.0-dev)
echo "✅ 驗證安裝版本... 注意: 自行建置的版本為 0.0.0-dev"
codex --version

# 設定 Bash 命令自動完成功能
# 這讓您可以使用 Tab 鍵自動完成 codex 命令
echo "⌨️ 設定命令自動完成..."
codex completion bash | sudo tee /etc/bash_completion.d/codex
source ~/.bashrc

echo "🎉 本地 OpenAI Codex CLI 安裝完成！"
```

## 如何設定 Azure OpenAI 金鑰給 codex 工具使用

1. 設定環境變數到 `~/.profile`

    ```sh
    # Azure OpenAI Service
    export AZURE_DEPLOYMENT_NAME='codex-mini'
    export AZURE_RESOURCE_NAME='your-resource-name'
    export AZURE_BASE_URL="https://${AZURE_RESOURCE_NAME}.openai.azure.com/openai"
    export AZURE_OPENAI_API_VERSION='2025-04-01-preview'
    export AZURE_OPENAI_API_KEY='your-api-key-from-azure'
    export OPENAI_API_KEY=$AZURE_OPENAI_API_KEY

    # Telegram Bot Token
    # https://core.telegram.org/bots#botfather
    export TG_BOT_TOKEN=''

    export CODEX_UNSAFE_ALLOW_NO_SANDBOX=1
    export CODEX_SANDBOX_NETWORK_DISABLED=0
    ```

2. 設定 Codex CLI 設定檔 `~/.codex/config.json`

    ```json
    {
        "model": "codex-mini",
        "provider": "azure"
    }
    ```

3. 設定完就可以預設用 Azure OpenAI Services 的端點執行！

## 相關連結

- [Developing inside a Container](https://code.visualstudio.com/docs/devcontainers/containers)
- [Development Containers](https://containers.dev/)
- <https://github.com/openai/codex>
