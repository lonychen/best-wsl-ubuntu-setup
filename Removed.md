# 已移除的設定

## GitHub Copilot CLI

移除理由: 執行效能太差，而且年久失修，建議用 [AIChat](https://github.com/sigoden/aichat) 取代！

```sh
gh extension install github/gh-copilot
gh extension upgrade github/gh-copilot

gh copilot --help

echo 'eval "$(gh copilot alias -- bash)"' >> ~/.bashrc

# 透過自然語言生成 CLI 指令建議 (ghcs = GitHub Copilot Suggest)
ghcs -h

# 透過自然語言生成 CLI 指令解釋 (ghce = GitHub Copilot Explain)
ghce -h
```
