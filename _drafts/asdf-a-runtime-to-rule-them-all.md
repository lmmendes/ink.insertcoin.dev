---
layout: post
title:  asdf a runtime manager to rule them all
categories:
    - java
    - spring
paginate:
    enabled: true
---
[asdf]: https://github.com/asdf-vm/asdf
[asdf-plugins]: https://asdf-vm.com/#/plugins-all?id=plugin-list
[sdkman]: https://sdkman.io/
[nvm]: https://github.com/nvm-sh/nvm
[rvm]: https://rvm.io/
[rbenv]: https://github.com/rbenv/rbenv

Currently I'm running multiple versions of multiple runtimes on my machine, eg: OpenJDK 8, 14 then ruby 2.7.0 and nodejs verions 8 and 12.

Normaly for each one of them I would use a specific runtime version manager, for example for Java I would use [sdkman][sdkman], for ruby a couple years ago I replaced [rvm][rvm] with [rbenv][rbenv] and for NodeJS I simply went with [nvm][nvm].

The main pain of using 3 runtime managers is the need to learn their diferent CLIs and nuances; With [rvm][rvm] and [rbenv][rbenv] you can choose a version of ruby per project by placing in the root of the folder a `.ruby-version` file stating the version to use, [nvm][nvm] also allowed this using `.nvmrc` and so on, but now this is water under the bridge.

## Installing asdf-vm

Simply clone the **asdf** repository to your home has shown bellow

```bash
# Clone the latest branch
git clone https://github.com/asdf-vm/asdf.git ~/.asdf
cd ~/.asdf
git checkout "$(git describe --abbrev=0 --tags)"
```

### Shell integration

On a MacOS or *nix using `bash` edit your `~/.bash_profile`

```bash
. $HOME/.asdf/asdf.sh
. $HOME/.asdf/completions/asdf.bash
```

If you are using `zshell` edit `~/.zshrc` and add the following to the end of the file

```bash
# append completions to fpath
fpath=(${ASDF_DIR}/completions $fpath)
# initialise completions with ZSH's compinit
autoload -Uz compinit
compinit
```

### Update

```
asdf update
```

## Installing plugins

To install plugins
