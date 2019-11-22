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
