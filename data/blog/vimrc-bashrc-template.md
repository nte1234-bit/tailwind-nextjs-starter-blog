---
title: '作業サーバに最初に置く .vimrc と .bashrc'
date: '2026-06-03'
tags: ['vim', 'bash', 'linux', 'dotfiles']
draft: false
summary: '踏み台サーバ越しに作業する環境向けに、作業サーバの .vimrc と .bashrc のテンプレートをまとめる。ファイル転送ができない環境からも、ブログで見ながらコピペできる。'
---
## はじめに

踏み台サーバ越しに作業サーバへログインして開発する現場では、  
ファイルを業務PCに転送できないことが多い。  

ローカルで設定ファイルを準備して持ち込む、という普通のやり方が使えない。

このメモは、そういう環境向けに **作業サーバに最初に置く `.vimrc` と `.bashrc`** をまとめたものだ。  
会社PCのブラウザで開いて、見ながらコピペすればそのまま動く。

## 前提

- RHEL系 OS（CentOS、RHEL、Rocky Linux など）+ bash を想定
- 作業サーバが自分専用、または設定ファイルを置いてよい状態
- 共用の踏み台サーバには置かず、作業サーバ側に置く

## .vimrc

`~/.vimrc` として作業サーバのホームディレクトリに置く。

```vim
" ========================================
" 基本設定
" ========================================
set number              " 行番号表示
set ruler               " カーソル位置を画面右下に表示
set showmatch           " 対応する括弧をハイライト
set showcmd             " 入力中のコマンドを画面右下に表示
set wildmenu            " コマンドモードの補完を強化
set laststatus=2        " ステータスラインを常に表示

" ========================================
" 検索
" ========================================
set ignorecase          " 検索時に大文字小文字を区別しない
set smartcase           " 大文字を含む検索のみ区別する
set incsearch           " インクリメンタル検索
set hlsearch            " 検索結果をハイライト

" Esc連打でハイライト解除
nnoremap <silent> <Esc><Esc> :nohlsearch<CR>

" ========================================
" インデント
" ========================================
set autoindent          " 自動インデント
set smartindent         " スマートインデント
set expandtab           " タブをスペースに展開
set tabstop=4           " タブ幅
set shiftwidth=4        " 自動インデントの幅
set softtabstop=4

" ========================================
" 表示
" ========================================
syntax on               " シンタックスハイライト
set background=dark     " 背景が暗い前提で配色
set cursorline          " 現在行をハイライト

" ========================================
" その他
" ========================================
set backspace=indent,eol,start  " Backspaceで何でも消せる
set clipboard=unnamed           " システムクリップボード連携
set mouse=a                     " マウス操作有効
set encoding=utf-8
set fileencodings=utf-8,euc-jp,sjis,cp932
set fileformats=unix,dos,mac

" 保存時の余分な空白を削除
autocmd BufWritePre * :%s/\s\+$//ge

" 行末の空白を可視化
set list
set listchars=tab:▸\ ,trail:·,nbsp:%
```

### この設定で得られること

- **行番号** が表示される。エラー行と照合しやすい
- **大文字小文字を含む検索だけ区別する** スマート検索。日常的な検索は気楽になる
- **Esc 2回連打で検索ハイライトを消す**。検索後に画面が色だらけのまま、というイライラが消える
- **保存時に行末の空白を自動削除**。コミット差分が綺麗になる
- **行末空白とタブを可視化**。`·` がスペース、`▸` がタブ。インデントの混在に気づける

タブ幅は4スペースに設定している。プロジェクトの規約と合わない場合は `tabstop`、`shiftwidth`、`softtabstop` を変える。

## .bashrc 追記

既存の `.bashrc` の **末尾に追記** する。既存内容は消さない。

```bash
# ========================================
# ヒストリ強化
# ========================================
# 履歴サイズを増やす
export HISTSIZE=10000
export HISTFILESIZE=20000

# 重複と先頭スペースを履歴に含めない
export HISTCONTROL=ignoreboth:erasedups

# タイムスタンプを履歴に含める
export HISTTIMEFORMAT='%Y-%m-%d %H:%M:%S '

# 複数セッション間で履歴を共有
shopt -s histappend
PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"

# 上矢印で入力中の文字に部分マッチする履歴を遡る
bind '"\e[A": history-search-backward'
bind '"\e[B": history-search-forward'

# ========================================
# よく使うエイリアス
# ========================================
alias ll='ls -lah --color=auto'
alias la='ls -A --color=auto'
alias l='ls -CF --color=auto'
alias ls='ls --color=auto'
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'

# ディレクトリ移動の省略形
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'

# 安全策（誤削除防止）
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# よく使うコマンド
alias h='history'
alias c='clear'
alias df='df -h'
alias du='du -h'
alias free='free -h'

# 現在のディレクトリのサイズ TOP10
alias dut='du -sh * 2>/dev/null | sort -hr | head -n 10'

# ========================================
# プロンプト
# ========================================
# 緑のユーザ名 + 青のホスト名 + パス + $
PS1='\[\e[32m\]\u\[\e[0m\]@\[\e[34m\]\h\[\e[0m\]:\[\e[33m\]\w\[\e[0m\]\$ '

# ========================================
# その他
# ========================================
# less でカラー表示
export LESS='-R'

# vim をデフォルトエディタに
export EDITOR=vim

# 文字化け対策
export LANG=ja_JP.UTF-8
```

### この設定で得られること

**ヒストリ強化が最大の価値**だ。  
`curl -X` と入力した状態で上矢印キーを押すと、過去の `curl -X` で始まるコマンドだけが遡れる。一度この挙動を経験すると、素のbashには戻れない。

**エイリアス** で日常コマンドを省タイプ。`ll` で詳細表示、`..` で1階層上、`...` で2階層上。`rm` `cp` `mv` は確認モード（`-i`）にして誤削除を防ぐ。

**プロンプト** はユーザ名・ホスト名・現在地を色分け表示する。複数の作業サーバを行き来する時、「いま自分はどのサーバに居るのか」が一目で分かる。これは予想以上に効く。

## 反映方法

`.vimrc` の反映：

```bash
vim ~/.vimrc
```

`i` で挿入モードに入って中身を貼り付け、`Esc` → `:wq` で保存して終了。

`.bashrc` の反映：

```bash
vim ~/.bashrc
```

`G` でファイル末尾にジャンプ、`o` で次の行から挿入モード、追記内容を貼り付け、`Esc` → `:wq`。

最後に現在のシェルにも反映：

```bash
source ~/.bashrc
```

これで設定が即時反映される。`.vimrc` は次回 `vim` を起動した時から反映される。

## まとめ

このテンプレは、私自身が踏み台越しの作業環境で使っているものだ。設定ファイルを業務PCに送れない制約があるので、ブログに置いて自分でも参照する。

設定ファイルを置くだけで作業サーバの居心地が変わる。特にヒストリ検索の強化は、一度体験すると元には戻れない。同じような環境で作業している人がいれば、コピペして試してみてほしい。