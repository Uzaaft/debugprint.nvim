# debugprint.nvim

## Overview

`debugprint` is a plugin for NeoVim that simplifies debugging for those who
prefer a low-tech approach. While using a real debugger like
[nvim-dap](https://github.com/mfussenegger/nvim-dap) is the gold standard for
debugging a script or program, some prefer the use of the 'print' statement to
trace the output during execution. `debugprint` allows quick insertion of appropriate 'print'
statements based on the language being edited.

debugprint also inserts into the 'print' statements the file name and line
numbers where debug lines are inserted, as well as a snippet of the previous or
following line and a unique counter value from the current NeoVim session to
help identify lines in debug output. Optionally it can also output variable
values.

`debugprint` supports the following filetypes/programming languages out-of-the-box:

*   `bash`
*   `c`
*   `cpp` (C++)
*   `cs` (C#)
*   `dart`
*   `dockerfile`
*   `go`
*   `java`
*   `javascript`
*   `lua`
*   `make`
*   `php`
*   `python`
*   `ruby`
*   `rust`
*   `sh` (Sh/Bash)
*   `typescript`
*   `vim`
*   `zsh`

It can also be extended to support more.

`debugprint` is inspired by
[vim-debugstring](https://github.com/bergercookie/vim-debugstring), which I've
used for several years, but is updated and refreshed for the NeoVim generation.
It provides various improvements:

*   Its configuration system is more 'NeoVim-like' and it is easier to add custom
    languages in your configuration.

*   It [dot-repeats](https://jovicailic.org/2018/03/vim-the-dot-command/) with NeoVim.

*   It can pick up a variable name from under the cursor.

*   It provides keymappings for visual mode, so you can select a variable
    visually and print it out.

*   It provides keymappings for operator-pending mode, so you can select a
    variable using a motion.

*   It indents the lines it inserts more accurately.

*   The output when printing a 'plain' debug line, or a variable, is more
    consistent.

*   It provides a command to delete all debugging lines added to the current buffer.

*   Able to optionally move to the inserted line (or not).

## Demo

<div align="center">
  <video src="https://user-images.githubusercontent.com/107015/211196425-2b371442-54f9-43d7-9187-a29185d43586.mp4" type="video/mp4"></video>
</div>

## Installation

**Requires NeoVim 0.8+.**

Optional dependency for NeoVim 0.8 only:
[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter). If this
is not installed, `debugprint` will not find variable names under the cursor and
will always prompt for a variable name. For NeoVim 0.9+, this dependency is
never needed.

Example for [`lazy.nvim`](https://github.com/folke/lazy.nvim):

```lua
return {
    url = "andrewferrier/debugprint.nvim",
    opts = { ... },
    -- Dependency only needed for NeoVim 0.8
    dependencies = {
        "nvim-treesitter/nvim-treesitter"
    },
    -- Remove the following line to use development versions,
    -- not just the formal releases
    version = "*"
}
```

Example for [`packer.nvim`](https://github.com/wbthomason/packer.nvim):

```lua
packer.startup(function(use)

    ...

    use({
        "andrewferrier/debugprint.nvim",
        config = function()
            opts = { ... }
            require("debugprint").setup(opts)
        end,
    })

    ...

end)
```

The sections below detail the allowed options that can appear in the `opts`
object.

Please subscribe to [this GitHub
issue](https://github.com/andrewferrier/debugprint.nvim/issues/25) to be
notified of any breaking changes to `debugprint`.

## Keymappings and Commands

By default, the plugin will create some keymappings and commands, which are the
standard way to use it. There are also some function invocations which are not
mapped to any keymappings or commands by default, but could be. This is all
shown in the following table.

| Mode             | Default Keymap/Command | Purpose                                                                                                                           | Equivalent Lua Function                                                                       |
| ---------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Normal           | `g?p`                  | Insert a 'plain' debug line appropriate to the filetype just below the current line                                               | `require('debugprint').debugprint()`                                                          |
| Normal           | `g?P`                  | The same, but above the current line                                                                                              | `require('debugprint').debugprint({above = true})`                                            |
| Normal           | `g?v`                  | Insert a variable debugging line below the current line. If the cursor is on a variable name, use that, otherwise prompt for one. | `require('debugprint').debugprint({variable = true})`                                         |
| Normal           | `g?V`                  | The same, but above the current line                                                                                              | `require('debugprint').debugprint({above = true, variable = true})`                           |
| Normal           | None by default        | Always prompt for a variable name, and insert a debugging line just below the current line which outputs it                       | `require('debugprint').debugprint({ignore_treesitter = true, variable = true})`               |
| Normal           | None by default        | Always prompt for a variable name, and insert a debugging line just above the current line which outputs it                       | `require('debugprint').debugprint({ignore_treesitter = true, above = true, variable = true})` |
| Visual           | `g?v`                  | Find the visually select variable name, and insert a debugging line just below the current line which outputs it                  | `require('debugprint').debugprint({variable = true})`                                         |
| Visual           | `g?v`                  | Find the visually select variable name, and insert a debugging line just below the current line which outputs it                  | `require('debugprint').debugprint({variable = true})`                                         |
| Operator-pending | `g?o`                  | Locate a variable using a motion, and insert a debugging line just above the current line which outputs it                        | `require('debugprint').debugprint({motion = true})`                                           |
| Operator-pending | `g?O`                  | Locate a variable using a motion, and insert a debugging line just above the current line which outputs it                        | `require('debugprint').debugprint({motion = true, above = true})`                             |
| Command          | `:DeleteDebugPrints`   | Delete all debug lines added to this buffer.                                                                                      | `require('debugprint').deleteprints()`                                                        |

The keymappings are chosen specifically because by default in NeoVim they are
used to convert sections to ROT-13, which most folks don't use. You can disable
the defaults above from being created by setting `create_keymaps` and/or
`create_commands`, and map them yourself to something else if you prefer:

```lua
opts = {
    create_keymaps = false,
    create_commands = false
    ...
}

require("debugprint").setup(opts)

vim.keymap.set("n", "<Leader>d", function()
    -- Note: setting `expr=true` and returning the value are essential
    return require('debugprint').debugprint()
end, {
    expr = true,
})
vim.keymap.set("n", "<Leader>D", function()
    -- Note: setting `expr=true` and returning the value are essential
    return require('debugprint').debugprint({ above = true })
end, {
    expr = true,
})
vim.keymap.set("n", "<Leader>dq", function()
    -- Note: setting `expr=true` and returning the value are essential
    return require('debugprint').debugprint({ variable = true })
end, {
    expr = true,
})
vim.keymap.set("n", "<Leader>Dq", function()
    -- Note: setting `expr=true` and returning the value are essential
    return require('debugprint').debugprint({ above = true, variable = true })
end, {
    expr = true,
})
vim.keymap.set("n", "<Leader>do", function()
    -- Note: setting `expr=true` and returning the value are essential
    -- It's also important to use motion = true for operator-pending motions
    return require('debugprint').debugprint({ motion = true })
end, {
    expr = true,
})

vim.api.nvim_create_user_command("DeleteDebugs", function(opts)
    -- Note: you must set `range=true` and pass through opts for ranges to work
    M.deleteprints(opts)
end, {
    range = true})
end)
...
```

or, to have a keymapping instead for deleting debug lines (this will only affect
the entire buffer, visual and operator-pending modes will not work):

```lua
vim.keymap.set("n", "g?d", function()
    M.deleteprints()
end)
```

## Other Options

`debugprint` supports the following options in its global `opts` object:

| Option              | Default      | Purpose                                                                                                                                      |
| ------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `create_keymaps`    | `true`       | Creates default keymappings - see above                                                                                                      |
| `move_to_debugline` | `false`      | When adding a debug line, moves the cursor to that line                                                                                      |
| `display_counter`   | `true`       | Whether to display/include the monotonically increasing counter in each debug message added                                                  |
| `display_snippet`   | `true`       | Whether to include a snippet of the line above/below in plain debug lines                                                                    |
| `filetypes`         | See below    | Custom filetypes - see below                                                                                                                 |
| `ignore_treesitter` | `false`      | Never use treesitter to find a variable under the cursor, always prompt for it - overrides the same setting on `debugprint()` if set to true |
| `print_tag`         | `DEBUGPRINT` | The string inserted into each print statement, which can be used to uniquely identify statements inserted by `debugprint`.                   |

## Add Custom Filetypes

*Note: Since `debugprint.nvim` is still relatively new, if you work out a
configuration for a filetype not supported out-of-the-box, it would be really
appreciated if you can open an
[issue](https://github.com/andrewferrier/debugprint.nvim/issues/new) to have it
supported out-of-the-box in `debugprint` so others can benefit from it.
Similarly, if you spot any issues with, or improvements to, the language
configurations out-of-the-box, please open an issue also.*

If `debugprint` doesn't support your filetype, you can add it as a custom
filetype in one of two ways:

*   In the `opts.filetypes` object in `setup()`.

*   Using the `require('debugprint').add_custom_filetypes()` method (designed for
    use from `ftplugin/` directories, etc.

In either case, the format is the same. For example, if adding via `setup()`:

```lua
local my_fileformat = {
    left = 'print "',
    right = '"',
    mid_var = "${",
    right_var = '}"',
}

require('debugprint').setup({ filetypes = { my_fileformat, another_of_my_fileformats, ... }})
```

or `add_custom_filetypes()`:

```lua
require('debugprint').add_custom_filetypes({ my_fileformat, ... })
```

Your new file format will be *merged* in with those that already exist. If you
pass in one that already exists, your configuration will override the built-in
configuration.

The keys in the configuration are used like this:

| Type of debug line  | Default keys            | How debug line is constructed                                                                                               |
| ------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Plain debug line    | `g?p`/`g?P`             | `my_fileformat.left .. "auto-gen DEBUG string" .. my_fileformat.right`                                                      |
| Variable debug line | `g?v`/`g?V`/`g?o`/`g?O` | `my_fileformat.left .. "auto-gen DEBUG string, variable=" .. my_file_format.mid_var .. variable .. my_fileformat.right_var` |

If it helps to understand these, you can look at the built-in configurations in
[filetypes.lua](lua/debugprint/filetypes.lua).

## Known Limitations

*   `debugprint` only supports variable names or simple expressions when using
    `g?v`/`g?V` - in particular, it does not make any attempt to escape
    expressions, and may generate invalid syntax if you try to be too clever.
    There's [an issue to look at ways of improving
    this](https://github.com/andrewferrier/debugprint.nvim/issues/20).

## Alternative Feature Comparison

(This table is quite wide, you may need to scroll horizontally)

| Feature                                                             | `debugprint.nvim` | [vim-debugstring](https://github.com/bergercookie/vim-debugstring) | [printer.nvim](https://github.com/rareitems/printer.nvim) | [refactoring.nvim](https://github.com/ThePrimeagen/refactoring.nvim) | [vim-printer](https://github.com/meain/vim-printer) | [vim-printf](https://github.com/mptre/vim-printf) | [logsitter](https://github.com/gaelph/logsitter.nvim) |
| ------------------------------------------------------------------- | ----------------- | ------------------------------------------------------------------ | --------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------- |
| Print plain debug lines                                             | :+1:              | :+1:                                                               | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Print plain debug lines using treesitter                            | :x:               | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :+1:                                                  |
| Print variables using current word/heuristic                        | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :+1:                                                | :+1:                                              | :x:                                                   |
| Print variables using treesitter                                    | :+1:              | :x:                                                                | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables/expressions using prompts                           | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables using motions                                       | :+1:              | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Print variables using visual mode                                   | :+1:              | :x:                                                                | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| Print debug lines above/below current line                          | :+1:              | :x:                                                                | (only via global config)                                  | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Supports [dot-repeat](https://www.vikasraj.dev/blog/vim-dot-repeat) | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Can control whether to move to inserted lines                       | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Command to clean up all debug lines                                 | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| *Built-in support for:*                                             | -                 | -                                                                  | -                                                         | -                                                                    | -                                                   | -                                                 | -                                                     |
| arduino                                                             | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| bash/sh                                                             | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| C                                                                   | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| C#                                                                  | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| C++                                                                 | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| CMake                                                               | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| dart                                                                | :+1:              | :x:                                                                | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Docker                                                              | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| fish                                                                | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Fortran                                                             | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Golang                                                              | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :+1:                                                  |
| Haskell                                                             | :x:               | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Java                                                                | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| Javascript/Typescript                                               | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :+1:                                                  |
| lua                                                                 | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :+1:                                                  |
| GNU Make                                                            | :+1:              | :+1:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Perl                                                                | :x:               | :x:                                                               | :x:                                                       | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| PHP                                                                 | :+1:              | :+1:                                                               | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Python                                                              | :+1:              | :+1:                                                               | :+1:                                                      | :+1:                                                                 | :+1:                                                | :x:                                               | :x:                                                   |
| Ruby                                                                | :+1:              | :+1:                                                               | :x:                                                       | :+1:                                                                 | :x:                                                 | :x:                                               | :x:                                                   |
| Rust                                                                | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| VimL                                                                | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| zsh                                                                 | :+1:              | :+1:                                                               | :+1:                                                      | :x:                                                                  | :+1:                                                | :x:                                               | :x:                                                   |
| Add custom filetypes (doced/supported)                              | :+1:              | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :+1:                                              | :+1:                                                  |
| Customizable callback formatter                                     | :x:               | :x:                                                                | :+1:                                                      | :x:                                                                  | :x:                                                 | :x:                                               | :x:                                                   |
| Implemented in                                                      | Lua               | VimL                                                               | Lua                                                       | Lua                                                                  | VimL                                                | VimL                                              | Lua                                                   |
