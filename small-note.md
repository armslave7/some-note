# atom使用代理

atom使用apm管理安装的包 只要设置了apm的代理即可，参见github->apm[主页](https://github.com/atom/apm#behind-a-firewall)


>Behind a firewall?

>If you are behind a firewall and seeing SSL errors when installing packages you can disable strict SSL by running:

>apm config set strict-ssl false
>Using a proxy?

>If you are using a HTTP(S) proxy you can configure apm to use it by running:

>apm config set https-proxy https://9.0.2.1:0


我使用了xx-net，为http代理，所以应为`apm config set https-proxy http://127.0.0.1:8087`

# Ubuntu安装netgen

Ubuntu 16.04版，安装netgen，进入主目录运行 ./configure出现_Tcl error_，运行`sudo apt-get install tcl-dev tk-dev mesa-common-dev libjpeg-dev libtogl-dev`解决./configure的依赖关系 make时出现_GL/glu.h: No such file or directory compilation terminated._ 错误，这是缺少opengl相关开发库的原因，安装相关库即可。

```script
sudo apt-get install mesa-common-dev
sudo apt-get install libgl1-mesa-dev libglu1-mesa-dev
```

## 解决/usr/bin/ld: cannot find -lxxx 问题

问题： 在linux环境编译应用程式或lib的source code时常常会出现如下的错误讯息： /usr/bin/ld: cannot find -lxxx 这些讯息会随着编译不同类型的source code 而有不同的结果出来如：

```
/usr/bin/ld: cannot find -lc
/usr/bin/ld: cannot find -lltdl
/usr/bin/ld: cannot find -lXtst
```

其中xxx即表示函式库文件名称，如上例的：libc.so、libltdl.so、libXtst.so。 其命名规则是：lib+库名(即xxx)+.so。 会发生这样的原因有以下三种情形： 1 系统没有安装相对应的lib 2 相对应的lib版本不对 3 lib(.so档)的symbolic link 不正确，没有连结到正确的函式库文件(.so) 解决方法： (1)先判断在/usr/lib 下的相对应的函式库文件(.so) 的symbolic link 是否正确，若不正确改成正确的连结目标即可解决问题。 (2)若不是symbolic link 的问题引起，而是系统缺少相对应的lib安装lib即可解决。 (3)如何安装缺少的lib： 以上面三个错误讯息为例： 错误1缺少libc的LIB 错误2缺少libltdl的LIB 错误3缺少libXtst的LIB 以Ubuntu为例： 先搜寻相对应的LIB再进行安装的作业如：

```
apt-cache search libc-dev
apt-cache search libltdl-dev
apt-cache search libXtst-dev
```

实例： 在进行输入法gcin的Source Code的编译时出现以下的错误讯息： /usr/bin/ld: cannot find -lXtst 经检查后发现是： lib(.so档)的symbolic link 不正确 解决方法如下： cd /usr/lib ln -s libXtst.so.6 libXtst.so 如果在/usr/lib的目录下找不到libXtst.so 档，那么就表示系统没有安装libXtst的函式库。 解法如下：

```
apt-get install libxtst-dev
```

## make出现Tcl/Tk错误

_/lib/libTogl.so: undefined reference to `Tk_InitStubs'_ 类似这样的提示错误，是因为Tcl和Tk版本的问题造成的，netgen版本为5.3.1。 在stackoverflow上面找到答案：

> This is an issue with linking to the correct versions of Tcl and Tk, as @keltar mentioned. If your goal is to build Netgen from source, your best bet is to stick with netgen-5.3.1, which works with tcl8.5 and tk8.5.

> I ran into this issue because when I ran sudo apt-get install tcl-dev tk-dev, apt-get fetched tcl8.6 and tk8.6 for me. (If you search online for these issues you can see that these linking issues can sometimes be resolved by updating your Tcl/Tk version.) I uninstalled the default tcl-dev and tk-dev packages and installed tcl8.5-dev and tk8.5-dev instead.

> Once you have the best fit version of Tcl/Tk, you can install Netgen from source. For example,

> ./configure --with-tclconfig=/usr/lib/tcl8.5/ --with-tkconfig=/usr/lib/tk8.5/ make make install

[链接](http://stackoverflow.com/questions/26334781/libtogl-undefined-references#comment41335571_26334781)

## 运行错误

编译链接后，运行时出现类似错误

```
Major opcode of failed request:  155 (GLX)
Minor opcode of failed request:  3 (X_GLXCreateContext)
Value in failed request:  0x0
Serial number of failed request:  559
Current serial number in output stream:  560
```

在sourceforge上找到解决方法

> edit file ng/drawing.tcl line 14: if {[catch {togl .ndraw -width 400 -height 300 -rgba true -double true -depth true -privatecmap false -stereo false -indirect true }] } { .... change "-indirect true" to "-indirect false"

[链接](https://sourceforge.net/p/netgen-mesher/discussion/905307/thread/946ccfc2/)

# chrome 设置用户目录

在chrome的快捷方式，或者可执行文件中，添加运行参数即可指定chrome用户目录和缓存目录

`--disk-cache-dir=`指定缓存目录 `--user-data-dir=`指定用户目录

例如：

```
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disk-cache-dir="C:\Cache\Chrome"
```

指定chrome缓存目录在"C:\Cache\Chrome"目录下。

# 使用endnot添加中文格式参考文献

将_geebinf modified by zz.ens_复制到_EndNote_安装目录的_Styles_下，依次"edit"->"output styles"-> "Open Style Manager..."，在弹出的对话框中，勾选"geebinf modified by zz"，最后在"output styles"中选中即可。

参考链接[EndNote中文Style](https://cnzhx.net/blog/endnote-output-style-cnzhx/)

2017/1/19 20:40:17
