# memolist.vimのgrepをDeniteのgrepにする

[memolist.vim](https://github.com/glidenote/memolist.vim)という便利なmemo pluginがあります。
軽量で使いやすくオススメなのですが、ここがなーと思っている箇所がありました。
MemoGrepというcommandが定義されているのですが、この実装はVimGrepをしているのです。

```
function! memolist#grep(word)
  let word = a:word
  if word == ''
    let word = input("MemoGrep word: ")
  endif
  if word == ''
    return
  endif

  try
    if get(g:, 'memolist_qfixgrep', 0) != 0
      exe "Vimgrep -r" s:escarg(word) s:escarg(g:memolist_path . "/*")
    else
      exe "vimgrep" s:escarg(word) s:escarg(g:memolist_path . "/*")
    endif
  catch
    redraw | echohl ErrorMsg | echo v:exception | echohl None
  endtry
endfunction
```

あれ？このオプション`let g:memolist_denite = 1`でDenite grepしてくれると思ってたのに…(´・ω・`)
ゆるふわ系VimmerとしてはVimGrepはハードボイルド過ぎるのでDenite grepをするようにしてみました。

```
" 適当にvimrcにでも書いて下さい

function! s:memoGrepDenite()
    let l:option = '-path=' . expand('$HOME/path/to/memolist')
    exe 'Denite' 'grep' l:option '-highlight-mode-insert=Search'
endfunction

command! MemoGrepDenite call s:memoGrepDenite()<CR>
```

これで:MemoGrepDeniteコマンドができて、ホクホク(・∀・)
