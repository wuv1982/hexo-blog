---
title: CocoaPods Private Pods
date: 2018-03-23 18:53:59
tags: [iOS, CocoaPods]
---
## Public Spec Repo
CocoaPodsにはMaven Centerみたいなlibraryを集約して保存する場所がないが、
[`Specs Master repo`](https://github.com/CocoaPods/Specs)というlibraryリストだけを管理する場所がある。
普通にCocoaPods使う時、`pod 'Alamofire'`のようにlibrary名だけを書けば、
libraryが特定されダウンロードされるのはSpecs Masterに登録されているからだ。
{% asset_img img-0101-masterrepo.png "CocoaPods Specs Master" %}

実際Specs Masterはただの普通のGithub repositoryだ。
CocoaPods使う度にlocal環境(`~/.cocoapods/repos/master`)にclone又はpullされる。
誰でも自分のlibraryを公開できるので、リストが増え続いてる。
結果大きすぎて更新するには時間結構かかる、CocoaPods反応遅いという指摘は大体ここに当る。

> __自分のlibraryをCocoaPodsとして公開すること__ == __Specs Masterに自分のpodspecをpushする__


## Private Spec Repo
[公式Document](https://guides.cocoapods.org/making/private-cocoapods.html)
速度、レビュー待ち、管理権限などの心配があれば、自分だけのPrivate Spec Repoでも作ることができる。
仕組みを理解すれば、documentにある`pod repo`のcommandを覚えなくていい。
以下の流れに従えば同じことができる。

0. git repoを新規

  普通のrepoを新規作ると同じ
```sh
cd MyRepo
git init
```

0. folder tree 構成

  このような階層構成にrepoに追加
```
MyRepo
├── Specs (省略できる)
    └── MyPod
        └── 1.1.1
            └── MyPod.podspec
```

0. podspec定義作成

  [document](https://guides.cocoapods.org/syntax/podspec.html)のようにpodの詳細を記載するpodspec fileを作成する。

0. lintでチェック

  podspec fileがあるpathにlintを実行し、定義が正しいことをcheck
```
cd MyRepo/Specs/MyPod/1.1.1
pod spec lint
```

0. GithubにPush

0. Podfileで運用

  app側のPodfileにPrivate Spec repo所在のGithub linkを最初に追加
```ruby
source 'https://github.com/my/private/repo'

platform :ios, '9.0'
inhibit_all_warnings!

target 'MyApp' do
  pod 'MyPod', '~> 1.1'
end
```


## Private Spec Repo更新する時の問題
MyPodを新しいversionを配布する時、新しいcommitをremoteのmasterにpushしないとtestで`pod update`実行することができない。
しかしtestする前remote/masterにpushするのは筋じゃない。

ではlocalだけで新しいversionの`pod update`のtest出来るか？

まず、`~/.cocoapods/repos`の下に行ってみる。

元々`master`というrepo一つしかなかったが、Private Spec Repoに使われたGithub accountと同じ名前のfolderが増えた。
中にPrivate Spec Repoがcloneされてる。

default branchは`master`なので、ここで新しいbranchを切って新versionを作る。commitしていいがpushはしない。

今度test appに戻って`pod update`てtestすると、errorが発生、remoteに修正中のbranchがないと怒られる。
[pod command reference](https://guides.cocoapods.org/terminal/commands.html)によると`--no-repo-update`がrepoの更新を抜くことができるのでこれを使う。

```sh
pod update --no-repo-update
```
これで安心でrelease出来る。

## Private Library Repoの場合
Podの元となるLibraryがGithub Private Project場合、binary libraryとして公開することができる。
[PodSpec Syntax `Source`](https://guides.cocoapods.org/syntax/podspec.html#source)を参考に。
```
spec.source = { :http => 'http://dev.wechatapp.com/download/sdk/WeChat_SDK_iOS_en.zip' }
```
