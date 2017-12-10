# IPA自动编译实现

xcode中ipa生成大致流程：Source > Clean > Archive > Export Archive > Sign IPA

整个过程可以基于 bash命令，xcodebuild命令以及mac自带的 defaults write/read 命令来实现。

具体编译脚本，请参考：[【autobuild-ipa】](https://github.com/tonyyls/autobuild-ipa.git)