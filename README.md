# Fisher

[![Build Status](https://img.shields.io/travis/jorgebucaran/fisher.svg)](https://travis-ci.org/jorgebucaran/fisher)
[![Releases](https://img.shields.io/github/release/jorgebucaran/fisher.svg?label=latest)](https://github.com/jorgebucaran/fisher/releases)

Fisher is a package manager for the [fish shell](https://fishshell.com). It defines a common interface for package authors to build and distribute shell scripts in a portable way. You can use it to extend your shell capabilities, change the look of your prompt and create repeatable configurations across different systems effortlessly.

## Features

- Zero configuration
- Oh My Fish package support
- High-speed concurrent package downloads⌁!
- If you've installed a package before, you can install it again offline
- Add, update and remove functions, completions, key bindings and configuration snippets from a variety of sources using the command line, editing your [fishfile](#using-the-fishfile) or both

## Installation

Download `fisher` to your fish functions directory or any directory in the fish function path.

```sh
curl https://git.io/fisher --create-dirs -sLo ~/.config/fish/functions/fisher.fish
```

Your shell can take a few seconds before refreshing the function path. If `fisher` is not immediately available after the download is complete, you can launch a new session, or [replace the current session](https://fishshell.com/docs/current/commands.html#exec) with a new one.

> **Note:** If the [`XDG_CONFIG_HOME`](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html#variables) environment variable is defined on your system, use `$XDG_CONFIG_HOME/fish` to find the path to your fish configuration directory.

### Dependencies

- [fish](https://github.com/fish-shell/fish-shell) 2.2+
- [curl](https://github.com/curl/curl) 7.10.3+
- [git](https://github.com/git/git) 1.7.12+

Stuck in fish 2.0 and can't upgrade your shell? Check our [legacy fish support guide](https://github.com/jorgebucaran/fisher/issues/510) and good luck!

### Bootstrap installation

To automate installing Fisher and the packages listed in your [fishfile](#using-the-fishfile) on a new system, add the following code to your fish configuration file.

```fish
if not functions -q fisher
    set -q XDG_CONFIG_HOME; or set XDG_CONFIG_HOME ~/.config
    curl https://git.io/fisher --create-dirs -sLo $XDG_CONFIG_HOME/fish/functions/fisher.fish
    fish -c fisher
end
```

### Changing the installation prefix

Use the `$fisher_path` environment variable to change the location where functions, completions, and [configuration snippets](#configuration-snippets) will be copied to when a package is installed. The default location will be your fish configuration directory.

```fish
set -g fisher_path /path/to/another/location

set fish_function_path $fish_function_path[1] $fisher_path/functions $fish_function_path[2..-1]
set fish_complete_path $fish_complete_path[1] $fisher_path/completions $fish_complete_path[2..-1]

for file in $fisher_path/conf.d/*.fish
    builtin source $file 2> /dev/null
end
```

Do I need this? It depends. If you want to keep your own functions, completions, and configuration snippets separate from packages installed with Fisher, you can customize the installation prefix. If you prefer to keep everything in the same place, you can skip this.

## Usage

You've found an interesting utility you'd like to try out. Or perhaps you've [created a package](#creating-your-own-package) yourself. How do you install it on your system? Now you want to update or remove it. How do you do that?

You can use Fisher to add, update, and remove packages interactively, taking advantage of fish [tab completion](https://fishshell.com/docs/current/index.html#completion) and syntax highlighting. Or [edit your fishfile](#using-the-fishfile) and commit your changes. Do you prefer a CLI-centered approach, text-based approach, or a mix of both?

### Adding packages

Add packages using the `add` command followed by the path to the repository on GitHub.

```
fisher add jethrokuan/z rafaelrinaldi/pure
```

To add a package from anywhere other than GitHub, use the address of the server and the path to the repository. HTTPS is always assumed so you don't need to specify the protocol.

```
fisher add gitlab.com/jorgebucaran/mermaid
```

To add a package from a private repository set the `fisher_user_api_token` variable to your username followed by a colon and your authorization token or password.

```fish
set -g fisher_user_api_token jorgebucaran:ce04da9bd93ddb5e729cfff4a58c226322c8d142
```

For a specific version of a package add an `@` symbol after the package name followed by the tag, branch or [commit-ish](https://git-scm.com/docs/gitglossary#gitglossary-aiddefcommit-ishacommit-ishalsocommittish) you want. Only one package version can be installed at any given time.

```
fisher add edc/bass@20f73ef jethrokuan/z@pre27
```

You can add packages from a local directory too. Local packages will be copied as [symbolic links](https://en.wikipedia.org/wiki/Symbolic_link) so changes in the original files will be reflected in future shell sessions without having to run `fisher` again.

```
fisher add ~/path/to/local/pkg
```

### Listing packages

List all the packages that are currently installed using the `ls` command. This shows only the packages that you've installed yourself, ignoring their dependencies.

```
fisher ls
jethrokuan/z
rafaelrinaldi/pure
gitlab.com/jorgebucaran/mermaid
edc/bass
~/path/to/myfish/pkg
```

Is the output too large? This command accepts a regular expression to refine the listed packages.

```
fisher ls "^gitlab|fish-.*"
```

### Removing packages

Remove packages using the `rm` command. If a package has dependencies, they too will be removed. If any dependencies are still shared by other packages, they will remain installed.

```
fisher rm rafaelrinaldi/pure
```

You can remove everything that is currently installed in one sweep using the following pipeline.

```sh
fisher ls | fisher rm
```

### Updating packages

Run `fisher` to update everything you've installed. There is no dedicated update command. Using the command line to add and remove packages is a facade for modifying and committing changes to your fishfile in a single step.

Looking for a way to update fisher itself? Use the `self-update` command.

```
fisher self-update
```

### Other commands

Use the `help` command to display usage help on the command line.

```
fisher help
```

Last but not least, use the `version` command to display the current version of fisher.

```
fisher version
```

### Using the fishfile

Whenever you add or remove a package from the command line we'll write to a text file in `~/.config/fish/fishfile`. This is your fishfile. It lists every package that is currently installed on your system. You should add this file to your dotfiles or version control if you want to reproduce your configuration on a different system.

You can edit this file to add or remove packages and run `fisher` to commit your changes. Only the packages listed in this file will be installed after `fisher` returns. If a package is already installed, it will be updated. Everything after a `#` symbol (comments) will be ignored.

```fish
vi ~/.config/fish/fishfile
```

```diff
- rafaelrinaldi/pure
- jethrokuan/z@pre27
gitlab.com/jorgebucaran/mermaid
edc/bass
+ FabioAntunes/fish-nvm
~/path/to/myfish/pkg
```

```
fisher
```

That will remove **rafaelrinaldi/pure** and **jethrokuan/z**, add **FabioAntunes/fish-nvm** and update the rest.

## Packages

Packages help you organize shell scripts into reusable, independent components that can be shared through a git URL or the path to a local directory. Even if your package is not meant to be shared with others, you can benefit from composition and the ability to depend on other packages.

The structure of a package can be adopted from the fictional project described below. These are the files that fisher looks for when installing or uninstalling a package. The name of the root directory can be anything you like.

```
fish-kraken
├── fishfile
├── functions
│   └── kraken.fish
├── completions
│   └── kraken.fish
└── conf.d
    └── kraken.fish
```

If your project depends on other packages, it should list them as dependencies in a fishfile. There is no need for a fishfile otherwise. The rules concerning the usage of the fishfile are the same rules we've already covered in [using the fishfile](#using-the-fishfile).

While some packages contain every kind of file, some packages contain only functions or configuration snippets. You are not limited to a single file per directory either. There can be as many files as you need or only one as in the next example.

```
fish-kraken
└── kraken.fish
```

The lack of private function scope in fish causes all package functions to share the same namespace. A good rule of thumb is to prefix functions intended for private use with the name of your package to reduce the possibility of conflicts.

### Creating your own package

The best way to show you how to create your own package is by building one together. Our first example will be a function that prints the raw non-rendered markdown source of a README file from GitHub to standard output. Its inputs will be the name of the owner, repository, and branch. If no branch is specified, we'll use the master branch.

Create the following directory structure and function file. Make sure the function name matches the file name, otherwise fish won't be able to autoload it the first time you try to use it.

```
fish-readme
└── readme.fish
```

```fish
function readme -a owner repo branch
    if test -z "$branch"
        set branch master
    end
    curl -s https://raw.githubusercontent.com/$owner/$repo/$branch/README.md
end
```

You can install it with the `add` command followed by the path to the directory.

```
fisher add /absolute/path/to/fish-readme
```

The next logical step is to share it with others. How do you do that? Fisher is not a package registry. Its function is to fetch fish scripts and put them in place so that your shell can find them. To publish a package put your code online. You can use GitHub, GitLab or BitBucket or anywhere you like.

Now let's install the package again, this time from its new location. Open your [fishfile](#using-the-fishfile) and replace the local version of the package we previously added with the URL of the remote repository. Save your changes and run `fisher`.

```diff
- /absolute/path/to/fish-readme
+ jorgebucaran/fish-readme
```

```
fisher
```

You can leave off the github.com part of the URL when adding or removing packages hosted on GitHub. If your package is hosted anywhere else, the address of the server is required.

### Configuration snippets

Configuration snippets consist of all the fish files inside your `~/.config/fish/conf.d` directory. They are evaluated on [shell startup](http://fishshell.com/docs/current/index.html#initialization) and generally used to set environment variables, add new key bindings, etc.

Unlike functions or completions which can be erased programmatically, we can't undo a fish file that has been sourced without creating a new shell session. For this reason, packages that use configuration snippets provide custom uninstall logic through an uninstall [event handler](https://fishshell.com/docs/current/#event).

Let's walk through an example that uses this feature to add a new key binding for the Control-G sequence that opens your fishfile in the `vi` editor. When you install the package, `fishfile_quick_edit_key_bindings.fish` will be sourced, adding the specified key binding and loading the event handler function. When you uninstall it, we'll emit an uninstall event where the key binding will be erased.

```
fish-fishfile-quick-edit
└── conf.d
    └── fishfile_quick_edit_key_bindings.fish
```

```fish
bind \cg "vi ~/.config/fish/fishfile"

set -l name (basename (status -f) .fish){_uninstall}

function $name --event $name
    bind -e \cg
end
```

## Uninstalling

You want to know how to remove this package manager and everything you've installed with it from your system. This command will purge all packages, cache, configuration and related files. We'll miss you.

```fish
fisher self-uninstall
```

## License

Copyright © 2016-2019 Jorge Bucaran <<https://jorgebucaran.com>>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
