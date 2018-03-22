---
title: iOS UI Test integrates with WDA(WebDriverAgent)
lang: ja
date: 2018-03-07 19:30:10
tags: [iOS, Test, UITest, XCTest, WebDriverAgent, WDA]
---

## XCTest
XcodeはUnit Testをサポートするため`XCTest`というFrameworkを提供してる。
余計なことを言うが、iOS Runtime Frameworkではないので、Test Buildに入れるFrameworkだ。自動操作の機能が強過ぎてアップスドアにアップできないかも。

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
標準のAPIが足りないと思えば、Facebookの[WebDriverAgent](https://github.com/facebook/WebDriverAgent)をcheckしてみて。
Private APIが一杯掘り出されてる。

---
## Xcode XCTestの基本

### 準備
Xcode projectを新規作成する時、案内画面の下に`Include Unit Tests`と`Include UI Tests`のcheckboxがあります。checkを入れると、Testに必要なBundle Build Targetが自動生成される。
{% asset_img img-0101-wizard.png new project wizard %}

checkを忘れた場合、改めてNew Targetで`iOS Unit Testing Bundle`或いは`iOS UI Testing Bundle`を追加しなければ相応のTest Buildができない、testも当然できない。
{% asset_img img-0102-target.png new testing bundle target %}

### Unit Test Case
Background methodをUnit Testだけに適応
* Targetsに`iOS Unit Testing Bundle`の存在を確認。`Host Application`の設定が特徴。
{% asset_img img-0204-general.png bundle target %}
* Xcodeの`New File` 案内画面から `Unit Test Case Class`を選択し、test source の file名と言語を決める。
{% asset_img img-0201-testcase.png new unit test %}
* Test Bundle Targetにcompile対象に含まれてるかを確認
{% asset_img img-0202-buildphases.png build phases %}
* 嬉しいことに見慣れた`setup`、`tearDown` methodがtemplateから自動生成されてる。初期化とリセット処理を自分で入れろう。
* testしたい内容を`test`の文字で始まるmethodを追加したら完成です。
{% asset_img img-0203-source.png source file %}
* Unit Test Caseを実行するにはsimulatorでなければならない。
{% asset_img img-0205-test.png run test %}
    0. 一括全部実行[`cmd + u`]
    0. file単位で実行
    0. method単位で実行

__Tips__

一括実行の時、実行順番はmethod名で決める。順番を指定するworkaroundは`test_01_firstMethod`、`test_02_secondMethod`のようなフォーマットでmethod名にする

### UI Test Case
UI操作と画面のtestしたい
* Targetsに`iOS Unit Testing Bundle`の存在を確認。`Target Application`の設定が特徴。
{% asset_img img-0301-general.png bundle target %}
* 今度Xcodeの`New File` 案内画面から 別の`UI Test Case Class`を選択し、file名を決める。
{% asset_img img-0302-testcase.png new ui test %}
* Unit Testのtemplateとほぼ一緒、違うのは新しいAPI`XCUIApplication()`。 `XCUIApplication()`はappのinstanceを返すmethod、appはtest target appのこと、画面の各elementのroot elementでもある。
{% asset_img img-0303-source.png source file %}
* `test`の文字で始まる空のmethodを追加し、括弧の中にクリックして編集のcursorをmethod内にすれば、編集windowの下にdisable状態だった赤いbuttonがenableに変わる。それは録画buttonだ。
{% asset_img img-0304-record.png record button %}
* 録画buttonをクリックしてたら、test target appが起動され、test手順を踏まえてappを操作すれば、methodの中にtestのscriptが自動生成される。
* 再度録画buttonをクリックすると録画終了となる。test結果の判定処理を追加すればtest methodが完成する。
* それからtest実行すると、録画された動作がreplayされる。

---

## WebDriverAgent (WDA)
Web test界には[Selenium](https://www.seleniumhq.org/)があるように、App test界は[Appium](http://appium.io/)がある。
Seleniumと同じjavascriptでtest scriptが作れる。
悪くないが、Appiumはtestのcross platform方案だ。
私の場合は暫くiOSだけが欲しい。

Appiumを深く調べれば、本当にAppium iOS testを支えっているのは[WebDriverAgent](https://github.com/facebook/WebDriverAgent)だと見つけた。
名前の文字通り、WebからiOS端末を駆動できるagentです。
素晴らしいが、恐しい！iPhoneがハッキングされてるじゃないか？初めてiPhoneが勝手に動く動画を見た時私もそう思った。

WDAのオーブンソースを読むと、大きい三つの部分に分けられる。
1. HTTP Server

  Cartfileから、外部ライブラリ`github "marekcirkos/RoutingHTTPServer"`が使われてることが分かる。

2. endpoint API

  URLを解析してnative APIに振り分ける

3. native API

  コアな部分、XCTestのPublic APIとPrivate APIを基づき使い易くにしてくれた

### Architecture
remote controlの流れ
{% asset_img img-0401-arch.png architecture %}
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
{% asset_img img-0402-inspector.png inspector %}

__Tips__

WIFIは便利ですけど、WIFIない時にUSB経由したい場合以下のツールが使えそうになる。
* [mobiledevice](https://github.com/imkira/mobiledevice)
* [usbmuxd](https://github.com/libimobiledevice/usbmuxd)

---

## WDA Native API
cross platformではないし、remote操作も要らないから必ずWDAのHTTP Serverを使うことがない。WDAはXCTestのたくさんPrivate APIを整理したことが凄く助かる。標準APIが満足できない時、助かるかもしれない。Objective-CとSwift以外のリソースも要らない。

使い方も簡単。普通のFramework LibraryみたいWDAのFrameworkをtest build targetに追加して、XCTest APIと一緒にObjective-C或いはSwiftでtest caseを書くと、private APIと便利な録画機能両方のメリットが受けられる。

後はいくつのAPIを覚えよう。

### Public API
XCTest Header filesに公開されているAPI。基本な検索と操作をサポートしている。

例:
0. test target appのinstanceを返す。
```swift
let app = XCUIApplication()
```
0. appを起動させる。
```swift
app.launch()
```
0. SafariのbundleIdentifierを指定してをSafariのinstanceを返す。
```swift
let safari = XCUIApplication.init(bundleIdentifier: "com.apple.mobilesafari")
```
0. listから"normal event"の行をtapする。
```swift
app.tables.staticTexts["normal event"].tap()
```
0. "IDをリセット"のbuttonをtapする。
```swift
app.sheets.buttons["IDをリセット"].tap()
```
0. appを停止させる。
```swift
app.terminate()
```


> [API reference](https://developer.apple.com/documentation/xctest/user_interface_tests?language=objc)

### Private API
WDAが公開されたXCTestのHeader fileに満足できず、binary libraryから使われてるobjectとmethod symbolをdumpして、使えそうな隠しAPIを洗い出した。更にそのPrivate APIを利用して便利なAPIを増やした。

0. 現在activeしているappのinstanceを返す。
```swift
FBApplication.fb_active
```
0. 画面表示が安定しているかの判定。
```swift
fb_waitUntilSnapshotIsStable
```
0. 座標指定してtap。
```swift
element.fb_tapCoordinate(relativeCoordinate, &err)
```

## 残タスク
まだまだ完全な自動操作ではない。
コマンドラインからxcodebuildを実行すればappやtest appが自動でinstallまた実行されるが、appを削除するために別のツールが必要となる。

* [ios-deploy](https://github.com/phonegap/ios-deploy)

```
# device ID listを出力
ios-deploy -c

# bundleIdentifier指定app削除
ios-deploy -9 -1 <bundleIdentifier>
```

* [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice)

## iOS beta test
iOS betaを使いためよく`xcode-select`を勧められるが、使い終わった後もう一回`xcode-select`で普通バージョンに戻らなければならないから非常に面倒臭い。

`xcode-select`のmanual docを見ると実際は環境変数DEVELOPER_DIRを変更したにすぎないため、
terminalで手動でDEVELOPER_DIRを変更することもできる。
```
export DEVELOPER_DIR=/Applications/Xcode-beta.app/Contents/Developer`
```
そうするとxcodeバージョンの切り替えはterminalのsession以外に影響持たない、設定を戻る必要がない。

## Swift Script
shell scriptで複雑なtest caseを管理するには抵抗がある。実際実行環境はxcode必須だから必ずSwiftがinstallされている。
rubyや、pythonと同じ、`Swift main.swift`でプログラムがすぐ実行できるし、script言語としても向いている。
下はScalaの`Process`Classの真似でStringの拡張メソットを作れば更に便利になる。
```Swift
// Stringをcommandに変換
extension String {
    public func run(_ dir:String = ".") -> Int32{
        let p = toProcess(dir: dir)
        p.launch()
        p.waitUntilExit()
        return p.terminationStatus
    }
}

// 実行例
let cmd = "xcodebuild" +
            " -scheme \(scheme)" +
            " -configuration \(configuration)" +
            " -sdk iphoneos" +
            " -destination platform=iOS,id=\(deviceIdentifier)" +
            " build"
cmd.run()
```
