# 在安装使用Linux过程中的一些坑

### virtualbox中安装archlinux

在使用virtualbox的iso文件安装增强功能时出现```archelinux failed to set up service vboxadd```类似的错误。
这是因为在编译过程中缺少```linux-header```，使用```pacman -S linux-header```即可修复。

### 使用yaourt安装软件的错误

在使用yaourt安装软件时，出现```error: key "****" could not be looked up remotely```，这是签名错误。
使用```pacman -S archlinuxcn-keyring```即可修复。[出处](https://bbs.archlinuxcn.org/viewtopic.php?id=3124)


### xubuntu中文输入问题

xubuntu-16.04LTS，默认安装好之后中文输入时没有出现候选项
[参考](http://blog.csdn.net/qq_21397217/article/details/52447263)
```
1. 安装Fcitx所需组件
$ sudo apt install fcitx fcitx-tools fcitx-config* fcitx-frontend* fcitx-module* fcitx-ui-* presage

2. 移除多余的组件(仅针对非KDE桌面)
因为上面的安装命令为了方便使用了通配符安装，所以会多安装一个fcitx-module-kimpanel，这个组件可能导致在非KDE桌面环境下Fcitx输入中文时不显示候选词框，如果是KDE桌面则无需移除。
$ sudo apt remove fcitx-module-kimpanel

3. 根据需要安装一个或多个输入法
$ sudo apt install fcitx-pinyin            # 拼音
$ sudo apt install fcitx-sunpinyin         # sun拼音
$ sudo apt install fcitx-googlepinyin      # google拼音
$ sudo apt install fcitx-table-wubi-large  # 五笔
```
