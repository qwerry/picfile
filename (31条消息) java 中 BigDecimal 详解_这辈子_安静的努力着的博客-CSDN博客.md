# (31条消息) java 中 BigDecimal 详解_这辈子_安静的努力着的博客-CSDN博客
首先，学习一个东西，我们都必须要带着问题去学，这边我分为 【为什么？】【是什么？】【怎么用？】

【为什么要用[BigDecimal](https://so.csdn.net/so/search?q=BigDecimal&spm=1001.2101.3001.7020)？】

首先，我们先看一下，下面这个现象

![](https://img-blog.csdnimg.cn/20190404154343440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODY4NDEy,size_16,color_FFFFFF,t_70)

那为什么会出现这种情况呢？

因为不论是float 还是double都是[浮点数](https://so.csdn.net/so/search?q=%E6%B5%AE%E7%82%B9%E6%95%B0&spm=1001.2101.3001.7020)，而计算机是二进制的，浮点数会失去一定的精确度。

注:根本原因是:十进制值通常没有完全相同的二进制表示形式;十进制数的二进制表示形式可能不精确。只能无限接近于那个值

但是，在项目中，我们不可能让这种情况出现，特别是金融项目，因为涉及金额的计算都必须十分精确，你想想，如果你的支付宝账户余额显示193.99999999999998，那是一种怎么样的体验？

【BigDecimal是什么？】

**1、简介**  
Java在java.math包中提供的API类BigDecimal，用来对超过16位有效位的数进行精确的运算。双精度浮点型变量double可以处理16位有效数。在实际应用中，需要对更大或者更小的数进行运算和处理。float和double只能用来做科学计算或者是工程计算，在商业计算中要用java.math.BigDecimal。BigDecimal所创建的是对象，我们不能使用传统的+、-、*、/等算术运算符直接对其对象进行数学运算，而必须调用其相对应的方法。方法中的参数也必须是BigDecimal的对象。构造器是类的特殊方法，专门用来创建对象，特别是带有参数的对象。

  
**2、构造器描述**   
BigDecimal(int)       创建一个具有参数所指定整数值的对象。   
BigDecimal(double) 创建一个具有参数所指定双精度值的对象。 //不推荐使用  
BigDecimal(long)    创建一个具有参数所指定长整数值的对象。   
BigDecimal(String) 创建一个具有参数所指定以字符串表示的数值的对象。//推荐使用

**3、方法描述**   
add(BigDecimal)        BigDecimal对象中的值相加，然后返回这个对象。   
subtract(BigDecimal) BigDecimal对象中的值相减，然后返回这个对象。   
multiply(BigDecimal)  BigDecimal对象中的值相乘，然后返回这个对象。   
divide(BigDecimal)     BigDecimal对象中的值相除，然后返回这个对象。   
toString()                将BigDecimal对象的数值转换成字符串。   
doubleValue()          将BigDecimal对象中的值以双精度数返回。   
floatValue()             将BigDecimal对象中的值以单精度数返回。   
longValue()             将BigDecimal对象中的值以长整数返回。   
intValue()               将BigDecimal对象中的值以整数返回。

特别说明一下，为什么BigDecimal(double)  不推荐使用，

![](https://img-blog.csdnimg.cn/20190404163436402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODY4NDEy,size_16,color_FFFFFF,t_70)

看上面代码运行结果，你就应该知道为什么不推荐使用了，因为用这种方式也会导致计算有问题，

为什么会出现这种情况呢？

 JDK的描述：1、参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。

        2、另一方面，String 构造方法是完全可预知的：写入 newBigDecimal("0.1") 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言，**通常建议优先使用String构造方法**

**当double必须用作BigDecimal的源时，**请使用`Double.toString(double)``转成String，然后`使用String构造方法，或使用BigDecimal的静态方法valueOf，如下

![](https://img-blog.csdnimg.cn/20190404164126593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODY4NDEy,size_16,color_FFFFFF,t_70)

【怎么用？】

这边我就不多说什么了，直接上代码，都挺简单的，最基本的加减乘除，应该能看的懂

![](https://img-blog.csdnimg.cn/20190404155544872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODY4NDEy,size_16,color_FFFFFF,t_70)

这边特别提一下，如果进行除法运算的时候，结果不能整除，有余数，这个时候会报java.lang.ArithmeticException: 

，这边我们要避免这个错误产生，在进行除法运算的时候，针对可能出现的小数产生的计算，必须要多传两个参数

    divide(BigDecimal，保留小数点后几位小数，舍入模式)

舍入模式

ROUND_CEILING    //向正无穷方向舍入
ROUND_DOWN    //向零方向舍入
ROUND_FLOOR    //向负无穷方向舍入
ROUND\_HALF\_DOWN    //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向下舍入, 例如1.55 保留一位小数结果为1.5
ROUND\_HALF\_EVEN    //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，如果保留位数是奇数，使用ROUND\_HALF\_UP，如果是偶数，使用ROUND\_HALF\_DOWN
ROUND\_HALF\_UP    //向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向上舍入, 1.55保留一位小数结果为1.6,也就是我们常说的“四舍五入”
ROUND_UNNECESSARY    //计算结果是精确的，不需要舍入模式
ROUND_UP    //向远离0的方向舍入

![](https://img-blog.csdnimg.cn/20190404165357710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODY4NDEy,size_16,color_FFFFFF,t_70)

需要对BigDecimal进行截断和四舍五入可用setScale方法，例：

![](https://img-blog.csdnimg.cn/20190404171013984.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODY4NDEy,size_16,color_FFFFFF,t_70)

参考博客连接：

[https://www.cnblogs.com/LeoBoy/p/6056394.html](https://www.cnblogs.com/LeoBoy/p/6056394.html)

[https://www.cnblogs.com/linjiqin/p/3413894.html](https://www.cnblogs.com/linjiqin/p/3413894.html)