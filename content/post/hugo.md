+++
date = "2015-04-02"
title = "VIM Plugin for Markdown"
description = "VIM plugin for writing and learning markdown syntax"
keywords = ["panos georgiadis", "hugo", "blog", "static", "html generator"]
menu = "How-To"
+++


In the early beginning, I found writing posts in markdown format quite tricky.
Things can become quite helpful by using an editor that
comes with features such as **syntax highlighting** and **HTML
monitoring**. Apparently, there are noumerous plugins out there, but for
me, *'vim-instant-markdown'* seems to be sufficient.



### Install

Proper installation via *Vundle*, requests to edit your `.vimrc` and put the **git**
source of the plugin:

```vim
 " plugin for MarkDown
Plugin 'suan/vim-instant-markdown'
```



#### Force `.md` files to be treated as `.markdown`

By default, only the `.markdown` extension is typically recognized as
markdown filetype. Given the fact that I'm one of those people who prefer to use
 the  `.md` extension as an alternative; VIM also needs to be configured. To
treat the content of my *$post.md* like any other *$post.markdown*, add this to `.vimrc`:

```vim
" Detect *.md as *.markdown
autocmd BufNewFile,BufReadPost *.md set filetype=markdown
```

To test, edit/create a file with `.md` extension and observe if the syntax has been
highlighted or not. Also, your web-browser should immediately pop-up and start
generating -- *on-the-fly* -- a webpage, based on the markdown syntax of the file.
