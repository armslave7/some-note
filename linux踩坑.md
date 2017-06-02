# 在安装使用Linux过程中的一些坑

### virtualbox中安装archlinux

在使用virtualbox的iso文件安装增强功能时出现```archelinux failed to set up service vboxadd```类似的错误。
这是因为在编译过程中缺少```linux-header```，使用```pacman -S linux-header```即可修复。

### 使用yaourt安装软件的错误

在使用yaourt安装软件时，出现```error: key "****" could not be looked up remotely```，这是签名错误。
使用```pacman -S archlinuxcn-keyring```即可修复。[出处](https://bbs.archlinuxcn.org/viewtopic.php?id=3124)


