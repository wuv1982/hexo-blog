---
title: install-tips1
language: ja
date: 2018-05-31 12:07:26
tags: tips
---

## nodeのバージョン切り替え
homebrewを使って古いバージョンのnodeをinstalls
```
brew install node@8
brew unlink node
brew link node@8
```

## setup java_home
1. download openJDK & unzip to favorite path
2. `cd /Library/Java/JavaVirtualMachines`
3. `ln -s /my/path/to/openJDK jdk1.x.x_xxx`
4. check with `/usr/libexec/java_home`


## fastlane
1. `sudo gem install fastlane -NV`
2. `sudo fastlane init`
3. `fastlane add_plugin google_cloud_storage`
4. need `pod repo add MyRepo url.myrepo.git` to use `pod_push` action

## zsh
1. `git clone <oh-my-zsh>`
2. read `oh-my-zsh/tool/install.sh`
3. `cp oh-my-zsh/template/xx-template ~/.zshrc`
4. `vi .zshrc` > `ZSH=<oh-my-zsh>` `ZSH_THEME`
5. download `powerline` font
6. change terminal font setting
