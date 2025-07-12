# 最佳 Container + Ubuntu 24.04.2 LTS 環境設定

這份文件主要是用來快速設定 [Ubuntu 24.04.2 LTS](https://hub.docker.com/_/ubuntu/tags) 容器環境，包含了常用的工具和配置。

## 啟動容器

```sh
docker run -it --name=codex ubuntu:24.04
```

## 隨時更新 Ubuntu 到最新版本

```sh
apt update
apt upgrade -y
```

> 💡 容器中預設就是以 `root` 身分登入，所以不需要執行 `sudo` 命令。

> 💡 建議不要執行 `apt dist-upgrade -y`

### 設定作業系統時區

```sh
# set timezone to +0800
ln -fs /usr/share/zoneinfo/Asia/Taipei /etc/localtime
```

### 安裝常用的工具

這些工具可以幫助你更方便地使用 Linux 環境，特別是對於開發者來說，這些工具都是必不可少的。

```sh
# Installing essential packages...
apt install -y tzdata net-tools ripgrep jq lftp moreutils btop bat zip lsb-release wget curl vim git

# yq: https://github.com/mikefarah/yq
wget -q https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/local/bin/yq && chmod +x /usr/local/bin/yq

cat <<'EOF' | tee -a ~/.profile > /dev/null
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
EOF

mkdir -p ~/.local/bin
source ~/.profile

ln -s /usr/bin/batcat ~/.local/bin/bat
```

### 設定 Node.js 環境

我會建議用 [nvm](https://github.com/nvm-sh/nvm) (Node Version Manager) 來管理 Node.js 的版本，這樣可以方便地切換不同的 Node.js 版本，真的可以減少很多麻煩事。

```sh
curl -s -o- https://raw.githubusercontent.com/nvm-sh/nvm/$(curl -s "https://api.github.com/repos/nvm-sh/nvm/releases/latest" | jq -r .tag_name)/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
nvm alias default 22
node -v

cat <<'EOF' | tee -a ~/.profile
export NODE_NO_WARNINGS=1
EOF
```

### 設定 Bash 環境

```sh
# profile setup
cat <<'EOF' | tee -a ~/.profile
export EDITOR=vim
export GPG_TTY=$(tty)
EOF

# bash setup
cat <<'EOF' | tee -a ~/.bashrc
# Enable programmable completion features
shopt -u direxpand
shopt -s no_empty_cmd_completion
EOF

# ssh setup
ssh-keygen -t rsa -b 4096 -f $HOME/.ssh/id_rsa -P ""
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
# Remember add your SSH key to GitHub: $(cat ~/.ssh/id_rsa.pub)"

# setup workspace
mkdir -p ~/projects
```

### 設定華麗的 Bash 提示符號

這裡我推薦用 [Starship](https://starship.rs/) 來當作為 Bash 的提示符號，這個工具非常輕量且功能強大，可以讓你的終端機看起來既美觀又實用。

```sh
# Install Starship
curl -sS https://starship.rs/install.sh | sh -s -- -y

# 參考設定 https://starship.rs/config/
mkdir -p ~/.config && touch ~/.config/starship.toml

# 安裝預設主題 https://starship.rs/presets/catppuccin-powerline
starship preset catppuccin-powerline -o ~/.config/starship.toml

cat <<'EOF' | tee -a ~/.bashrc > /dev/null
eval "$(starship init bash)"
EOF

source ~/.bashrc
```

### 安裝 [fzf](https://github.com/junegunn/fzf) 工具

這是一個非常強大的命令列模糊搜尋工具，可以讓你在終端機中快速找到檔案或目錄。

```sh
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install --all
source ~/.bashrc
```

💡 常用快速鍵：

1. 按下 `Ctrl-R` 可以快速搜尋歷史命令
2. 按下 `Ctrl-T` 可以快速搜尋檔案 (從當前資料夾開始找起)

### 設定 Git 環境

```sh
# git setup
npx -y @willh/git-setup --name 'Your Name' --email username@gmail.com
git config --global core.autocrlf input
git config --global init.defaultBranch main
```

> 💡 這個 [@willh/git-setup](https://www.npmjs.com/package/@willh/git-setup) 是我多年前開發的小工具，換新電腦的時候很實用，支援跨平臺自動設定 Git 常見參數。

如果您打算使用 Azure Repos 或 Azure DevOps，則需要進行一些額外的設定，以下命令可以讓你在 WSL 直接跟 Windows 共用同一個 [Git Credential Manager (GCM)](https://github.com/GitCredentialManager/git-credential-manager)：

```sh
git config --global credential.https://dev.azure.com.useHttpPath true
```

### 設定 Vim 編輯器

Vim 是一個非常強大的文字編輯器，雖然它的學習曲線有點陡峭，但一旦掌握了，就會發現它的效率非常高。

```sh
cat <<EOF | tee ~/.vimrc
syntax on
set background=dark

set encoding=utf-8
set fileencoding=utf-8
set fileencodings=utf-8,ucs-bom,cp936,gb18030,big5

let &t_SI .= "\<Esc>[?2004h"
let &t_EI .= "\<Esc>[?2004l"

inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()

function! XTermPasteBegin()
  set pastetoggle=<Esc>[201~
  set paste
  return ""
endfunction
EOF
```

### 安裝 [GitHub CLI](https://cli.github.com/)

> [Installing gh on Linux and BSD](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)

```sh
(type -p wget >/dev/null || (apt update && apt-get install wget -y)) \
    && mkdir -p -m 755 /etc/apt/keyrings \
        && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        && cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && mkdir -p -m 755 /etc/apt/sources.list.d \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && apt update \
    && apt install gh -y

gh help environment

gh auth login --web -h github.com
gh auth status

gh extension install github/gh-copilot
gh extension upgrade github/gh-copilot

gh copilot --help

echo 'eval "$(gh copilot alias -- bash)"' >> ~/.bashrc
source ~/.bashrc
```

```txt
github.com
  ✓ Logged in to github.com account doggy8088 (/root/.config/gh/hosts.yml)
  - Active account: true
  - Git operations protocol: https
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo', 'workflow'
```

```sh
# 透過自然語言生成 CLI 指令建議 (ghcs = GitHub Copilot Suggest)
ghcs -h

# 透過自然語言生成 CLI 指令解釋 (ghce = GitHub Copilot Explain)
ghce -h
```

### 安裝 [AIChat](https://github.com/sigoden/aichat) 工具

[AIChat: All-in-one LLM CLI Tool](https://github.com/sigoden/aichat) 是一個多合一的 LLM CLI 工具，具有 Shell 助理、CMD & REPL 模式、RAG、AI 工具與代理等功能，讓你再也不必記憶指令。

以下這個命令會將 `aichat` 最新版下載並解壓縮到 `/usr/local/bin/aichat` 路徑下：

```sh
AIChatVersion=$(curl -s "https://api.github.com/repos/sigoden/aichat/releases/latest" | jq -r .tag_name)
bash -c "
  curl -sL https://github.com/sigoden/aichat/releases/download/${AIChatVersion}/aichat-${AIChatVersion}-x86_64-unknown-linux-musl.tar.gz \
    | tar -xzO aichat > /usr/local/bin/aichat && chmod +x /usr/local/bin/aichat
"
aichat -V
```

設定使用 Gemini 來讓 [AIChat](https://github.com/sigoden/aichat) 回答問題，你可以從 <https://ai.dev/app/apikey> 取得免費金鑰：

```sh
cat <<'EOF' | tee -a ~/.profile > /dev/null
export GEMINI_API_KEY='YOUR_GEMINI_API_KEY'
export AICHAT_PLATFORM=gemini
# 追求高品質回應，可用 gemini-2.5-pro 模型
#export AICHAT_MODEL=gemini:gemini-2.5-pro
# 追求高效率回應，可用 gemini-2.5-flash-lite-preview-06-17 模型
export AICHAT_MODEL=gemini:gemini-2.5-flash-lite-preview-06-17
EOF

source ~/.profile
```

> 💡 完整設定可以參見 <https://github.com/sigoden/aichat/wiki/Environment-Variables>

第一次用 [AIChat](https://github.com/sigoden/aichat) 要先更新模型清單：

```sh
aichat --sync-models

aichat hi
```

### 安裝 [Codex CLI](https://github.com/openai/codex) 程式設計代理人工具

```sh
npm install -g @openai/codex

# 使用 ChatGPT 登入 Codex CLI
codex login
```

開啟畫面上的登入連結，走完 OAuth 授權流程後，將瀏覽器最後的網址透過以下命令在容器內執行：

```sh
curl -v -L 'http://localhost:1455/success?xxxxxxplatform.openai.com'
```

> 💡 目前 (2025-07-12) 這個 codex `0.5.0` 版本完全無法使用 Azure OpenAI Services 的端點。如果要使用 Azure OpenAI Services 的端點與金鑰，必須參考 [OpenAI_Codex_CLI.md](OpenAI_Codex_CLI.md) 從原始碼開始建置 `codex` 程式才能用。

如果要設定 Azure OpenAI Services 的端點與金鑰，我原本預期可以這樣設定

1. 建立 `AZURE_OPENAI_API_KEY` 環境變數
2. 建立 `~/.codex/config.toml` 設定檔

但目前 `v0.5.0` 這樣的設定完全不能用：

```sh
mkdir -p ~/.codex

cat <<'EOF' | tee ~/.codex/config.toml > /dev/null
model_provider = "azure"

[model_providers.azure]
name = "Azure"
base_url = "https://YOUR-RESOURCE-NAME.openai.azure.com/openai"
env_key = "AZURE_OPENAI_API_KEY"
query_params = { api-version = "2025-04-01-preview" }
wire_api = "responses"

[profiles.codex-mini]
model_provider = "azure"
model = "codex-mini"

[profiles.o4-mini]
model_provider = "azure"
model = "o4-mini"
EOF

```

### 安裝 [Gemini CLI](https://github.com/google-gemini/gemini-cli/) 程式設計代理人工具

我有翻譯一份 [Gemini CLI 使用手冊](https://gemini-cli.gh.miniasp.com/)，可以參考看看。

```sh
npm install -g @google/gemini-cli && gemini -v

# 設定 Gemini CLI 登入
gemini
```

安裝 Gemini CLI 設定精靈

```sh
mkdir -p ~/.local/bin && curl -sSL https://github.com/doggy8088/gemini-init/raw/main/gemini-init -o ~/.local/bin/gemini-init && chmod +x ~/.local/bin/gemini-init
```

之後在任意資料夾就可以用以下命令快速初始化常用 Gemini CLI 設定：

```sh
gemini-init
```

> 此命令會寫入 `.gemini/settings.json` 設定檔。

### 安裝 [Claude Code](https://www.anthropic.com/claude-code) 程式設計代理人工具

```sh
npm install -g @anthropic-ai/claude-code

# 設定任務完成的鈴聲
claude config set --global preferredNotifChannel terminal_bell

# 設定暗黑佈景主題
claude config set -g theme dark

# CI 模式必須關閉 verbose
claude config set -g verbose false

# 啟動程式
claude
```

### 安裝 [uv](https://docs.astral.sh/uv/getting-started/installation/) 工具 (Python)

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
uv -V
```

如果要執行一個 Python 程式，可以這樣執行：

```sh
uv run main.py
```

### 安裝 Google Cloud SDK

```sh
echo "Installing Google Cloud SDK..."
curl -sSL https://sdk.cloud.google.com | bash
source  ~/.bashrc
gcloud init
```
