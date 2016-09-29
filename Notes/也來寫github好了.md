# 也來寫<font color="yellow">Github</font>好了

## Github 裝屌指南

---

# 我的第一個repo
- 醒醒吧你沒有星星
  ![my first repo](http://i.imgur.com/F3BX9WX.png)

---

[![afnetworking](http://i.imgur.com/Xx09Ydi.png)](https://github.com/AFNetworking/AFNetworking)

---

# 沒有高手的程度，也要有高手的帥度

---

# 帥氣三大要素

1. 花俏的badge們
2. 自動化測試、發佈
3. CI CD 

---

# 何以裝屌

1. 花俏的badge們<font color="orange"> (codecov, cocoapods) </font>
2. 自動化測試   <font color="orange">  (Travis CI + Fastlane)</font>
3. 用看起來很專業的commit log 自動生成 Release Note
4. CI CD    <font color="orange">    (Fastlane)</font>

---

# DUFramework

- Do nothing but good looking.
- That's the goal.

---

# 今天寫啥Log?

~~[擲筊決定](http://whatthecommit.com)~~

---

# Pro-looking commit log

![commit](http://i.imgur.com/nOx11cq.png)

---

![commit log](http://i.imgur.com/xsN6sfR.png)

---

# Voila! CHANGELOG from commits!!

![changelog](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016010603.png)

---

# Commit message format 

三本柱：Header, Body, Footer
```shell
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

Ref: [Angular Style Commit Guide](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#-git-commit-guidelines)

---

# [Commitizen](https://github.com/commitizen/cz-cli)

* 統一commit log風格
* commit 目的更明確
* 可自動生成change log

---

# Header（唯一必要）

* type: commit類別，有feat, fix, refactor等
* scope: commit影響範圍，如model, controller, build, test
* subject: 標題(動詞開頭，祈使句型，不加句號)
  （第一行會被限制在100字內）

---

![type](http://i.imgur.com/YByeAPO.png)

---

# Body

* 具體描述commit
* 動詞用現在式（change not changed)
* 若可能說明動機與先前行為之比對

---

# Footer

* BREAKING CHANGE: 不相容之變動，以BREAKING CHANGE開頭，接著描述更動及其緣由和migration方式
* Close ISSUE:
```shell
Closes #4, #5566
```
(Fixes等字也可，可用字參考[這裡](https://help.github.com/articles/closing-issues-via-commit-messages/))

---

# BREAKING CHNAGE

```shell
BREAKIING CHANGE: isolate scope binding has changed.
    To migrate the code follow the example below:
    Before:
    {...}
    After:
    {...}
```

---

# Revert

* 特殊情況，用在撤銷先前的commit
* Body 格式一律固定
```shell
revert: feat(pencil): add 'graphWidth' option
This reverts to commit 6687dzcdeqrXXXXXXX.
```

---

# Validate commit message

* [JS Code](https://github.com/pofat/DUFramework/blob/develop/validate-commit-msg.js)
* 放入repo，取名validate-commit-msg.js
```javascript
"config": {
  "ghooks":{
    "commit-msg":"./validate-commit-msg.js"
  }
}
```

---

![validation](http://i.imgur.com/WzabZ91.png)

---

# Generate Change Log

* Use `conventional-changelog`
* 在既有檔案中再加上新的log（從前一次release至今）
```shell
conventional-changelog -p angular -o CHANGELOG.md -w
```
* 生成全部的changelog
```shell
conventional-changelog -p angular -o CHANGELOG.md -w -r 0
```

---

# Make it easier

* 加入package.json
```javascript
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -o CHANGELOG.md -w"
  }
}
```
* 執行
```shell
npm run changelog
```

---

# Start Fastlane

1. 以<font color="grey"> `sudo gem install fastlane`</font> 安裝
2. 在專案根裡新增 fastlane 資料夾，並在其目錄下建立三種檔案：

   1. Fastfile 
   2. actions (資料夾，放自定義function in ruby)
   3. .env 等環境變數

---

# Write your lanes
```ruby
desc "Runs all tests for the given environment"
desc "####Example:"
desc "```\nfastlane test_framework configuration:Debug --env ios84\n```"
desc ""

lane :test_framework do |options|
  scan(
    configuration: options[:configuration]
  )
end
```

---

# 善用 .env*

```shell
DU_IOS_SDK=iphonesimulator10.0
DU_CONFIGURATION=Release
SCAN_WORKSPACE=$DU_WORKSPACE
SCAN_SCHEME=$DU_IOS_FRAMEWORK_SCHEME
SCAN_DESTINATION="OS=10.0,name=iPhone 7"
SCAN_SDK=$DU_IOS_SDK
SCAN_OUTPUT_DIRECTORY=fastlane/test-output
```

---

![envs](http://i.imgur.com/DgyappJ.png)

---

# Run test

```shell
fastlane test_framework configuration:Release env:ios93
```

---

# Then?

* 引入不同環境設定檔
* 批次執行
* 對以上做版本控制

---

# 在Travis CI中使用fastlane

```yaml
before_install:
  - gem install fastlane --no-rdoc --no-ri --no-document --quiet
  - gem install xcpretty --no-rdoc --no-ri --no-document --quiet
```

---

# <font color="orange">ISSUE of Xcode 7.3 image</font>

```yaml
before_install:
# workaround for Fastlane conflict https://github.com/travis-ci/travis-ci/issues/6325
  - gem uninstall json -v 2.0.1
  - gem update fastlane --no-rdoc --no-ri --no-document --quiet
```

---

# 設定各環境的測試

```yaml
matrix:
  include:
    - osx_image: xcode8
      env: FASTLANE_LANE=code_coverage FASTLANE_ENV=default
    - osx_image: xcode8
      env: FASTLANE_ENV=ios84
    - osx_image: xcode8
      env: FASTLANE_ENV=ios93
```

---

# 執行測試

```yaml
env:
  global:
  - FASTLANE_LANE=ci_commit
script:
  - set -o pipefail
  - fastlane $FASTLANE_LANE configuration:Debug --env $FASTLANE_ENV
  - fastlane $FASTLANE_LANE configuration:Release --env $FASTLANE_ENV
```

---

# CI_COMMIT

```ruby
lane :ci_commit do |options|
  if ENV["DU_CONFIGURATION"]
    configuration = ENV["DU_CONFIGURATION"]
  else
    configuration = "Release"
  end

  test_framework(configuration: configuration)

  build_example(configuration: configuration)

end
```

---

# 發佈前的準備

1. Release git flow
2. Bump version number
3. Commit, tag & push

---

# Release git flow

![git flow](http://i.imgur.com/e00gt26.jpg)
[圖片出處](http://www.slideshare.net/lioramilbaum/community-live-1)

---

# 具體來說

1. create a release branch from develop
2. prepare for release (cocoapod)
3. merge with master
4. create tag and push
5. notify completion
- 執行範例
```shell
fastlane prepare_framework_release version:0.3.4 --env deploy
```

---

# 完成發佈

1. ensure clean branch (master)
2. git pull
3. pod trunk push
4. spec lint
- 本機執行範例
```shell
fastlane complete_framework_release skip_ci_check:true --env deploy
```

---

# 整合至Travis CI

```yaml
deploy:
  provider: script
  script: fastlane complete_framework_release --env deploy
  on:
    tags: true
```

---

# All together

1. 開發完成，準備release
```shell
fastlane prepare_framework_release version:0.3.4 --env deploy
```
1. 上述會push 到master branch，由Travis CI接手進行test
2. 完成test後Travis CI會進行deploy （push podspec)
3. 一行指令搞定！

---

# Badges
沒有它，一切都不帥惹

---

# Build badge

- Travis CI
  ![build pass](http://i.imgur.com/ZsKoblP.png)

---

# Codecov badge

![codecov](http://i.imgur.com/LOqA0Q6.png)

---

# Pod version

* https://img.shields.io/cocoapods/v/URFramework.svg
* ![pod version](https://img.shields.io/cocoapods/v/DUFramework.svg)

---

# Platform Badge

* https://img.shields.io/cocoapods/p/URFramework.svg?style=flat
* ![platform](https://img.shields.io/cocoapods/p/DUFramework.svg?style=flat)

---

# 中二 Badge

![badge](https://img.shields.io/badge/%E5%90%8C%E6%AD%A5%E7%8E%87-400%25-red.svg)

---

# Final Result

* [DUFramework](https://github.com/pofat/DUFramework)
* [Fastlane](https://github.com/pofat/DUFrameworkFastlane)（所有的lanes皆在此）
* [Document](http://cocoadocs.org/docsets/DUFramework/0.1.4/)

---

[![Diuit](http://api.diuit.com/images/logo.png)](http://api.diuit.com/)

今日指南若有任何批評意見，歡迎[指教](mailto:pofattseng@diuit.com)

---


