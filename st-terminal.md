## 关于terminal插件

### 使用git-bash

设置好git之后，发现git的bash很好用，配置了颜色，Linux基本功能命令也很齐全
想用其替换windows的cmd在st3种使用。

配置了如下代码（git的安装位置）
```
	"terminal": "C:\\Program Files\\Git\\git-cmd.exe",
	"parameters": ["/START", "%CWD%"]
```
启动的console与cmd基本无差别。

配置了如下代码即可
```
	"terminal": "C:\\Program Files\\Git\\git-bash.exe",
```
添加了启动参数会导致启动失败，可能是git-bash的工作方式，已经自动设置了当前路径。

### keybinding

添加*Key Bindings*
```
	{ "keys": ["ctrl+alt+t"],
    "command": "open_terminal" },
```
自动在当前目录打开console
