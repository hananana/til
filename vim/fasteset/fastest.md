# 最速最堕落なUnity開発環境構築

ぼくはせっかちで怠惰です。
なるたけ己は考えず、手を動かさず、待たず、物事を推し進めるのが理想だと思っています。
それはつまり、「今、解かねばならない問題だけを考えれる」環境が良いでしょう。
そんな環境とは...

- 手はキーボードから動かさない
- ホームポジションからも動かさない
- 自分用のCLIツールを作り、作業を自動化する
- ２度同じコマンド（作業）はやらない。自動化する
- 待たない
- Buildなどのどうしても時間がかかるものはしょうがないのでコーヒーを飲んで騎士団長殺しを読んでリラックスする

上記を実現するにはさまざまな方法があるかと思いますが、
この記事では**vim**と**zsh**を使って実現する方法を話します。

## May the vim and zsh be with you

Xcodeでプロジェクトを開いて最初の一文字を打つまでに何秒かかりますか？ぼくは22秒かかりました。
Android Studioだと何秒かかりますか？ぼくは27秒かかりました。
さて、じゃあIDEは閉じましょう。GUIでなければできない作業以外はzshでやります。
CLIで作業は自動化するのです。
同じ作業をマウスでポチポチすることに怒りを覚えなければなりません。
scriptを作り自動化してください。
コマンドを打ってmacに働いてもらってる間、今日のランチは一風堂にするか豚野郎にするか真剣に考えるべきです。

### Setup Zsh

```
# MacOSに標準で入ってるヤーツでも大丈夫です
$ brew install zsh

# agは必須
$ brew install the_silver_searcher

# なんでもpecoってしまいましょう
$ brew install peco

$ touch ~/.zshrc

# zplugで各種拡張を入れてどんどん便利にしていきます
$ curl -sL --proto-redir -all,https https://zplug.sh/installer | zsh
```

#### zshrc
```
source ~/.zplug/zplug
zplug "b4b4r07/enhancd", of:init.sh
zplug "zsh-users/zsh-completions", of:src
zplug "mollifier/cd-gitroot"
zplug "zsh-users/zsh-syntax-highlighting", of:zsh-syntax-highlighting.zsh

if ! zplug check --verbose; then
    printf "Install? [y/N]: "
    if read -q; then
        echo; zplug install
    fi
fi
zplug load --verbose

# zshの補完をsmartcaseにする
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}' 'r:|.=*'

# pyenvの設定
export PYENV_ROOT="${HOME}/.pyenv"
export PATH=${PYENV_ROOT}/bin:$PATH
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

### Setup Python

自動で作業をやるためのscriptですが、様々な選択肢があります。
shellscript, ruby, python, go, etc...
それぞれ好きな言語でいいじゃない、と思いますがvimとの相性を見るに
goかpythonがいいのではないかと思います。
ぼくはpythonを選びました。
なのでここではpythonのための設定を書きます。

先の話になりますが、最速補完を実現するためにYouCompleteMeを使います。
その場合pyenv経由でinstallする際に注意が必要です。
[公式](https://github.com/Valloric/YouCompleteMe)にきちんと書かれていますが念のため。

```
$ brew install pyenv
$ brew install pyenv-virtualenv
$ PYTHON_CONFIGURE_OPTS="--enable-framework" pyenv install 3.6.0
$ pyenv global 3.6.0
$ pyenv rehash
```

MacOSならば初めからpython入ってるじゃん！なのですが、
python2系はオワコン宣言がなされたのでpython3系を使いたいのです。

### Setup Vim

やっとvimの設定です。
要となるのは高速なんでも検索のctrlpとag
そして最速補完なYouCompleteMeです。

```
brew install vim --with-client-server --with-override-system-vi
touch $HOME/.vimrc
```

#### .vimrc
```
filetype plugin indent off

if has('vim_starting')
    set runtimepath+=~/.vim/plugged/vim-plug
    if !isdirectory(expand('~/.vim/plugged/vim-plug'))
        echo 'install vim-plug...'
        call system('mkdir -p ~/.vim/plugged/vim-plug')
        call system('git clone https://github.com/junegunn/vim-plug.git ~/.vim/plugged/vim-plug/autoload')
    end
endif

call plug#begin('~/.vim/plugged')

Plug 'ctrlpvim/ctrlp.vim'
Plug 'LeafCage/yankround.vim'
Plug 'rking/ag.vim'
Plug 'Valloric/YouCompleteMe', {'do': './install.py --clang-completer --omnisharp-completer'}
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'

Plug 'junegunn/vim-plug', {'dir': '~/.vim/plugged/vim-plug/autoload'}


call plug#end()

filetype plugin indent on

```

# 参考
