color_coded: semantic highlighting with vim
---
color_coded is a vim plugin that provides realtime (fast), tagless code highlighting for C++, C, and Objective C using libclang.

  * Environmentally friendly
  * Lua binding for VimL -> C++
  * Exhaustive customization possibilities
  * Unintrusive highlighting; self-contained
  * Utilizes self-contained clang 3.6.0

**NOTE:** color_coded is currently _alpha_ software; please report issues [here](https://github.com/jeaye/color_coded/issues) if you find them and make some noise on the [clang bugs I've reported](https://github.com/jeaye/color_coded/issues/2).

Before color_coded | After color_coded
:------------------:|:------------------:
<img src="http://jeaye.com/cc/trimmed_cc_t_no_3.png" width="100%"/> | <img src="http://jeaye.com/cc/trimmed_cc_t_yes_3.png" width="100%"/>
<img src="http://jeaye.com/cc/trimmed_cc_t_no_1.png" width="100%"/> | <img src="http://jeaye.com/cc/trimmed_cc_t_yes_1.png" width="100%"/>

Installation
---
#### OS X
Before installing color_coded on OS X, please read the following. The vim that ships with (even the latest) OS X is not new enough to support color_coded. Furthermore, the default macvim from [homebrew](http://brew.sh/) doesn't have lua enabled. To ensure you have a valid macvim install on OS X, please do the following:
```bash
brew update
brew install macvim --with-lua --override-system-vim
brew linkapps macvim
# Spawn a new shell to use the new system vim
```

**NOTE:** color_coded does not currently play nicely with luajit on OS X. For the most stability, please ensure you're using macvim as specified above.

#### Ubuntu/Debian
First, install all required dependencies.
```bash
sudo apt-get install build-essential liblua5.2-dev lua5.2 libncurses-dev libboost-dev libz-dev
```

Next, you need to ensure you have GCC 4.9. If you don't, you can try the following (tested on Ubuntu 14.04).
```bash
# Install GCC 4.9
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install g++-4.9

# Prefer 4.9 to other versions
sudo update-alternatives --remove-all g++
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.9 50
```

#### All
Installation has been tested using [vundle](https://github.com/gmarik/Vundle.vim), but should also be compatible with [pathogen](https://github.com/tpope/vim-pathogen). To install using vundle (add the line to your `~/.vimrc`, restart vim, run `:PluginInstall`):

```viml
Plugin 'jeaye/color_coded'
```

Since color_coded has a compiled component, you'll need to manually compile when installing and updating. Compilation works as follows, assuming usage of vundle (see the [dependencies](https://github.com/jeaye/color_coded#dependencies) section to ensure you can `make` properly):

```bash
cd ~/.vim/bundle/color_coded
./configure && make
```

For various compatibility reasons, color_coded will attempt to download a known version of clang. This may add time to your configuration process, but it offers more stability across multiple platforms.

**NOTE:** color_coded, to my knowledge, has not been tested on Windows.
Usage
---
Once color_coded is installed and compiled, it will automatically begin working the next time you start vim. In order for color_coded to know how your code must be compiled, you _may_ need to create a file describing the required compiler options. Note, color_coded assumes sane defaults and tries to allow for immediate usage (it favors C++ in this regard).

That said, in any non-trivial case, you'll find yourself needing to supply a `.color_coded` file for your project. color_coded will search from the current working directory all the way up to the root of the filesystem looking for a `.color_coded` file. This makes it possible for you to have one in your home directory, for example, and then in each of your projects' directories. If you don't specify one in a project directory, the one in your home directory is used. Again, if no such files are found, sane defaults will be applied.

The `.color_coded` file can be automatically generated using [YCM-Generator](https://github.com/rdnetto/YCM-Generator).

**color_coded will try its hardest to highlight your code for you, even if there are errors in the translation unit.** This allows for better highlighting while typing and highlighting of valid code when color_coded doesn't know everything about your project.

You can test that color_coded is working properly after installation by running `make run`, which will open a small C++ file known to be valid. Highlighting compilation may take a second, but, after moving the cursor, you should see the code becomes highlighted.

### .color_coded file contents
The contents of a `.color_coded` file is simply a line-separated list of compiler options. For example, the `.color_coded` file for color_coded is:

```
-std=c++1y
-Iinclude
```

**NOTE:** Out of simplicity, no other data is permitted in the `.color_coded` file. That is to say, comments are not supported.

### Using for languages other than C++
#### Using for C
Since color_coded defaults to using C++, you'll need to tell it if you're planning on using it for a C project. You can do this by adding these two lines to your `.color_coded` file, along with all your other flags:
```
-x
c
```

Options
---

#### `g:color_coded_enabled`
This option controls whether or not color_coded will function. Invoking the `:CCtoggle` command will modify this option to achieve its results. You may also specify it as `0` in your `~/.vimrc` to globally disable color_coded.

Default:
```viml
let g:color_coded_enabled = 1
```

#### `g:color_coded_filetypes`
This option controls the filetypes which color_coded will try to compile.

Default:
```viml
let g:color_coded_filetypes = ['c', 'h', 'cpp', 'hpp', 'cc', 'm', 'mm']
```

Highlighting
---
There are many new highlighting groups which color_coded adds. They are designed to follow [libclang's internals](http://clang.llvm.org/doxygen/group__CINDEX.html#gaaccc432245b4cd9f2d470913f9ef0013) as closely as possible. To tie things together, some wrapper groups have been created that will allow more sweeping changes. The clang groups, by default, are mapped to use these, along with vim's normal groups (`Function`, `Macro`, `Number`, etc).

#### Generic groups
```viml
" To get full highlighting, specify these in your colorscheme.
" See colors/twilighted.vim for example usage

hi Member " Any non-static member variable
hi Variable " Any non-member variable
hi Namespace
hi EnumConstant
```

#### Example of clang groups
```viml
hi link StructDecl Type
hi link UnionDecl Type
hi link ClassDecl Type
hi link EnumDecl Type
```

#### Sample colorscheme
color_coded ships with a sample color scheme demonstrating support for the extended groups. To use it, invoke `:colorscheme twilighted`. As the name may suggest, the colorscheme is forked from vim's own twilight theme and has been appropriately punned into that which you'll see in the repo. To examine the implementation, visit `colors/twilighted.vim`. Note that color_coded can work with any colorscheme; this is just a sample.

For more information on all of the supported groups, see `after/syntax/color_coded.vim` and the [clang documentation](http://clang.llvm.org/doxygen/group__CINDEX.html#gaaccc432245b4cd9f2d470913f9ef0013).

Commands
---

#### `:CCerror`
This command outputs the last compilation error message from libclang. If your highlighting is not working properly, you may have a misconfigured `.color_coded` file or you may have syntax errors in your source. When in doubt, check here first.

#### `:CCtoggle`
This command enables color_coded, if it's currently disabled, or disables it, if it's currently enabled. This enable/disabling is done via the `g:color_coded_enabled` option.

Dependencies
---
Compilation of color_coded requires:
  * GCC ≥ 4.9 OR Clang ≥ 3.4
  * Lua ≥ 5.2.x

Usage of color_coded requires vim:
  * Version: 7.4p330+
  * Compiled with Lua support (+lua)

Troubleshooting/FAQ
---

#### As I type, the highlighting becomes messed up
color_coded tries to compile your coded as you type it. Due to how vim works, color_coded can only update the highlighting once you do something (move the cursor, make a change, etc). When you're done typing and the highlighting has not finished updating, assuming your `.color_coded` file is sufficient, moving the cursor (or causing events some other way) will allow color_coded to finish.

Note, there is a hold event in vim which triggers *after* you stop typing and *after* some delay. color_coded also hooks into this and will use it to apply highlighting if possible.

#### Some bits aren't highlighted or are highlighted incorrectly
Yup. Believe it or not, these are almost certainly libclang bugs. I've been sorting out a few of them and I'm maintaining my own fork of libclang. If you would like to report such an issue, check out [this ticket](https://github.com/jeaye/color_coded/issues/2).

#### The highlighting isn't refreshed in a new buffer until I move the cursor
This is intentional. The first time you open a buffer, color_coded doesn't know if it's going to compile properly and it doesn't want you to wait while it tries to figure this out. color_coded will always compile in the background and events like moving the cursor or changing text will poll for updates. **Note, however,** that, once a buffer has highlighting, leaving that buffer and coming back to it will synchronously apply the previous highlighting.

#### "color_coded unavailable: you need to compile it"
See the above installation docs. When you install color_coded, you need to manually `./configure && make` (any errors will be reported) before you can successfully use it.

#### "color_coded has been updated: you need to recompile it"
Assuming you've updated a working installation of color_coded, you'll get this error if the update requires you to recompile color_coded (i.e. there have been changes to the native API). To recompile, follow the same exact steps you took to compile initially. Generally, this just means `./configure && make`.

#### "xz is required to unpack clang"
As of clang 5.0, both Linux and OS X tarballs are compressed with [xz](http://tukaani.org/xz/). To install, consider one of the following (or further documentation for your OS/distribution):
###### OS X
```bash
brew install xz # for homebrew
port install xz # for macports
```
###### Ubuntu/Debian
```bash
sudo apt-get install xz-utils
```

#### Does color_coded work with neovim?
I doubt it. color_coded has not been tested on neovim and there has been significant work to the plugin API and the lua interface. For now, at least, color_coded is not supporting neovim.

License
---
color_coded is under the MIT open-source license.  
See the `LICENSE` file or http://opensource.org/licenses/MIT
