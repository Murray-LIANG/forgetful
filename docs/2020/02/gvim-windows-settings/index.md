# GVIM Settings on Windows


## Copy the vimrc from devenv git repo to Windows home directory, name it `_gvimrc`
The reason to give the name `_gvimrc` is to avoid the vim in git bash to load it.
This would let the vim in git bash lighter and faster.

## Run below commands in git bash (copied from all_set.sh of devenv)

```bash
mkdir -p ~/.vim/autoload ~/.vim/bundle ~/.vim/colors \
    && git clone -q https://github.com/altercation/vim-colors-solarized ~/.vim/bundle/vim-colors-solarized \
    && ln -s ~/.vim/bundle/vim-colors-solarized/colors/solarized.vim ~/.vim/colors/solarized.vim \
    && git clone -q https://www.github.com/VundleVim/Vundle.vim ~/.vim/bundle/Vundle.vim \
    && vim +PluginInstall +qall \
    && git clone -q --depth=1 https://github.com/powerline/fonts.git ~/Downloads/fonts
```

## Add guifont setting in _gvimrc

```bash
set guifont=DejaVu_Sans_Mono:h12
```
