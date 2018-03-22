---
title: Xcodeとの日常(1)
date: 2018-03-22 14:49:48
tags: [iOS, Xcode]
---

## Carthage build Error
先日Carthage(0.28.0) build した際に `building for tvOS, but linking in object file built for iOS`が出てきて、compile errorとなった。
しかしXcodeで同じschemeでは再現しない。
次の日にCarthageをupgradeしたら(0.29.0)問題なくなった。

__libraryのbuild platform検出__
```sh
otool -l library.a | grep LC_VERSION
```
