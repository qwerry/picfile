# (31条消息) java之BigDecimal的加减乘除_java bigdecimal加减乘除运算_乌龟壳的博客-CSDN博客
在小数之间加减乘除的时候，由于二进制不能精确表示小数，从而导致精度丢失。在实际开发中，这种情况是很糟糕的。为了解决这一情况，我们可以利用BgiDecimal。但是这中间还有些问题需要注意的。

**1、加减乘**

首先，我们来看一个实例

```null
BigDecimal num1 = new BigDecimal(3.0);BigDecimal num2 = new BigDecimal(2.1);System.out.println(num1.add(num2));
```

打印结果

![](https://img-blog.csdn.net/20171207152012221?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzAxNjA3Mjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

并不是我们期望的结果,从jdk参考手册，我们可以找到原因

1.  The results of this constructor can be somewhat unpredictable. One might assume that writing`new BigDecimal(0.1)` in Java creates a`BigDecimal` which is exactly equal to 0.1 (an unscaled value of 1, with a scale of 1), but it is actually equal to 0.1000000000000000055511151231257827021181583404541015625. This is because 0.1 cannot be represented exactly as a`double` (or, for that matter, as a binary fraction of any finite length). Thus, the value that is being passed_in_ to the constructor is not exactly equal to 0.1, appearances notwithstanding.
2.  The `String` constructor, on the other hand, is perfectly predictable: writing`new BigDecimal("0.1")` creates a`BigDecimal` which is _exactly_ equal to 0.1, as one would expect. Therefore, it is generally recommended that theString constructor be used in preference to this one.
3.  When a `double` must be used as a source for a `BigDecimal`, note that this constructor provides an exact conversion; it does not give the same result as converting the`double` to a`String` using the `Double.toString(double)` method and then using the`BigDecimal(String)` constructor. To get that result, use the`static` `valueOf(double)` method. 

  

我们再来看另外一个例子

```null
BigDecimal num1 = BigDecimal.valueOf(3.0);BigDecimal num2 = BigDecimal.valueOf(2.1);System.out.println(num1.add(num2));
```

打印输出

![](https://img-blog.csdn.net/20171207152210282?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzAxNjA3Mjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

这次是我们期望的结果。

所以，我创建BigDecimal对象的时候，最好通过静态方法valueOf(double)

当然，我们也可以通过另外一种方式。先将double值转为字符串，再通过Bigdecimal的构造函数创建对象

```null
BigDecimal num1 = new BigDecimal(Double.toString(3.0));BigDecimal num2 = new BigDecimal(Double.toString(2.1));System.out.println("add->"+num1.add(num2));System.out.println("subtract->"+num1.subtract(num2));System.out.println("multiply->"+num1.multiply(num2));
```

打印输出

![](https://img-blog.csdn.net/20171207152312982?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzAxNjA3Mjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

**2.除**

jdk提供了几种小数之间相除的方法。

```null
BigDecimal num1 = new BigDecimal(3.0);BigDecimal num2 = new BigDecimal(2.1);System.out.println("divide->"+num1.divide(num2));
```

直接运行，奇怪的发现出现异常了

![](https://img-blog.csdn.net/20171207155627326?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzAxNjA3Mjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

从异常信息可以知道，相除的结果不能够用二进制精确表示，所以需要使用另外一个方法

BigDecimal num1 = new BigDecimal(3.0);

BigDecimal num2 = new BigDecimal(2.1);

System.out.println("divide->"+ num1.divide(num2,3,RoundingMode.FLOOR));

![](https://img-blog.csdn.net/20171207155731140?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzAxNjA3Mjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  

第二个参数表示，当结果是无限小数时，保留几位有效数字

第三个参数表示，如何保留有效数字，例如四舍五入、向下取等

<script>  
var \_hmt = \_hmt || \[\];  
(function() {  
  var hm = document.createElement("script");  
  hm.src = "https://hm.baidu.com/hm.js?5e96c091ce7bc89fd43194bbc14eaadc";  
  var s = document.getElementsByTagName("script")\[0\];   
  s.parentNode.insertBefore(hm, s);  
})();  
</script>