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

calltree是在linux下面看c代码（尤其是复杂的内核代码）的神器。

推荐  calltree+vim + ctags + cscope + taglist 【 vim: 搭建vim看代码的环境   http://www.cnblogs.com/mylinux/p/5013588.html】

或者 calltree + source insight

source insight能方便地查看向上和向下的函数（变量等）调用关系，并且支持多种语言，几乎是无可替代的。但调用深度太大的时候，人就记不住了，这个时候
calltree可以生成一个全局的调用图，便于很快掌握代码框架。

如想看内核代码start_kernel干了些什么：

#calltree -np -b  list=start_kernel    depth=3 `find ./init/ ./kernel/ -name "*.c"` > maps

#vi maps




 对于C语言的项目，一些文件动辄几千行代码，上百个函数体，理解起来颇有些费劲。
 这个时候我们可以使用calltree工具对代码进行静态分析，
 然后产生调用关系树，使得我们可以对代码的构成有个初步的认识。
 这样可以让我们站在高处，俯览全局，制定出一个着实可行的阅读理解方案。

calltree是一个针对C语言代码的静态分析工具。它可以以图像的形式产出函数的调用关系。但是calltree和cflow不一样，cflow使用的是lint工具（一个更古老的工具）去预处理代码，而calltree使用的是自己的解释器。这样带来什么问题呢？那就是calltree可以运行于没有预装lint工具的系统，增强了其适用性。可惜的是calltree的C语言代码解释器实现的不是那么好，导致其可能无法找到所有函数。一个典型的例子就是通过函数指针进行函数调用的场景。

上文中还提到一个工具cflow。可能有人要问为什么不使用cflow去分析呢？每个工具都有利弊，calltree是我觉得正好够用且使用方便的一个工具。特别是其可以指定函数名去分析，这个原生的功能非常重要。因为一般在开源项目中，如果对全局或者某个文件进行分析，可能分析出非常杂乱的调用关系图。导致分析出来的结果对代码的解读没有一点帮助。



# calltree的编译

执行make命令 可能会报错   提示 fexecve 与 getline 函数 与GCC的函数重名

所以需要修改这些函数名

```sh
# 查找所有.c/.h文件 将其中的fexecve 修改为 fexecve_calltree    ；  getline 修改为 getline_calltree
find . -name "*.[c|h]" |xargs sed -i -e "s/fexecve/fexecve_calltree/"
find . -name "*.[c|h]" |xargs sed -i -e "s/getline//"

```

# calltree的使用

calltree --help指令我们可以看到其参数说明

        -g输出函数所在文件的目录
        -m参数只用于分析main函数中的函数调用关系。
        -p参数是默认的。它表示要使用C语言预处理程序分析代码。缺点是它会产生很多我们不关心的消息。
        -np和-p是相反的。它表示不要使用C语言预处理程序分析代码。如果指定它，可能会导致分析过程出错。因为像开源项目，有几个不需要预处理处理下呢？
        -xvcg参数表示导出一个可以使用VCG软件处理的格式的文件。
        -dot参数表示导出一个dot格式文件，可以供graphviz处理的。
        
        -b 就是那个竖线了，很直观地显示缩进层次。
        -g 打印内部函数的所属文件名及行号，外部函数所属文件名和行号也是可打印的，详man
        -np 不要调用c预处理器，这样打印出的界面不会很杂乱，但也可能会产生错误哦，如果我们只看函数的调用关系的话，不会有大问题。
        -m 告诉程序从main开始
        还有一个重要的选项是listfunction ，缩写是lf，用来只打印某个函数中的调用，用法是： lf=your_function
        depth=#选项： 例如： calltree -gb -np -m bind9/bin/named/*.[c.h] depth=2 > codecalltree.txt

注意：
　　调用关系一般比较复杂，最好设置好（1）想要关心的函数（2）调用深度（3）关心的目录，否则又会引入过多无关选项，干扰视线。
  
## 文本输出

文本输出只是为了展示calltree的能力。我们libev库的ev_run方法为例，切到代码目录后调用
        
calltree -bg list="ev_run" *.c 


## 图形化输出

图像化输出我们只看dot格式。

首先我们使用下面命令把结果保存到我们指定的文件中

    calltree -dot list="ev_run" *.c > ev_run.dot

然后调用graphviz（没有安装的可以使用apt-get install graphviz先安装）

    dot -Tgif ev_run.dot -o ev_run.gif

然后我们将图片打开查看，就发现图形化输出更加便于理解。

# 总结

calltree+graphviz是分析C源码很好的组合。有人提出使用cflow替代calltree，一是因为calltree不再维护，而cflow“一直在更新”。
然后我发现，calltree的最后一个版本是2004年的；cflow最后一个版本是2011年的。就目前而言，
它们两的更新都停滞了，所以我认为“更新进度”不能成为排除calltree而选择cflow的原因。而且相对于cflow，calltree的使用非常方便，
并且它具有一个cflow不具备的优点：calltree可以直接生成dot文件，然后借助graphviz将其转换成图片。
而cflow只能输出ASCII的调用关系图，不借助中间工具不能转成dot。


