
# 静态分析C语言生成函数调用关系的利器——calltree

[静态分析C语言生成函数调用关系的利器——cflow](https://blog.csdn.net/breaksoftware/article/details/75576878)

calltree -	static call tree generator for C programs

     The calltree command parses  a  collection	 of  input  files
     (assuming	C  syntax) and builds a	graph that represents the
     static call structure of these files.

     Calltree  is  similar  to	cflow(1)  but  unlike	cflow(1),
     calltree  is not based on lint(1).	 Calltree implements some
     more functions than cflow(1), but does not	list  the  return
     types of the functions. This is because calltree includes an
     own C parser and thus may be used even on systems that don't
     have lint(1).  The	disadvantage is	that the C parser that is
     used by calltree is not completely	correct	and may	not  find
     all  calls	of a function. This is mainly true for calls that
     are done via function pointers.

     Calltree is able to detect	recursive  function  calls  (e.g.
     functions	that  call themselves).	 Recursive function calls
     are marked	with an	ellipsis in the	output.

 对于C语言的项目，一些文件动辄几千行代码，上百个函数体，理解起来颇有些费劲。
 这个时候我们可以使用calltree工具对代码进行静态分析，
 然后产生调用关系树，使得我们可以对代码的构成有个初步的认识。
 这样可以让我们站在高处，俯览全局，制定出一个着实可行的阅读理解方案。

calltree是一个针对C语言代码的静态分析工具。它可以以图像的形式产出函数的调用关系。但是calltree和cflow不一样，cflow使用的是lint工具（一个更古老的工具）去预处理代码，而calltree使用的是自己的解释器。这样带来什么问题呢？那就是calltree可以运行于没有预装lint工具的系统，增强了其适用性。可惜的是calltree的C语言代码解释器实现的不是那么好，导致其可能无法找到所有函数。一个典型的例子就是通过函数指针进行函数调用的场景。

上文中还提到一个工具cflow。可能有人要问为什么不使用cflow去分析呢？每个工具都有利弊，calltree是我觉得正好够用且使用方便的一个工具。特别是其可以指定函数名去分析，这个原生的功能非常重要。因为一般在开源项目中，如果对全局或者某个文件进行分析，可能分析出非常杂乱的调用关系图。导致分析出来的结果对代码的解读没有一点帮助。

