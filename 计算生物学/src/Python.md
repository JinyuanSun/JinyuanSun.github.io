## Python基础与实例

### Python3

Python（念作：‘拍僧’，我曾经有个同学和我说他他在学 ‘非统’ 😂），可以说是这个时代最流行的语言了，风格及其接近伪代码，现在很多大学理工科专业的程序语言设计课程都从VB、C++换成了Python。

Python之所以受欢迎一方面是简单代码少，可读性强（缩进和空格非常严格），更是因为强大的开源社区提供了数以万计包`package`只需要简单的`import`就可以调用许多功能强大的`函数`。所谓`函数`其实就是`A->B`，我们输入A，得到B。

这一份教程不会从变量开始一点点介绍Python，我曾经系统学习过，但是很快就忘光了，还是在解决问题的过程中学习的比较扎实，我会通过一个个实用的例子讲解python的具体用法。我们会用到它处理文件，做简单的计算，写一些简单的流程，也会涉及一些机器学习。

### 使用Python处理fasta文件

#### 提取fasta

> 问题:我们有一个含有几十万条蛋白序列的巨大的fasta文件，现在要从中分离出给定的序列(好希望有一根智能柱子)。
>
> 方法:把fasta文件做成序列名称和序列对应的结构。想象成抗体和抗原。

`字符串`是Python中的一种数据类型，一切都可以被看作字符串，字符串不可变，就是符号串在一起。

`字典`是Python中一种可变容器模型，且可存储任意类型对象。

`字典`的每个`键值对`( **key=>value**) 用冒号` : `分割，每个键值对之间用逗号 `,` 分割，整个`字典`包括在花括号 `{}` 中 ,格式如下所示：

```python
d = {key1 : value1, key2 : value2 }
```

