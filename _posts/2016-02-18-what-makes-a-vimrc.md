---
layout: post
title: "What Makes a Vimrc"
date: 2016-02-18
categories: vim, development, commandline
---

Let's get this out of the way. The default vi editor is horrible. I think I first used it on my Raspberry Pi and, like many other people, had no clue what was going on.

{% include image.html img="trapped-vim.png" title="Default Vim setup" caption="A memory of my first Vim experience"%}

It's only after taking the time to experiment and explore the flexibility Vim offers can I believe that it's the best text editor out there. This post is meant to show off some things in my Vim setup that I think can be useful to you. Also, this post assumes you have some understanding of Vim (modes, movement, basic editing). Although [my `vimrc`](https://github.com/melinysh/vimrc) is on Github, I don't recommend copy and pasting all of it. It's too personalized to me, instead read this post first and build up your own. My `vimrc` can be used as an example of how to create a unique `vimrc` for one's needs. 

The default version of Vim that ships on Macs is out of date so I recommend using [Homebrew](http://brew.sh) to `brew install vim` and use that latest version. Some of the plugins below require newer versions of Vim.

# Make Vim Great Again
I'd seen countless people on the internet praise Vim and show off some pretty terminal screenshots of Vim. How do they do it? It's all in their magical Vim configuration file, `vimrc`, which is usually placed as a hidden file in the home directory (`~/.vimrc`). 

## The Basics
The very first thing I have in my `vimrc` is 

{% highlight viml %}
set nocompatible             " be iMproved
filetype on              
filetype plugin on

" Set utf8 as standard encoding 
set encoding=utf-8
" Use Unix as the standard file type
set ffs=unix,dos,mac

" enables syntax highlighting
syntax on 

" always show status line
set laststatus=2
" show line numbers
set number
" Enable mouse support in console
set mouse=a

set ruler
set showcmd
{% endhighlight %}

{% include image.html img="basic-vim.png" %}

Doing this sets up the basics. Next, an awesome feature of Vim is backups. Make sure you have a `.vim/tmp` and a `.vim/backup` directory setup to make use of Vim backups.

{% highlight viml %}
set backup
set writebackup
set backupdir=~/.vim/backup
set directory=~/.vim/tmp
{% endhighlight %}

You can set it and forget it and Vim will automatically make backups of the file you edit. Having these backups has saved me on more than one occasion.

## Mappings
Mappings take one Vim command and direct them to another. Some commands, like enabling spell check, can be cumbersome to type out everytime `:setlocal spell spelllang=en_ca` in Vim. 

Its easier to create a kind of shortcut key to activate our own commands in Vim. This key is called the [Leader](http://vi.stackexchange.com/questions/836/what-is-leader). I define my Leader as the Space key.

{% highlight viml %}
let mapleader = "\<Space>"
{% endhighlight %}

So we can remap the spelling command seen above to a much simpler command using `nnoremap`:
{% highlight viml %}
nnoremap <Leader>sp :setlocal spell spelllang=en_ca<CR> 
au BufRead *.txt,*.md setlocal spell
{% endhighlight %}

When it hit `<Space>sp` or when I open a `.txt` or `.md` file, spell check is enabled. The `<CR>` at the end represents the `<Enter>` key and makes sure the command gets executed. To disable it, I also use a custom mapping.

{% highlight viml %}
nnoremap <Leader>nsp :set nospell<CR>
{% endhighlight %}

Once you learn the features and commands of Vim and other plugins, see which ones you use the most and remap them to quicker commands. In the long run, it will save you plenty of time.

## Plugins
Plugins are programs that integrate and interface with Vim. To be able to use plugins, you need a plugin manager. I use and recommend [Vundle](https://github.com/VundleVim/Vundle.vim). Other options are [Pathogen](https://github.com/tpope/vim-pathogen) and [NeoBundle](https://github.com/Shougo/neobundle.vim). This post will use Vundle as the plugin manager and will use commands specific to Vundle. For any of the following plugins, you can read about their usage and settings by typing `:help [Plugin Name]` in Vim after being installed.

{% highlight viml %}
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'

" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
{% endhighlight %}


### [Airline](https://github.com/vim-airline/vim-airline)
Definitely the greatest Vim plugin. Airline is a "lean & mean status/tabline for Vim that's light as air." The default Vim status bar leaves a lot to be desired and Airline fills this void. It integrates with a bunch of other plugins and is very customizable. If you're looking for a first plugin, Airline is the place to start. 

![Airline Demo](https://raw.githubusercontent.com/wiki/vim-airline/vim-airline/screenshots/demo.gif)

With Vundle, add the plugin like so:

{% highlight viml %}
Plugin 'bling/vim-airline'
{% endhighlight %}

To get the nice symbols to appear, a patched font is required called powerline. You can read more about it [here](https://github.com/vim-airline/vim-airline#integrating-with-powerline-fonts). Enabling the fonts, after being installed, requires these option to be set:

{% highlight viml %}
let g:airline_powerline_fonts = 1
let g:Powerline_symbols = 'fancy'
{% endhighlight %}


### [YouCompleteMe](https://github.com/Valloric/YouCompleteMe)
YouCompleteMe is "a fast, as-you-type, fuzzy-search code completion engine for Vim." If you're used to writing code in an IDE, autocomplete is a logical, and sometimes essential feature to boost your productivity. YouCompleteMe currently works with C-family languages, Go, Rust, JavaScript, and Python.

![YCM Example](https://camo.githubusercontent.com/1f3f922431d5363224b20e99467ff28b04e810e2/687474703a2f2f692e696d6775722e636f6d2f304f50346f6f642e676966)

{% highlight viml %}
Plugin 'Valloric/YouCompleteMe'

" Settings for integration with Airline
let g:ycm_error_symbol = '!!'
let g:ycm_warning_symbol = '>>'

" Additional autocomplete settings
let g:ycm_complete_in_comments = 0
let g:ycm_complete_in_strings = 1

" What's this?
let g:ycm_global_ycm_extra_conf = '~/.ycm_extra_conf.py'
{% endhighlight %}

There are plenty of customizable settings for YCM and I encourage you to read through the help menu to see them all. For YCM to complete C-family languages, it requires an addtional configuration file to function, `.ycm_extra_conf.py`. There is an addtional Vim plugin that will generate this file for you, provided that there is a Makefile for your current code project in the current directory. I also remap this generation command.

{% highlight viml %}
" For generating file required for YouCompleteMe
Plugin 'rdnetto/YCM-Generator' 
" Remap to Control-Y
nnoremap <C-y> :YcmGenerateConfig <CR>
{% endhighlight %}

Ok...but what about the Makefile? I still have to generate that for each project. This is where a project of mine started from. [Pymake](https://github.com/Melinysh/PyMake) will automatically generate the Makefile for you based on the files in your current working directory. The global config file you see in the code snippet above is a fallback config file that may not always be the right fit for the project I'm working in. To tie everything together, it made complete sense to script the whole thing in my `vimrc`:

{% highlight viml %}
function SetupDevEnvironment()
	" make sure dev environment hasn't already been setup
	if !empty(glob("./Makefile")) || !empty(glob(".ycm_extra_conf.py"))
		echom "It appears that your dev environment is already setup."
		return
	endif
	" save existing file
	silent write 
	" generate Makefile in background
	silent !pymake	
	" generate YCM config in background
	silent YcmGenerateConfig 
	" Restart YCM so it detects new config
	silent YcmRestartServer
	redraw! 
	echom "Developer environment created!"
endfunction

nnoremap <Leader>sd :call SetupDevEnvironment() <CR>

nnoremap <Leader>f :YcmCompleter FixIt <CR>
nnoremap <Leader>d :YcmCompleter GetDoc <CR>
nnoremap <Leader>t :YcmCompleter GetType <CR>
{% endhighlight %}

In addition to the function, I have a few YCM specific mappings. The first is the most useful. When YCM and detects a syntax error or another minor error, I can place my cursor on that current line, hit `<Leader>f` and it will automatically correct the error for me!

### [Syntastic](https://github.com/scrooloose/syntastic)
Syntastic is a syntax checking plugin for Vim that utilizes external syntax checkers and displays any errors to to the user with tight integration with Airline. Syntastic can also automatically check your syntax everytime you write to disk. You can customize the feedback mechanism and have it display error messages in your statusline.

{% highlight viml %}
Plugin 'scrooloose/syntastic'
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*
{% endhighlight %}

![Syntastic Example](https://raw.githubusercontent.com/scrooloose/syntastic/master/_assets/screenshot_1.png)

### [Fugitive](https://github.com/tpope/vim-fugitive)
There is no other plugin that comes close to Fugitive for a git wrapper for Vim. My entire workflow in git can be done through Fugitive, without leaving the comfort of Vim.  

{% highlight viml %}
Plugin 'tpope/vim-fugitive'
" Sample of my fugitive-specific mappings
nnoremap <Leader>gc :Gcommit<CR>
nnoremap <Leader>gd :Gdiff<CR>
nnoremap <Leader>gb :Gblame<CR>
nnoremap <Leader>gs :Gstatus<CR>
{% endhighlight %}

### [NERD Tree](https://github.com/scrooloose/nerdtree)
NERD Tree is a plugin to help you explore your filesystem and open files in Vim. Its very helpful for quickly switching between files and directories. 

{% highlight viml %}
Plugin 'scrooloose/nerdtree'
" Close NERDTree if its only window open
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") && b:NERDTreeType == "primary") | q | endif 
" These two open NERDTree on start if no file specified on launch
autocmd StdinReadPre * let s:std_in=1 
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
" Toggle NERDTree on and off
nnoremap <C-n> :NERDTreeToggle <cr> 
{% endhighlight %}

The long command is too tedious to type everytime. Mappings to the rescue. Sometime I start Vim, but don't specify the file, so I have NERD Tree start if no file is specified and I can quickly find my way around my file system.

{% include image.html img="nerdtree.gif" %}

## Color Schemes
[VimColors.com](http://vimcolors.com) highlights a wide variety of colorschemes for Vim. Take a look and choose one that fits you. I use [Molokai](https://github.com/tomasr/molokai) and like it (it's the scheme in the gif above). Choosing the right color scheme can brighten up your Vim setup and make it more enjoyable to work in.  

## Vim Jokes  
Here's a collection of some great Vim jokes I've seen around the Internet.
![Quitting Vim](https://pbs.twimg.com/media/CCvkKq7WYAAAvkc.png)
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">I&#39;ve been using Vim for about 2 years now, mostly because I can&#39;t figure out how to exit it.</p>&mdash; I Am Devloper (@iamdevloper) <a href="https://twitter.com/iamdevloper/status/435555976687923200">February 17, 2014</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">NO GRANDPA<br>JUST HIT THE POWER BUTTON<br>YOU CANT EXIT VIM</p>&mdash; SecuriTay (@SwiftOnSecurity) <a href="https://twitter.com/SwiftOnSecurity/status/611247035925164032">June 17, 2015</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Vim is a lot more fun if you treat it like a text adventure game:<br><br>&gt; EXIT<br>- You remain stationary <br>&gt; ESC<br>- You remain stationary<br>&gt; HELP</p>&mdash; I Am Devloper (@iamdevloper) <a href="https://twitter.com/iamdevloper/status/480056139988893697">June 20, 2014</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<br> 

* * * 

<br>  
The plugins, mappings, and configurations you see above just scratch the surface. I encourage you to tinker and experiment with Vim and the numerous plugins out there. I've tried to cover the basics of Vim customizations that will work for almost everybody, but there are many more customizations that can help you and your workflow, you just need to find them. :wq
