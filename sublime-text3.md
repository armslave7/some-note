# sublime text的一些插件和配置

## package control

package control必备

## side bar

方便管理同一个文件夹的文件，需要使用`File->Open Folder...`来使用，打开文件该插件并没有启用

## ConverToUTF8

转换编码方式插件，有中文或者其他编码文字必备

## terminal

方便在当前文件目录下打开终端，Ubuntu下测试可直接使用，win7默认打开*power shell*，根据其官方插件给出的说明，配置如下即可打开默认的cmd

		{
		  "terminal": "C:\\Windows\\System32\\cmd.exe",
		  "parameters": ["/START", "%CWD%"]
		}

## MarkDownExtend&MarkDownEditing&MonokaiExtend

非常方便的编辑markdown的插件，输入习惯和markdownpad相似
需要配合MarkdownExtend和MonokaiExtend，其自带的配色很难适应（不知是否是我自己配置的问题）

## MarkDownPreview

迅速查看markdown

## Material Theme（主题）

安装这个主题可以不用安装MonokaiExtend，这个主题配置很简单，package->active->config，实时配置，这个主题有很漂亮


## fortran

安装*Fortran*插件，支持Fortran语法等

在*Linux*中不需要额外修改什么，但是在windows中，需要修改一些设置才可以使用。

*sublime text*版本：build-3126
系统window7 和window10
在系统中安装好mingw之后，配置好环境变量
在st3中直接build，console中会出现错误信息，初步查看应该是字符引用出错

    gfortran 'E:\markdown\tmp.f90' error.

这需要修改内置的build system，在st2中build的文件信息在package文件中可以找到，但是st3中并没有在package文件夹中，

这需要在st3中安装PackageResourceViewer（通过package control安装），之后打开package control界面，输入prv，然后选择open source选项，在下拉菜单中找到需要修改的语言项目，这里为fortran，之后找到GFortran.sublime-build文件，进入这个文件进行修改即可。

通过对比c的build文件，发现fortran的build文件中单引号的使用有误，原有的配置代码为：

    "shell_cmd": "gfortran '${file}' -o '${file_path}/${file_base_name}'"

应改为：

    "shell_cmd": "gfortran \"${file}\" -o \"${file_path}/${file_base_name}\""

完整的build文件内容为：

    {
        "shell_cmd": "gfortran \"${file}\" -o \"${file_path}/${file_base_name}\"",
        "file_regex": "^(?xi:( ^[/] [^:]* ) : (\\d+) : (\\d+) :)",
        "working_dir": "${file_path}",
        "selector": "source.modern-fortran, source.fixedform-fortran",
        "syntax": "GFortranBuild.sublime-syntax",

        "variants":
        [
            {
                "name": "Run",
                "shell_cmd": "gfortran \"${file}\" -o \"${file_path}/${file_base_name}\" && \"${file_path}/${file_base_name}\""
            }
        ]
    }





## Settings

很多设置都可以在setting中中配置，如字号，当前行高亮，设置tab等等。

配置譬如

	{
		"color_scheme": "Packages/Material Theme/schemes/Material-Theme-Darker.tmTheme",
		"font_size": 14,
		"default_line_ending": "unix",
		"highlight_line": true,
		"highlight_modified_tabs": true,
		"fortran_disable_docs": false,
		"fortran_disable_latex": false,
		"ignored_packages":
		[
			"Vintage"
		],
		"material_theme_big_fileicons": true,
		"material_theme_bright_scrollbars": true,
		"theme": "Material-Theme-Darker.sublime-theme"
	}



##  其他插件和设置

###  使用sublime text3写LaTeX文档

使用package control安装LaTeXTools，设置好texlive，像这样

	"texpath" : "E:\\texlive\\2016\\bin\\win32;$PATH",

如果要在处理中文文档，需要使用exlatex来编译文档，这在texstudio中只需要设置即可，但在LaTeXtools没有相关设置，可以在文档首行指定编译器，譬如：

	%!TEX program = xelatex

这样便可以处理中文。

###  markdown代码块行号

在网页中发现很多贴出来的代码是有行号，并且是带语法高亮的，经过搜索发现，同样的MarkDown代码，使用不同的渲染器，出来的结果不一样，所以这是渲染选项或者渲染器造成的，在markdownpreview设置中，设置如下代码，便可实现代码显示行号

    "enable_pygments": true,
    "enabled_extensions": [
                "codehilite(use_pygments=True)",
                "codehilite(guess_lang=True)",
                "codehilite(linenums=True)",
                "codehilite(noclasses=True)",
                //"codehilite(pygments_style=emacs)",

    ],

但是这样仅显示了行号，并没有实现语法颜色的渲染，设置中`"codehilite(pygments_style=emacs)"`这一条语句加上的话行号也会消失，暂时不知道原因，现在这样也还是很好的。


### InsertData

插入时间的插件，设置非常简单，需要修改格式可以查看其文档
修改快捷键只需要在*key bindings*中设置即可