其中`键`是不可以重复的，对同一个`键`多次赋予不同的`值`，只会保留最后一个。具体可参见[字典](https://www.runoob.com/python/python-dictionary.html)。

`sys`是一个python内置的包，用于终端的交互。

`#`开头的行在Python代码中是注释，解释器不会运行这一行，只是给你看的。

```python
import sys

#我们使用 def functionname(): 定义一个函数，方便调用。
def fasta_2_dic(fasta):
    #创建一个名叫dic的空字典，我们将要建立序列名称和序列的一一对应关系。
    dic = {}
    #fasta是被传入函数的变量名称，是要被处理的fasta文件，此处我们一行一行的迭代阅读这个巨大的fasta文件，line代表了每一行的字符串
    for line in fasta:
      #if 用于判断条件，.startswith(X)的意思是字符串以X开始
        if line.startswith(">"):
          #根据fasta的规定，>开始的行就是名称，我们去掉行里的"\n"，这代表着换行
            head = line.replace("\n","")
            #向空字典dic这个容器中放入一个键值对，这可能比较抽象，可以想象成在固体颗粒上固定好了抗体准备吸附抗原了
            dic[head] = ''
        else:#esle表示了和它缩进相同的if的反面，这里就是字符串不以>开始，那么这一行应该是序列。
          #去掉末尾的换行符
            seq = line.replace("\n","")
            #往键上赋值，+=代表了在之前的值后面续接。可以想象成这个抗原是个寡聚蛋白，抗体抓住抗原的一个亚基后，其他的亚基自己聚合上来。
            dic[head] += seq
    #循环结束以后输出dic，这个dic包含了一一对应的序列名与序列
    return dic
#使用try进行异常控制  
try:
  #sys.argv[2]代表了python3后面第三个空格后输入的内容，此处应该是原始fasta文件
  #open()用来打开文件，如果不存在就创建，“w+”表示覆盖原文件，创建一个空的，“a+”表示打开文件，光标移到最后一行
    seqdump = open(sys.argv[2])
    dic = fasta_2_dic(seqdump)
    ofile = open(sys.argv[3],"w+")
    ofile = open(sys.argv[3],"a+")
    lst = open(sys.argv[1])
    #迭代待分离的列表文件
    for x in lst:
    	h = x.replace("\n","")
    	try:
        #根据待分离文件中的名称（抗体），在字典中中找对应的序列（抗原）
        	seq = dic[h]
          #对符合条件的序列输出打印到文件ofile
        	print(x+seq+"\n",file=ofile)
          #如果没有该名称
    	except KeyError:
        #显示该名称不存在于原始文件中
        	print(h+" not found in "+sys.argv[2])
#输出使用帮助
except IndexError:
    print("Usage: python3 split.py <list> <fasta> <outfile>")
```



#### 过滤过长或过短的序列

> 问题：我们在构建多序列比对的时候，为了比对的可靠性，需要过滤特别长或者特别短的序列。
>
> 方法：把fasta文件做成序列名称和序列对应的字典，求所有序列平均长度，输出长度符合范围的序列。

`列表`是Python中一种序列数据结构，是一种常见的数据类型，其中元素有自己从前到后，从`0`开始的索引（index），放在中括号内：

```python
l = [element1,element2,element3]
```

`numpy`是一个广泛使用的用于一些计算的包，在求平均数等方面非常好用。

```python
import sys
import numpy as np

#我们使用 def functionname(): 定义一个函数，方便调用。
def fasta_2_dic(fasta):
    #创建一个名叫dic的空字典，我们将要建立序列名称和序列的一一对应关系。
    dic = {}
    #fasta是被传入函数的变量名称，是要被处理的fasta文件，此处我们一行一行的迭代阅读这个巨大的fasta文件，line代表了每一行的字符串
    for line in fasta:
      #if 用于判断条件，.startswith(X)的意思是字符串以X开始
        if line.startswith(">"):
          #根据fasta的规定，>开始的行就是名称，我们去掉行里的"\n"，这代表着换行
            head = line.replace("\n","")
            #向空字典dic这个容器中放入一个键值对，这可能比较抽象，可以想象成在固体颗粒上固定好了抗体准备吸附抗原了
            dic[head] = ''
        else:#esle表示了和它缩进相同的if的反面，这里就是字符串不以>开始，那么这一行应该是序列。
          #去掉末尾的换行符
            seq = line.replace("\n","")
            #往键上赋值，+=代表了在之前的值后面续接。可以想象成这个抗原是个寡聚蛋白，抗体抓住抗原的一个亚基后，其他的亚基自己聚合上来。
            dic[head] += seq
    #循环结束以后输出dic，这个dic包含了一一对应的序列名与序列
    return dic

def _count(dic):
    ofile = open(sys.argv[2],"w+")
    ofile = open(sys.argv[2],"a+")
    #空字典cdic
    cdic = {}
    #空列表clist
    clist = []
    #遍历字典dic中的每一个键key
    for key in dic:
      #len()返回字符串长度，列表元素数量，字典的键的个数，此处我们用dic[key]获得dic中储存的序列名对应的序列，并获得长度，将长度储存于变量seql中，seql应该代表一个整数，为长整型。
        seql = len(dic[key])
        #在字典cdic中储存序列名和序列长
        cdic[key] = seql
        #在clist中用append方法添加元素
        clist.append(seql)
    #此循环结束后，clist中储存了所有的序列长，cdic中储存了序列名和序列长的对应关系，dic中储存了序列名和序列的对应关系
    #用numpy的mean()方法求clist所有元素的平均数
    m = np.mean(clist)
    #迭代dic
    for key in dic:
      #如果某序列的长度与所有序列平均值的比值大于0.8，小于1.2，则认为长度适中。
        if 0.8 < cdic[key]/m < 1.2:
          #对符合条件的序列输出打印到文件ofile
            print(key+"\n"+dic[key]+"\n",file=ofile)

try:
    seqdump = open(sys.argv[1])
    dic = fasta_2_dic(seqdump)
    _count(dic)
except IndexError:
    print("Usage: python3 fliter.py <sequences.fasta> <out file>")
```


