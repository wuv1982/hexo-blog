---
title: iOS UI Test integrates with WDA(WebDriverAgent)
lang: ja
date: 2018-03-07 19:30:10
tags: iOS, Test, UITest, XCTest, WebDriverAgent, WDA
---

XcodeはUnitTestをサポートするため`XCTest`というFrameworkを提供してる。
余計なことを言うが、iOSのRuntime Frameworkではない、XcodeのTest Build時のFrameworkだ。
Xcode6まではbackground methodのtestだけだったが、Xcode7からUIの自動操作によってtestもできるようになった。
Xcodeに統合されているから_File->New UI Test Case_で簡単に作れるし、且つ動作の録画機能もある。
滅多にTestソース書かず、一回だけ手動でやればscriptが自動生成されるので、
世の中ほとんどのアプリが十分楽にtestできると思う。

私の場合はちょっと複雑な状況だった。
test target appの画面から一度外に出ることがあって、手動で戻さないとtestが続かないから、完全な自動化と言えない。
この問題を解決するため、Facebookの[WebDriverAgent](https://github.com/facebook/WebDriverAgent)を借りることとなった。

---
## Xcode XCTestの一般的な使い方

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
* 録画buttonをクリックしてたら、test target appが起動され、test手順を踏まえてappを操作すれば、なんと空だったmethodの中にtestのscriptが自動生成される。
* 再度録画buttonをクリックすると録画終了となる。test結果の比較処理を追加すればtest methodも追加完成する。

---

## WebDriverAgent (WDA)
Web test界には[Selenium](https://www.seleniumhq.org/)があるように、App test界は[Appium](http://appium.io/)がある。
Seleniumと同じjavascriptでtest scriptが作れる。
悪くないが、Appiumはtestのcross platform方案だ。
私は暫くiOSだけが欲しい。

Appiumを深く調べれば、本当にAppium iOS testを支えっているのは[WebDriverAgent](https://github.com/facebook/WebDriverAgent)だと見つけた。
名前の文字通り、WebからiOS端末を駆動できるagentです。
素晴らしいが、恐しい！iPhoneがハッキングされてるじゃないか？初めてiPhoneが勝手に動く動画を見た時私もそう思った。

### Architecture
remote controlの流れ
![architecture](ios-ui-test/img-0401-arch.png)
* remote PCからWDAのRunner AppにHTTP requestを送る
* WDAのRunner AppがXCTestのAPI(ほぼPrivate API)を呼ぶ
* XCTest APIがtarget Appを操作


### Web Base API
WDAのRunner Appが一つのHTTP Server Appだと思えばいい。
Server URLは`http://x.x.x.x:8100`のようになる。
後はrequestを送ってtest target Appの画面を検査したり操作したりするだけ。

__一部API__：

|endpoint|説明|
|:---|:---|
|`session`|接続情報|
|`inspector`|端末画面情報|
|`source`|画面elements TreeのJSON|
|`session`/:sessionId/elements|element検索|
|`element/:elementId/elements`|subelement検索|
|`element/:elementId/click`|elementをクリック|
|...|...|

### Browserからremote control
Browserでinspectorのendpointを送信すると、inspector画面が表示される。また端末のスクリーンコピーからelement treeを参照し、elementをクリックしたりできる。
![inspector](ios-ui-test/img-0402-inspector.png)

---

## 改造
私WDAたくさんのAPIを覚えてscriptを書くことをやりたくないと思う。Xcodeの録画機能があんなに便利且つ簡単なのに捨てることが勿体ない。
WDAの根本はHTTP Serverではない、XCTest APIを操作する部分だ。この部分だけを使って自分のTest-Runner-Appを作ることにした。

### XCTest Private API

### life show

## iOS beta test
