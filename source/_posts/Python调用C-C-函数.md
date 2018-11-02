---
title: Python调用C/C++函数
date: 2013-06-27 16:43:14
tags: [C++]
---
Python是一门功能强大的高级脚本语言，它的强大不仅表现在其自身的功能上，而且还表现在其良好的可扩展性上，正因如此，Python已经开始受到越来越多人的青睐，并且被屡屡成功地应用于各类大型软件系统的开发过程中。
与其它普通脚本语言有所不同，Python程序员可以借助Python语言提供的API，使用C或者C++来对Python进行功能性扩展，从而即可以利用Python方便灵活的语法和功能，又可以获得与C或者C++几乎相同的执行性能。执行速度慢是几乎所有脚本语言都具有的共性，也是倍受人们指责的一个重要因素，Python则通过与C语言的有机结合巧妙地解决了这一问题，从而使脚本语言的应用范围得到了很大扩展。
下面以一个简单的例子说明如何把一个C函数编译为共享库供python调用的：
```bash
//wrap.cpp
#include
using namespace std;

//目标函数
int fact(int n)
{
    if (n <= 1)
        return 1;
    else
        return n * fact(n - 1);
}

//python 包装
PyObject* wrap_fact(PyObject* self, PyObject* args)
{
      int n, result;

      if (! PyArg_ParseTuple(args, "i:fact", &n))  //把python的变量转换成c的变量
           return NULL;
      result = fact(n);
      return Py_BuildValue("i", result);//把c的返回值n转换成python的对象
}

//  方法列表
static PyMethodDef exampleMethods[] =
{
      {"fact", wrap_fact, METH_VARARGS, "Caculate N!"},
      {NULL, NULL}
};
//方法列表中的每项由四个部分组成：方法名、导出函数、参数传递方式和方法描述。
方法名是从Python解释器中调用该方法时所使用的名字。
参数传递方式则规定了Python向C函数传递参数的具体形式，可选的两种方式是METH_VARARGS和METH_KEYWORDS，
其中METH_VARARGS是参数传递的标准形式，它通过Python的元组在Python解释器和C函数之间传递参数，若采用METH_KEYWORD方式，则Python解释器和C函数之间将通过Python的字典类型在两者之间进行参数传递。

//初始化函数
PyMODINIT_FUNC initexample()
{
      PyObject* m;
      m = Py_InitModule("example", exampleMethods);
}
//所有的Python扩展模块都必须要有一个初始化函数，以便Python解释器能够对模块进行正确的初始化。Python解释器规定所有的初始化函数的函数名都必须以init开头，并加上模块的名字。对于模块example来说，则相应的初始化函数为
```
生成共享库.so供python调用：
```bash
g++ -fPIC -c -I /usr/include/python2.3 wrap.cpp
g++ -shared -o example.so wrap.o
```
测试一下：
test.py
```bash
#!/usr/bin/python
import sys
sys.path.append('...') //添加example.so所在的路径
import example
a = example.fact(4)
print a
输出：24
```