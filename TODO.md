# TODO

## 預計加入的工具清單

1. [Claude Code](https://www.anthropic.com/claude-code)

    ```sh
    npm install -g @anthropic-ai/claude-code

    # 設定任務完成的鈴聲
    claude config set --global preferredNotifChannel terminal_bell

    # 設定暗黑佈景主題
    claude config set -g theme dark

    # CI 模式必須關閉 verbose
    claude config set -g verbose false
    ```

2. [eza](https://github.com/eza-community/eza) & [eza-themes](https://github.com/eza-community/eza-themes)

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
