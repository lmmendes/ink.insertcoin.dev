---
layout: post
title:  asdf a runtime manager to rule them all
categories:
    - version manager
    - runtime manager
    - asdf
paginate:
    enabled: true
last_modified_at: "2020-12-01T00:00:00+00:00"
changelog:
  - date: "2020-12-01T00:00:00+00:00"
    message: Fix typos
---
[asdf]: https://github.com/asdf-vm/asdf
[asdf-plugins]: https://asdf-vm.com/#/plugins-all?id=plugin-list
[sdkman]: https://sdkman.io/
[nvm]: https://github.com/nvm-sh/nvm
[rvm]: https://rvm.io/
[rbenv]: https://github.com/rbenv/rbenv
[asdf-plugins]: https://github.com/asdf-vm/asdf-plugins
[tool-versions]: https://asdf-vm.com/#/core-configuration

Currently I'm running multiple versions of multiple runtimes on my machine, eg: OpenJDK 8, 14 then ruby 2.7.0 and NodeJS versions 8 and 12.

Normally for each one of them I would use a specific runtime version manager, for example for Java I would use [sdkman][sdkman], for ruby a couple years ago I replaced [rvm][rvm] with [rbenv][rbenv] and for NodeJS I simply went with [nvm][nvm].

The main pain of using 3 runtime managers is the need to learn their different CLIs and nuances; With [rvm][rvm] and [rbenv][rbenv] you can choose a version of ruby per project by placing in the root of the folder a `.ruby-version` file stating the version to use, [nvm][nvm] also allowed this using `.nvmrc` and so on, but now this is water under the bridge.

<!--more-->

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

## Plugins

Plugins are how adsf understands how to handle different packages, meaning that if you wish to add to adsf the ability to handle java, python, mysql or even minikube you just need to install a plugin.

See the following page to view [all available plugins][asdf-plugins].

### Add

If the plugin exists in the [plugin repository][asdf-plugins] you can just add it by name, otherwise you need to add it providing it's git repository url.

```
# asdf plugin add <name>
asdf plugin add java

# asdf plugin add <name> <git url>
asdf plugin-add kotlin https://github.com/asdf-community/asdf-kotlin.git
```

# List installed

Listing installed plugins

```
asdf plugin list
# java
# maven
# nodejs
# python
# rust
```

Listing installed plugins with their repository url's

```
asdf plugin list --urls
# java    https://github.com/halcyon/asdf-java.git
# maven   https://github.com/halcyon/asdf-maven.git
# nodejs  https://github.com/asdf-vm/asdf-nodejs.git
# python  https://github.com/danhper/asdf-python.git
# rust    https://github.com/code-lever/asdf-rust.git
```

### Updating installed

You can update a specific plugin by name

```
asdf plugin update java
# Updating java...
# remote: Enumerating objects: 108, done.
# remote: Counting objects: 100% (108/108), done.
# remote: Compressing objects: 100% (64/64), done.
# remote: Total 289 (delta 67), reused 82 (delta 44), pack-reused 181
# Receiving objects: 100% (289/289), 192.46 KiB | 1.17 MiB/s, done.
# Resolving deltas: 100% (161/161), completed with 5 local objects.
# From https://github.com/halcyon/asdf-java
#   b050643..eedd04e  master     -> master
#   b050643..eedd04e  master     -> origin/master
# Already on 'master'
# Your branch is up to date with 'origin/master'.
```

Or update all plugins in one go

```
asdf plugin update --all
```

### Remove

```
# asdf plugin remove <name>
asdf plugin remove kotlin
```

## Versions

### List all available versions

```
asdf list all <name>
# asdf list all java
```

### Install

Installing a version is as simple as installing a plugin.

```
asdf install <name> <version>
# asdf install java adopt-openjdk-11.0.6+10_openj9-0.18.1
```

Install the lasted version of a stable version

```
asdf install <name> latest:<version>
asdf install java latest:11
```

### List installed versions

```
asdf list <name>
# asdf list java
```

Filtering displayed versions

```
asdf list all <name> <version>
# asdf list all java openjdk-11
```

### Set current version

You can set a specific plugin version globally, just in the current shell or per folder.

```
asdf global <name> <version>
asdf shell <name> <version>
asdf local <name> <version>
```

| scope | description          |
|-------|----------------------|
| global| writes the version to `$HOME/.tools_versions`, making the version global all shells |
| shell | sets the version just for this shell, but setting `ASDF_${LANG}_VERSION` environment variable |
| local | writes the version to the file `.tools_versions` on the current working directory |

You can read more about the [.tool-versions][tool-versions] file [here][tool-versions].


### View current version

To view whats the current version of the installed plugins

```
asdf current
# java           adopt-openjdk-11.0.6+10_openj9-0.18.1 (set by /Users/lmm/.tool-versions)
# maven          3.5.4    (set by /Users/lmm/.tool-versions)
# nodejs         8.17.0 12.18.2 (set by /lmm/mendelui/.tool-versions)
# python         3.8.4    (set by /Users/lmm/.tool-versions)
# rust           1.45.0   (set by /Users/lmm/.tool-versions)
```

You can also view the version of a specific version

```
asdf current java
# adopt-openjdk-11.0.6+10_openj9-0.18.1 (set by /Users/mendelui/.tool-versions)
```

### Uninstall

```
# asdf uninstall <name> <version>

asdf uninstall rust 1.45.0
```
