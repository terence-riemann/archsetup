To set up Lazyvim (from https://github.com/folke/lazy.nvim), I need Luarocks first. So download that first (in pacman).
```bash
sudo pacman -S luarocks
```
For our VimTeX setup, we want to use Zathura as our pdf reader. Zathura needs a backend plugin to read files like pdfs. So we install the following:
```bash
sudo pacman -S zathura zathura-pdf-mupdf
```
Then afterwards, we follow the installation instructions. The init.lua file should have:
```lua
require("config.lazy")
```
We make a lua/config directory inside the nvim directory and add a lazy.lua file:
```lua
-- Bootstrap lazy.nvim
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({ "git", "clone", "--filter=blob:none", "--branch=stable", lazyrepo, lazypath })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath)

-- Make sure to setup `mapleader` and `maplocalleader` before
-- loading lazy.nvim so that mappings are correct.
-- This is also a good place to setup other settings (vim.opt)
vim.g.mapleader = " "
vim.g.maplocalleader = "\\"

-- Setup lazy.nvim
require("lazy").setup({
  spec = {
    -- import your plugins
    { import = "plugins" },
  },
  -- Configure any other settings here. See the documentation for more details.
  -- colorscheme that will be used when installing plugins.
  install = { colorscheme = { "habamax" } },
  -- automatically check for plugin updates
  checker = { enabled = true },
})
```
Any extra plugins (like VimTeX) will be put in a .lua file in lua/plugins. For VimTeX (https://github.com/lervag/vimtex), we add a plugin spec like this from the github:
```lua
{
  "lervag/vimtex",
  lazy = false,     -- we don't want to lazy load VimTeX
  -- tag = "v2.15", -- uncomment to pin to a specific release
  init = function()
    -- VimTeX configuration goes here, e.g.
    vim.g.vimtex_view_method = "zathura"
  end
}
```
This gives errors as there needs to be a return before the array:
```lua
return {
  "lervag/vimtex",
  lazy = false,     -- we don't want to lazy load VimTeX
  -- tag = "v2.15", -- uncomment to pin to a specific release
  init = function()
    -- VimTeX configuration goes here, e.g.
    vim.g.vimtex_view_method = "zathura"
  end
}
```
We will edit this later to configure it more specifically.

Now we install Latex onto our system. We use texlive:
```bash
sudo pacman -S texlive texlive-latexextra
```
The latex compiler latexmk is inside texlive-binextra which is installed via texlive-latexextra.

Now we edit the vimtex.lua file with more specific configurations:
```lua
return {
  "lervag/vimtex",
  lazy = false,

  init = function()
    -- Use Zathura to view PDFs
    vim.g.vimtex_view_method = "zathura"

    -- Use latexmk to compile documents
    vim.g.vimtex_compiler_method = "latexmk"

    -- Set compiler options
    vim.g.vimtex_compiler_latexmk = {
      options = {
        "-verbose", -- Displays more detailed compilation information.
        "-file-line-error", -- Reports errors with filenames and line numbers.
        "-synctex=1", -- Enables synchronization between your .tex source and PDF.
        "-interaction=nonstopmode", -- Prevents LaTeX from stopping for interactive error prompts.
        "-shell-escape", -- Allows LaTeX to execute external commands.
      },
    }
  end,
}
```
Now we do some useful config of Zathura:
```bash
mkdir -p ~/.config/zathura
nvim ~/.config/zathura/zathurarc
```
We add the following configuration:
```
set selection-clipboard clipboard -- Uses the system clipboard when copying for example. 
set statusbar-h-padding 10 -- Horizontal spacing inside the status bar.
set statusbar-v-padding 5 -- Vertical spacing inside the status bar. 
set page-padding 2 -- Spacing between pages when viewing multiple pages.
set adjust-open width -- Adjusts the page to fit the available window width.
set smooth-scroll true -- Smooth scrolling instead of abrupt scrolling between pages.
set scroll-step 50 -- Speed of scroll.
set statusbar-basename true -- Displays just the PDF filename in the status bar rather than its full path.
```
