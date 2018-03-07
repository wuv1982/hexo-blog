---
title: iOS UI Auto Test integrated with WDA(WebDriverAgent)
lang: ja
date: 2018-03-07 19:30:10
tags: iOS, UITest, XCTest, WebDriverAgent, WDA
---

__XCTest__はiOSのUnitTestをサポートするframeworkです。
iOS 8まではbackgroundでmethodのtestだけだが、iOS 9からUIの自動操作によってtestもできるようになった。
Xcodeに統合されているから_File->New UI Test Case_で簡単に作れるし、且つ動作の録画機能もある。
滅多にTestソース書かずに一回だけ手動でやればscriptが自動生成されるので、
世の中ほとんどのアプリが十分楽にtestできると思う。

例外のはtest target appの画面から一度外に出ることがある場合だ。
そこFacebookの[WebDriverAgent](https://github.com/facebook/WebDriverAgent)を借りることとなった。

## Xcode XCTest
### Unit Test Case
### UI Test Case
![](xcode images)

## WDA
### architecture
### web base api
### life show
### Appnium

## 改造
### native private API
### life show
