---
title: github accountの管理
date: 2018-03-22 15:08:04
tags: [git, github]
---
## SSH private key
[githubのhelp](https://help.github.com/enterprise/2.12/user/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

```sh
# key作成
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# ssh-agentに追加
ssh-add ~/.ssh/id_rsa_yours
```

## Access Token
Access TokenでHTTPS接続する。

あるprivate goプロジェクトをcloneする時下記で落とした。
```sh
git clone ${AccessToken}:x-oauth-basic@github.com/my/repo
```

しかし`dep ensure`の時に認証通らなかった。原因は他のprivate repoも参照されてたが、Access Tokenを指定できる場所がない。
それでglobal設定を変更しなきゃならない。

global configを変更
```sh
# It will rewrite all url of "https://github.com" with "${AccessToken}:x-oauth-basic@github.com/" when cloning dependencies.
git config --global url."${AccessToken}:x-oauth-basic@github.com/".insteadOf "https://github.com/"
```
