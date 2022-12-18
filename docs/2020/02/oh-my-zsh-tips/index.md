# Oh My Zsh Tips


## Installation

1. `sudo apt install -y zsh`
2. `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
3. `git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`


## Customized Configuration

1. In `~/.zshrc`, set `ZSH_THEME` to `robbyrussell`.
2. Modify `plugins` to:
    ```bash
    plugins=(
      git
      zsh-autosuggestions
    )
    ```
3. Append below lines to `~/.zshrc`.
    ```bash
    export ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=10'

    export GOPATH=~/git/go
    export PATH=$PATH:$GOPATH/bin:/snap/bin

    # Jupyter notebook
    alias inote='cd /mnt/hgfs/liangr-win10/git/ipython && \
        nohup /home/liangr/git/ml/venv/py36/bin/jupyter notebook \
        --ip 0.0.0.0 --port 19032 --no-browser \
        &> /mnt/hgfs/liangr-win10/git/ipython/notebook.log &'   
    ```
4. Update `~/.oh-my-zsh/themes/robbyrussell.zsh-theme`.
```bash
local ret_status="%(?:%{$fg_bold[green]%}❯:%{$fg_bold[red]%}❯)"
PROMPT='%{$fg[cyan]%}%~ $(git_prompt_info)'$'\n''${ret_status}%{$reset_color%} '
ZSH_THEME_GIT_PROMPT_PREFIX="%{$fg_bold[blue]%}git:(%{$fg[red]%}"
ZSH_THEME_GIT_PROMPT_SUFFIX="%{$reset_color%} "
ZSH_THEME_GIT_PROMPT_DIRTY="%{$fg[blue]%}) %{$fg[yellow]%}✗"
ZSH_THEME_GIT_PROMPT_CLEAN="%{$fg[blue]%}) %{$fg[green]%}✓"
```

