## Move the quickfix window to the right side

It's quite a bad experience when working with quickfix window, I guess because back in the old days, the screen was small, so putting the quickfix below by default was a good decision, but we have wide screens nowadays, so let's move the quickfix to the right side.

First note the `:set splitright` does not apply to quickfix. It only splits the current window into the right, not the quickfix.

Then I found out that we can move the quickfix window with the sequence `ctrl-w` + `shift-j`, and it is equivalent with `wincmd L`

### Let's write some vimscript :D

Add those lines into your `.vimrc`:
```
function! <SID>MoveQuickFixRight()
    wincmd L             " After open, move the window
    vertical resize 40   " split will make it too wide
endfunction
```
Then execute it with auto-command:

```sh
augroup
    autocmd FileType qf call <SID>MoveQuickFixRight()
augroup end
```

### Result <3
![open quickfix on the right side](https://cdn.hashnode.com/res/hashnode/image/upload/v1614421616797/kDFM9w5xr.png)

Now, open your quickfix and voila, it is on the right side of your current window :D

Happy vimming!