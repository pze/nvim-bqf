# nvim-bqf

The goal of nvim-bqf is to make Neovim's quickfix window better.

<p align="center">
    <img width="864px" src=https://user-images.githubusercontent.com/17562139/105016283-6b260e00-5a7d-11eb-8e5f-cd4e034e2d14.gif>
</p>

## Table of contents

* [Table of contents](#table-of-contents)
* [Features](#features)
* [Todo](#todo)
* [Quickstart](#quickstart)
  * [Requirements](#requirements)
  * [Installation](#installation)
  * [Minimal configuration](#minimal-configuration)
  * [Usage](#usage)
    * [fzf mode](#fzf-mode)
    * [Search and replace example](#search-and-replace-example)
* [Documentation](#documentation)
  * [Function table](#function-table)
  * [Buffer Commands](#buffer-commands)
  * [Commands](#commands)
  * [Quickfix context](#quickfix-context)
    * [Why use an additional context?](#why-use-an-additional-context?)
    * [Supported keys](#supported-keys)
    * [Simple vimscript tests for understanding](#simple-vimscript-tests-for-understanding)
  * [Highlight groups](#highlight-groups)
* [Advanced configuration](#advanced-configuration)
  * [Customize configuration](#customize-configuration)
  * [Integrate with other plugins](#integrate-with-other-plugins)
* [Feedback](#feedback)
* [License](#license)

## Features

- Magic window make your eyes comfortable
- Extend built-in context of quickfix
- Fix some annoying bugs of quickfix
- Support convenient actions inside quickfix window
- Fast start time, compare with others lua plugins
- Integrate [fzf](https://github.com/junegunn/fzf) as a filter in quickfix window

## Todo

- [ ] Add tests
- [ ] Provide some useful functions to users
- [ ] Batch filter items with signs
- [ ] Find a better way to list history and switch to one
- [ ] Use context field to override the existed configuration

## Quickstart

### Requirements

- Neovim [nightly](https://github.com/neovim/neovim#install-from-source)
- [fzf](https://github.com/junegunn/fzf) (optional, 0.24.0 later)

> Preview with fzf needs a pipe command, Windows can't be supported. It must be stated that
> I'm not working under Windows.

### Installation

Install nvim-bqf with your favorite plugin manager! For instance: [Vim-plug](https://github.com/junegunn/vim-plug):

```vim
Plug 'kevinhwang91/nvim-bqf'
```

> The default branch is main, please upgrade vim-plug if you encounter any installation issues.

### Minimal configuration

```vim
Plug 'kevinhwang91/nvim-bqf'

" if you install fzf as system package like `pacman -S fzf` in ArchLinux,
" please comment next line
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }

" highly recommended
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
```

The nvim-bqf's preview builds upon the buffers. I highly recommended to use
[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) to do syntax to the buffer,
because vim's syntax is very lagging and is extremely bad for the user experience in large files.

### Usage

1. If you are familiar with quickfix, use quickfix as usual.
2. If you don't know quickfix well, you can run `:vimgrep /\w\+/j % | copen` under a buffer inside
   nvim to get started quickly.
3. If you want to taste quickfix like demo, check out [Integrate with other plugins](#integrate-with-other-plugins),
   and pick up the configuration you like.

#### fzf mode

Press `zf` in quickfix window will enter fzf mode.

fzf in nvim-bqf supports `ctrl-t`/`ctrl-x`/`ctrl-v` key bindings that allow you to
open up an item in a new tab, a new horizontal split, or in a new vertical split.

fzf becomes a quickfix filter and create a new quickfix list when multiple items are selected and
accepted.

#### Search and replace example

Using external grep-like program to search `qftool` and replace it to `mytool`,
but filter `fzf.lua` file.

<p align="center">
    <img width="960px" src="https://user-images.githubusercontent.com/17562139/105032702-16d95900-5a92-11eb-8e2d-8d57ca36e4fb.gif">
</p>

> Demonstrating batch undo just show that quickfix has this feature

## Documentation

```lua
root = {
    auto_enable = {
        description = [[enable nvim-bqf in quickfix window automatically]],
        default = true
    },
    magic_window = {
        description = [[give the window magic, when the window is splited horizontally, keep
            the distance between the current line and the top/bottom border of neovim unchanged.
            It's a bit like a floating window, but the window is indeed a normal window, without
            any floating attributes.]],
        default = true
    },
    auto_resize_height = {
        description = [[resize quickfix window height automatically.
            Shrink higher height to size of list in quickfix window, otherwise extend height
            to size of list or to default height (10)]],
        default = true
    },
    preview = {
        auto_preview = {
            description = [[enable preview in quickfix window automatically]],
            default = true
        },
        border_chars = {
            description = [[border and scroll bar chars, they respectively represent:
                vline, vline, hline, hline, ulcorner, urcorner, blcorner, brcorner, sbar]],
            default = {'│', '│', '─', '─', '╭', '╮', '╰', '╯', '█'}
        },
        delay_syntax = {
            description = [[delay time, to do syntax for previewed buffer, unit is millisecond]],
            default = 50
        },
        win_height = {
            description = [[the height of preview window for horizontal layout]],
            default = 15
        },
        win_vheight = {
            description = [[the height of preview window for vertical layout]],
            default = 15
        }
    },
    func_map = {
        description = [[see ###Function table for detail]]
    },
    filter = {
        fzf = {
            action_for = {
                ['ctrl-t'] = {
                    description = [[press ctrl-t to open up the item in a new tab]],
                    default = 'tabedit'
                },
                ['ctrl-v'] = {
                    description = [[press ctrl-v to open up the item in a new vertical split]],
                    default = 'vsplit'
                },
                ['ctrl-x'] = {
                    description = [[press ctrl-x to open up the item in a new horizontal split]],
                    default = 'split'
                }
            },
            extra_opts = {
                description = 'extra options for fzf',
                default = {'--bind', 'ctrl-o:toggle-all'}
            }
        }
    }
}
```

After loaded any modules, `lua print(vim.inspect(require('bqf.config')))` will show you all about
the current configuration.

You can set value on the fly without any validation, good luck!

### Function table

`Function` only works in the quickfix window

| Function    | Action                                             | Key     |
| ----------- | -------------------------------------------------- | ------- |
| open        | open the item under the cursor                     | `<CR>`  |
| openc       | like `open`, and close quickfix window             | `o`     |
| tab         | open the item under the curosr in a new tab        | `t`     |
| tabb        | like `tab`, but stay at quickfix window            | `T`     |
| split       | open the item under the cursor in vertical split   | `<C-x>` |
| vsplit      | open the item under the cursor in horizontal split | `<C-v>` |
| prevfile    | go to previous file under the cursor               | `<C-p>` |
| nextfile    | go to next file under the cursor                   | `<C-n>` |
| prevhist    | go to previous quickfix list                       | `<`     |
| nexthist    | go to next quickfix list                           | `>`     |
| pscrollup   | scroll up half-page in preview window              | `<C-b>` |
| pscrolldown | scroll down half-page in preview window            | `<C-f>` |
| pscrollorig | scroll back to original postion in preview window  | `zo`    |
| ptogglemode | toggle preview window between normal and max size  | `zp`    |
| ptoggleitem | toggle preview for an item of quickfix list        | `p`     |
| ptoggleauto | toggle auto preview when cursor move               | `P`     |
| fzffilter   | use fzf as a filter for quickfix list              | `zf`    |

### Buffer Commands

- `BqfEnable`: Enable nvim-bqf in quickfix window
- `BqfDisable`: Disable nvim-bqf in quickfix window
- `BqfToggle`: Toggle nvim-bqf in quickfix window

### Commands

- `BqfAutoToggle`: Toggle nvim-bqf enable automatically

### Quickfix context

Vim grant users an ability to stuff a context to quickfix, please run `:help quickfix-context` for detail.

#### Why use an additional context?

nvim-bqf will use the context to implement missing features of quickfix. If you are familiar with
quickfix, you know quickfix only contains `lnum` and `col` to locate a position in an item, but
lacks of range. To get better highlighting experience, nvim-bqf processeds the vim regrex pattern
and [lsp range](https://microsoft.github.io/language-server-protocol/specification#range) from the
context additionally.

The context's format that can be processed by nvim-bqf is:

```vim
" vimscript
let context = {'context': {'bqf': {}}}
```

```lua
-- lua
local context = {context = {bqf = {}}}
```

nvim-bqf only occupies a key of `context`, which makes nvim-bqf get along well with other plugins
in context of the quickfix window.

#### Supported keys

```lua
context = {
    bqf = {
        pattern_hl = {
            description = [[search pattern from current poistion]],
            type = 'string'
        },
        lsp_ranges_hl = {
            description = [[a list of lsp range. The length of list is equal to the items',
            and each element corresponds one to one]],
            type = 'list in vimscript | table in lua'
        }

    }
}
```

#### Simple vimscript tests for understanding

```vim
function s:create_qf()
    enew
    let bufnr = bufnr('%')
    for i in range(1, 3)
        call setline(i, i .. ' | ' .. strftime("%F"))
    endfor

    call setqflist([{'bufnr': bufnr, 'lnum': 1, 'col': 5},
                \  {'bufnr': bufnr, 'lnum': 2, 'col': 10},
                \  {'bufnr': bufnr, 'lnum': 3, 'col': 13}])
endfunction

function! Test_bqf_pattern()
    call s:create_qf()
    call setqflist([], 'r', {'context': {'bqf': {'pattern_hl': '\d\+'}},
                \ 'title': 'pattern_hl'})
    cwindow
endfunc

function! Test_bqf_lsp_ranges()
    call s:create_qf()
    let lsp_ranges = []
    call add(lsp_ranges, {
                \ 'start': {'line': 0, 'character': 4},
                \ 'end': {'line': 0, 'character': 8}
                \ })
    call add(lsp_ranges, {
                \ 'start': {'line': 1, 'character': 9},
                \ 'end': {'line': 1, 'character': 11}
                \ })
    call add(lsp_ranges, {
                \ 'start': {'line': 2, 'character': 12},
                \ 'end': {'line': 2, 'character': 14}
                \ })
    call setqflist([], 'r', {'context': {'bqf': {'lsp_ranges_hl': lsp_ranges}},
                \ 'title': 'lsp_ranges_hl'})
    cwindow
endfunc

" Save me, source me. Run `call Test_bqf_pattern()` and `call Test_bqf_lsp_ranges()`
```

nvim-bqf actually works with context in
[Integrate with other plugins](#integrate-with-other-plugins).

### Highlight groups

```vim
highlight default link BqfPreviewFloat Normal
highlight default link BqfPreviewBorder Normal
highlight default link BqfPreviewCursor Cursor
highlight default link BqfPreviewRange Search
```

- `BqfPreviewFloat`: highlight floating window
- `BqfPreviewBorder`: highlight border of floating window
- `BqfPreviewCursor`: highlight the cursor format `[lnum, col]` in preview window
- `BqfPreviewRange`: highlight the range format `[lnum, col, range]`, `pattern_hl` and `lsp_ranges_hl`

## Advanced configuration

### Customize configuration

```vim
call plug#begin('~/.config/nvim/plugged')

Plug 'kevinhwang91/nvim-bqf'

Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }

call plug#end()

highlight BqfPreviewBorder guifg=#50a14f ctermfg=71
highlight link BqfPreviewRange IncSearch

lua <<EOF
require('bqf').setup({
    auto_enable = true,
    preview = {
        win_height = 12,
        win_vheight = 12,
        delay_syntax = 80,
        border_chars = {'┃', '┃', '━', '━', '┏', '┓', '┗', '┛', '█'}
    },
    func_map = {
        vsplit = '',
        ptogglemode = 'z,'
    },
    filter = {
        fzf = {
            extra_opts = {'--bind', 'ctrl-o:toggle-all', '--prompt', '> '}
        }
    }
})
EOF
```

### Integrate with other plugins

```vim
call plug#begin('~/.config/nvim/plugged')

Plug 'kevinhwang91/nvim-bqf'

Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }

Plug 'neoclide/coc.nvim'

" :h CocLocationsChange for detail
let g:coc_enable_locationlist = 0
augroup Coc
    autocmd!
    autocmd User CocLocationsChange ++nested call Coc_qf_jump2loc(g:coc_jump_locations)
augroup END

nmap <silent> gr <Plug>(coc-references)
nnoremap <silent> <leader>qd <Cmd>call Coc_qf_diagnostic()<CR>

function! Coc_qf_diagnostic() abort
    let diagnostic_list = CocAction('diagnosticList')
    let items = []
    let loc_ranges = []
    for d in diagnostic_list
        let type = d.severity[0]
        let text = printf('[%s%s] %s [%s]', (empty(d.source) ? 'coc.nvim' : d.source),
                    \ (d.code ? ' ' . d.code : ''), split(d.message, '\n')[0], type)
        let item = {'filename': d.file, 'lnum': d.lnum, 'col': d.col, 'text': text, 'type': type}
        call add(loc_ranges, d.location.range)
        call add(items, item)
    endfor
    call setqflist([], ' ', {'title': 'CocDiagnosticList', 'items': items,
                \ 'context': {'bqf': {'lsp_ranges_hl': loc_ranges}}})
    botright copen
endfunction

function! Coc_qf_jump2loc(locs) abort
    let loc_ranges = map(deepcopy(a:locs), 'v:val.range')
    call setloclist(0, [], ' ', {'title': 'CocLocationList', 'items': a:locs,
                \ 'context': {'bqf': {'lsp_ranges_hl': loc_ranges}}})
    let winid = getloclist(0, {'winid': 0}).winid
    if winid == 0
        aboveleft lwindow
    else
        call win_gotoid(winid)
    endif
endfunction

Plug 'mhinz/vim-grepper'

augroup Grepper
    autocmd!
    autocmd User Grepper call setqflist([], 'r',
                \ {'context': {'bqf': {'pattern_hl': histget('/')}}}) |
                \ botright copen
augroup END

let g:grepper = {
            \ 'open': 0,
            \ 'quickfix': 1,
            \ 'searchreg': 1,
            \ 'highlight': 0,
            \ }

" try `gsiw` under word
nmap gs  <plug>(GrepperOperator)
xmap gs  <plug>(GrepperOperator)

call plug#end()
```

## Feedback

- If you get an issue or come up with an awesome idea, don't hesitate to open an issue in github.
- If you think this plugin is useful or cool, consider rewarding it a star.

## License

The project is licensed under a BSD-3-clause license. See [LICENSE](./LICENSE) file for details.
