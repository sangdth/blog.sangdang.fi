## coc-explorer review

Recently I found out `coc-explorer` has float mode, which is really useful.

In case you don't know what is it, `coc-explorer` is a file explorer for neovim, but I think it could work with normal vim despite I didn't test.

First step, of course,  [install it](https://github.com/weirongxu/coc-explorer#usage) , please go to the GitHub doc.

Then add a preset into `coc-settings.json` (you can open it by using `:CocConfig` command.)

```
"explorer.presets": {
    "preset-name-here": {
          "position": "floating",
          "floating-position": "center",
          "floating-width": 100,
          "content-width": -10,
          "floating-height": -5,
          "open-action-strategy": "sourceWindow"
    }
}
````

After that you can use it (in .vimrc):
```
nmap <c-e> :CocCommand explorer --preset preset-name-here<CR>
```

Note that you can change `preset-name-here` with anything you want. In my settings, I have 2 presets, one named `float` and one named `static`. It's easier to debug in the future.

Now you can press `control` + `e` to open the file explorer, it should float on top of everything.

Happy vimming :)