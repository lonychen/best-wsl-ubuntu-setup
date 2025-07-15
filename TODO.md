# TODO

## 預計加入的工具清單

### Playwright

```sh
sudo npx playwright install-deps
```

### 安裝 [eza](https://github.com/eza-community/eza) & [eza-themes](https://github.com/eza-community/eza-themes) 工具

一套 `ls` 的現代替代品。

```sh
# Install by cargo
# https://github.com/eza-community/eza/blob/main/INSTALL.md
cargo install eza

# Setup theme
cd ~
git clone https://github.com/eza-community/eza-themes.git
mkdir -p ~/.config/eza
ln -sf "$(pwd)/eza-themes/themes/catppuccin.yml" ~/.config/eza/theme.yml

# Setup ll alias
alias ll='eza -laF'
```

PowerShell

```ps1
mkdir "$env:USERPROFILE\.config\eza"
$env:EZA_CONFIG_DIR = "$env:USERPROFILE\.config\eza"

# https://github.com/eza-community/eza-themes/blob/main/themes/dracula.yml
notepad "$env:USERPROFILE\.config\eza\theme.yml"

Remove-Alias ll
function ll {
    $env:EZA_CONFIG_DIR = "$env:USERPROFILE\.config\eza"
    eza -laF $args
}
```
