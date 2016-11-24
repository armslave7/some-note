# latex中的空行

在latex的导言区中，空行是不能随意插入的，譬如设置ctex的参数时，如下代码会编译报错：


    \ctexset{section={
				format+ = \zihao{-4} \heiti \raggedright,
				name={,、},
				number= \chinese{section},
				beforeskip = 1.0ex plus 0.2ex minus .2ex,
				afterskip = 1.0ex plus 0.2ex minus .2ex,
				aftername = \hspace{0pt}
			},
			subsection = {
				format+ = \zihao{5} \heiti \raggedright,
				name = {,、},
				number = \arabic{subsection},
				beforeskip = 1.0ex plus 0.2ex minus .2ex,
				afterskip = 1.0ex plus 0.2ex minus .2ex,
				aftername = \hspace{0pt}
			}
			...
	}


此段代码设置ctex中，中文节和小结的格式，使其更符合中文排版习惯。

因为在省略号处，多了一行空格（有时写代码的习惯），导致编译出错，但是在latex的正文区域，多余的空行是不影响的，但是在导言区中，空行影响了编译，这点需要注意。

