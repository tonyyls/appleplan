iOS 应用安装包是后缀名为`.ipa`的文件，实际上是一个压缩文件。它是通过 `XCode` 经过一系列的步骤`Export`出来的安装程序。

# IPA组成

除了可正常运行的源码外，还包含：

1. 打包证书（Certificates）：用于标识该应用是哪个厂商开发的
2. 推送证书（APS Certificates）：需要用到苹果推送服务的应用才需要（Optional）
3. 应用标识（Identifiers）：应用的唯一标识\(AppID\)，Bundle Identifier
4. 描述文件（Provisioning）：将 AppID、证书信息、设备三者绑定在一起

## IPA类型

ipa包：分为**【发布包】**和【**测试包】**

### 发布包

发布包是用于正式对外发布的ipa，可以从【AppStore】或者【企业分发渠道】进行下载，视购买的开发计划而定。

发布包（Production）：Distribution（发布证书）+ Distribution（描述文件）

### 测试包

测试包通常是提供给测试人员或者部分已经登记在开发者网站上的`Devices`的用户使用的。也就是说使用者是受限的，无法随意分发给任何人。

测试包（Development）：Development（开发证书）+ Development（描述文件）

## IPA构建流程

xcode中ipa生成大致流程：Source > Clean > Archive > Export Archive > Sign IPA

