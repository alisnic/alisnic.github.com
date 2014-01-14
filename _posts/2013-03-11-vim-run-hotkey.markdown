---
layout: post
title: Vim hotkey to run current file depending on the filetype
category: posts
---

So, here's the thing, you're editing a file and you need to run it. And, since
you are in vim, you might want to be clever and do something like
`:map <F5> :!ruby %<cr>` on the fly. But, you can't ever be clever enough in vim.

[Autocmd][1] is a feature of vim which allows to execute a command when an
event occurs. In our context, we need vim to map a key to execute the current
file in ruby if it is a ruby script. All we need is to add the following statement
to `.vimrc`:

    autocmd FileType ruby nmap <F5> :!ruby %<cr>

This means that when the "FileType" variable in vim will be set to ruby, the
key `F5` will be mapped to run current file with ruby. This is very convenient
because usually that variable is set by syntax detectors.

Then, we can extend this and add new file types:

```vim
autocmd FileType ruby nmap <F5> :!ruby %<cr>
autocmd FileType coffee nmap <F5> :!coffee %<cr>
```

Neat, huh? Based on the file type, vim will execute the current file in the
corresponding interpreter.

### Taking it a bit further

Also, it would be nice to save the file each time you want to run it, and also
measure the time it took to run. For that, you will need a function:

```vim
function RunWith (command)
  execute "w"
  execute "!clear;time " . a:command . " " . expand("%")
endfunction

autocmd FileType coffee nmap <F5> :call RunWith("coffee")<cr>
autocmd FileType ruby   nmap <F5> :call RunWith("ruby")<cr>
```

`RunWith` is a function which accepts the executable name which will be used to
execute the current file. It will save the file before executing it, and will
clear the console screen.

**Pro tip:** auto map a rspec run hotkey.

```vim
autocmd BufRead *_spec.rb nmap <F6> :w\|!clear && rspec % --format documentation --color<cr>
```

(each time a file with `_spec.rb` is loaded, the editor will map the `F6` key
to execute the rspec on the current file)

[1]: http://vimdoc.sourceforge.net/htmldoc/autocmd.html
