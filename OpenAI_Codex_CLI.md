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
