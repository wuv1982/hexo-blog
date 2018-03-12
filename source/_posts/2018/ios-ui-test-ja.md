---
title: iOS UI Test integrates with WDA(WebDriverAgent)
lang: ja
date: 2018-03-07 19:30:10
tags: iOS, Test, UITest, XCTest, WebDriverAgent, WDA
---

## XCTestの歴史
XcodeはUnit Testをサポートするため`XCTest`というFrameworkを提供してる。
余計なことを言うが、iPhoneに存在するiOS Frameworkではないので、Test BuildとTest時に使われるのFrameworkだ。

* XCTest

  Xcode5、6までのbackground method用のFramework

* UIAutomation

  Xcodeの拡張developer tool、javascriptを使う。Accessibility Frameworkと一緒に使って画面のelementを検索できる。

* XCUITest

  Xcode7からXCTestとUIAutomationを統合したFrameworkが、Accessibility使わなくてもできる。Objective-CとSwiftで書ける。

XCUITestがXcodeに統合されているから_File->New UI Test Case_で簡単に作れるし、且つ動作の録画機能もある。
滅多にTestソース書かず、一回だけ手動でやればscriptが自動生成されるので、世の中ほとんどのアプリが十分楽にtestできると思う。

私の場合はちょっと複雑な状況だった。
testの途中にtest target appの画面から何度も外に出ることがあって、手動で戻させないとtestが続かないから、完全な自動化と言えない。
この問題を解決するため、Facebookの[WebDriverAgent](https://github.com/facebook/WebDriverAgent)を借りることとなった。

---
## Xcode XCTestの使い方

### 準備
Xcode projectを新規作成する時、案内画面の下に`Include Unit Tests`と`Include UI Tests`のcheckboxがあります。checkを入れると、Testに必要なBundle Build Targetが自動生成される。
![new project wizard](ios-ui-test/img-0101-wizard.png)

checkを忘れた場合、改めてNew Targetで`iOS Unit Testing Bundle`或いは`iOS UI Testing Bundle`を追加しなければ相応のTest Buildができない、testも当然できない。
![new testing bundle target](ios-ui-test/img-0102-target.png)

### Unit Test Case
Background methodをUnit Testだけに適応
* Targetsに`iOS Unit Testing Bundle`の存在を確認。`Host Application`の設定が特徴。
![bundle target](ios-ui-test/img-0204-general.png)
* Xcodeの`New File` 案内画面から `Unit Test Case Class`を選択し、test source の file名と言語を決める。
![new unit test](ios-ui-test/img-0201-testcase.png)
* Test Bundle Targetにcompile対象に含まれてるかを確認
![build phases](ios-ui-test/img-0202-buildphases.png)
* 嬉しいことに見慣れた`setup`、`tearDown` methodがtemplateから自動生成されてる。初期化とリセット処理を自分で入れろう。
* testしたい内容を`test`の文字で始まるmethodを追加したら完成です。
![source file](ios-ui-test/img-0203-source.png)
* Unit Test Caseを実行するにはsimulatorでなければならない。
![run test](ios-ui-test/img-0205-test.png)
    0. 一括全部実行[`cmd + u`]
    0. file単位で実行
    0. method単位で実行

__Tips__
> 一括実行の時、実行順番はmethod名で決める。順番を指定するworkaroundは`test_01_firstMethod`、`test_02_secondMethod`のようなフォーマットでmethod名にする

### UI Test Case
UI操作と画面のtestしたい
* Targetsに`iOS Unit Testing Bundle`の存在を確認。`Target Application`の設定が特徴。
![bundle target](ios-ui-test/img-0301-general.png)
* 今度Xcodeの`New File` 案内画面から 別の`UI Test Case Class`を選択し、file名を決める。
![new ui test](ios-ui-test/img-0302-testcase.png)
* Unit Testのtemplateとほぼ一緒、違うのは新しいAPI`XCUIApplication()`。 `XCUIApplication()`はappのinstanceを返すmethod、appはtest target appのこと、画面の各elementのroot elementでもある。
![source file](ios-ui-test/img-0303-source.png)
* `test`の文字で始まる空のmethodを追加し、括弧の中にクリックして編集のcursorをmethod内にすれば、編集windowの下にdisable状態だった赤いbuttonがenableに変わる。それは録画buttonだ。
![record button](ios-ui-test/img-0304-record.png)
* 録画buttonをクリックしてたら、test target appが起動され、test手順を踏まえてappを操作すれば、methodの中にtestのscriptが自動生成される。
* 再度録画buttonをクリックすると録画終了となる。test結果の比較処理を追加すればtest methodも追加完成する。
* それからtest実行すると、xxTest-Runner見たいなアプリがインストールし、起動され、その後target appに切り替す。

---

## WebDriverAgent (WDA)
Web test界には[Selenium](https://www.seleniumhq.org/)があるように、App test界は[Appium](http://appium.io/)がある。
Seleniumと同じjavascriptでtest scriptが作れる。
悪くないが、Appiumはtestのcross platform方案だ。
私は暫くiOSだけが欲しい。

Appiumを深く調べれば、本当にAppium iOS testを支えっているのは[WebDriverAgent](https://github.com/facebook/WebDriverAgent)だと見つけた。
名前の文字通り、WebからiOS端末を駆動できるagentです。
素晴らしいが、恐しい！iPhoneがハッキングされてるじゃないか？初めてiPhoneが勝手に動く動画を見た時私もそう思った。

WDAのオーブンソースを読むと、大きい三つの部分に分けられる。
1. HTTP Server

  Cartfileから、外部ライブラリ`github "marekcirkos/RoutingHTTPServer"`が使われてることが分かる。

2. endpoint API

  URLを解析してnative APIに振り分ける

3. XCTest native API

  コアな部分、test target Appを人形のように操る

### Architecture
remote controlの流れ
![architecture](ios-ui-test/img-0401-arch.png)
* remote PCからWDAのRunner AppにHTTP requestを送る
* WDAのRunner AppがXCTestのnative APIを呼ぶ
* XCTest APIがtarget Appを操作


### Web Base API
WDAのRunner Appが一つのHTTP Server Appだと思えばいい。
Server URLは`http://x.x.x.x:8100`のようになる。
後はrequestを送ってtest target Appの画面を検査したり操作したりするだけ。

__一部API__：

|endpoint|説明|
|:---|:---|
|`session`|接続情報|
|`inspector`|端末画面詳細|
|`source`|画面elements TreeのJSON|
|`session/:sessionId/elements`|element検索|
|`element/:elementId/elements`|sub-element検索|
|`element/:elementId/click`|elementをクリック|
|...|...|

### Browserからremote control
Browserでinspectorのendpointを送信すると、inspector画面が表示される。また端末のスクリーンコピーからelement treeを参照し、elementをクリックしたりできる。
![inspector](ios-ui-test/img-0402-inspector.png)

---

## WDAの違う使い方
Xcodeの録画機能は本当に便利且つ簡単で好きな機能だから、WDAのHTTP APIのため録画機能を捨てることが勿体ないなと思う。
実際HTTP Serverを起動せず、native APIの部分だけを使うもできる。

### 準備
極めて簡単、WDAのFrameworkをtest build targetに追加して、
Objective-CとSwiftで普通のFrameworkとして使えばいい。

後はいくつのAPIを覚えよう。

### Public API
XCTest Header filesに公開されているAPI。基本な検索と操作をサポートしている。

0. `XCUIApplication`

  test target appのinstanceを返す。

0. `XCUIApplication.init(bundleIdentifier: "com.apple.mobilesafari")`

  bundle IDを指定して任意のappのinstanceを返す。例はSafariを返す。

0. `buttons`

  sub-elementからbuttonを全部検索して返す。

0. `staticTexts`

  sub-elementから同じtextあるelementを検索して返す。例: `app.staticTexts["send event button"]`

0. `tap`

  elementのtap eventを発信する

> [API reference](https://developer.apple.com/documentation/xctest/user_interface_tests?language=objc)

### Private API
WDAが公開されたXCTestのHeader fileに満足できず、binary libraryから使われてるobjectとmethod symbolをdumpして、使えそうな隠しAPIを洗い出した。更にそのPrivate APIを利用して便利なAPIを増やした。

0. `FBApplication.fb_active`

  現在activeしているappのinstanceを返す。

0. `fb_waitUntilSnapshotIsStable`

  画面表示が安定しているかの判定。

0. `fb_screenshotWithError`

  スクリーンショット画像を返す。

### life show

## ios-deploy
まだ完全な自動ではない。
コマンドラインからxcodebuildを実行すればappやtest appが自動でinstallまた実行されるが、appを削除するために[ios-deploy](https://github.com/phonegap/ios-deploy)を借りた。

* bundleIdentifier指定app削除

  `ios-deploy -9 -1 <bundleIdentifier>`

## iOS beta test
