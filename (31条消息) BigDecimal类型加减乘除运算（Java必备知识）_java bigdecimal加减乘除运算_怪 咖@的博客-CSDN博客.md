# (31条消息) BigDecimal类型加减乘除运算（Java必备知识）_java bigdecimal加减乘除运算_怪 咖@的博客-CSDN博客
在现实开发当中经常会遇到这种计算，这里特此整理一下为方便以后学习，希望能帮助到其他的萌新。  

1、为什么要用[BigDecimal](https://so.csdn.net/so/search?q=BigDecimal&spm=1001.2101.3001.7020)计算？
------------------------------------------------------------------------------------------

因为 float, double等浮点的存储和操作（比如：相加，相减…）存在误差(7.22f - 7.0f = 0.21999979 而不是 0.22)。

![](https://img-blog.csdnimg.cn/20210330004627893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjEwMTgzOQ==,size_16,color_FFFFFF,t_70)

2、浮点计算误差产生的原因
-------------

将十进制数转为二进制，在计算机运行中本就存在误差  
来看一个例子：将十进制的0.2转化为二进制，按照乘二取整法

> 0.2 * 2 = 0.4 0  
> 0.4 * 2 = 0.8 0  
> 0.8 * 2 = 1.6 1  
> 0.6 * 2 = 1.2 1  
> … … …  
> 0.2*2 = 0.4 0

很明显，已经进入了无限循环，受有效位数23位的影响，也就是说，存入计算机中的十进制数不是精准的。  
float的有效位数只有7位有效数字，如果一个大数和一个小数相加时，会产生很大的误差，因为尾数得截掉好多位。

3、bigdecimal的初始化
----------------

Bigdecimal的初始化时用尽量用String，假如传的是浮点类型，会丢失精度。阿里的开发规范当中也明确说明了。

这一点在BigDecimal类的构造方法注释中有说明。

```clike
	BigDecimal num1 = new BigDecimal(0.005);
	BigDecimal num2 = new BigDecimal(1000000);
	
    BigDecimal string1 = new BigDecimal("0.005");
    BigDecimal string2 = new BigDecimal("1000000");
    
    System.out.println(num1);
    System.out.println(num2);
    System.out.println(string1);
    System.out.println(string2);

```

**输出结果**:

![](https://img-blog.csdnimg.cn/20210330234059625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjEwMTgzOQ==,size_16,color_FFFFFF,t_70)

4、bigdecimal的加减乘除
-----------------

```clike
	BigDecimal num1 = new BigDecimal("11.111");
    BigDecimal num2 = new BigDecimal("1");
    BigDecimal num3 = new BigDecimal("-11.111");
    
    
    BigDecimal result1 = num1.add(num2);
    System.out.println("num1 + num2 = " + result1);
    
    
    BigDecimal result2 = num1.subtract(num2);
    System.out.println("num1 - num2 = " + result2);

    
    BigDecimal result3 = num1.multiply(num2);
    System.out.println("num1 * num2 = " + result3);

    
    BigDecimal result5 = num1.divide(num2,20,BigDecimal.ROUND_HALF_UP);
    System.out.println("num1 / num2 = " + result5);
    
    
    BigDecimal result4 = num3.abs();
    System.out.println("num3的绝对值  = " + result4);

```

**输出结果**：

![](https://img-blog.csdnimg.cn/20210330233843431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjEwMTgzOQ==,size_16,color_FFFFFF,t_70)

5、除法divide()参数使用
----------------

使用除法函数在divide的时候要设置各种参数，要精确的小数位数和舍入模式，不然会出现报错

我们可以看到divide函数配置的参数如下  
![](https://img-blog.csdnimg.cn/20210330234520828.png)
  
BigDecimal divisor 除数， int scale 精确小数位， int roundingMode 舍入模式

6、八种舍入模式解释如下
------------

### 6.1、ROUND_UP

舍入远离零的舍入模式。

在丢弃非零部分之前始终增加数字(始终对非零舍弃部分前面的数字加1)。

注意，此舍入模式始终不会减少计算值的大小。

### 6.2、ROUND_DOWN

接近零的舍入模式。

在丢弃某部分之前始终不增加数字(从不对舍弃部分前面的数字加1，即截短)。

注意，此舍入模式始终不会增加计算值的大小。

### 6.3、ROUND_CEILING

接近正无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND_UP 相同;

如果为负，则舍入行为与 ROUND_DOWN 相同。

注意，此舍入模式始终不会减少计算值。

### 6.4、ROUND_FLOOR

接近负无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND_DOWN 相同;

如果为负，则舍入行为与 ROUND_UP 相同。

注意，此舍入模式始终不会增加计算值。

### 6.5、ROUND\_HALF\_UP

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。

如果舍弃部分 >= 0.5，则舍入行为与 ROUND\_UP 相同;否则舍入行为与 ROUND\_DOWN 相同。

注意，这是我们大多数人在小学时就学过的舍入模式(四舍五入)。

### 6.6、ROUND\_HALF\_DOWN

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为上舍入的舍入模式。

如果舍弃部分 \> 0.5，则舍入行为与 ROUND\_UP 相同;否则舍入行为与 ROUND\_DOWN 相同(五舍六入)。

### 6.7、ROUND\_HALF\_EVEN

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。

如果舍弃部分左边的数字为奇数，则舍入行为与 ROUND\_HALF\_UP 相同;

如果为偶数，则舍入行为与 ROUND\_HALF\_DOWN 相同。

注意，在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。

此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况。

如果前一位为奇数，则入位，否则舍去。

以下例子为保留小数点1位，那么这种舍入方式下的结果。

1.15>1.2 1.25>1.2

### 6.8、ROUND_UNNECESSARY

断言请求的操作具有精确的结果，因此不需要舍入。

如果对获得精确结果的操作指定此舍入模式，则抛出ArithmeticException。

**注意：一般无特殊情况下，我们都是用的ROUND\_HALF\_UP（四舍五入）**

7、保留小数位
-------

开发当中经常会遇到浮点保留两位小数，这里我们也可以利用BigDecimal 来进行保留。

```clike
BigDecimal bg = new BigDecimal("0.005");
double f1 = bg.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();

```

8、比较大小
------

等于：new BigDecimal(“123.123”).compareTo(new BigDecimal(“123.123”))==0 —> true  
小于：new BigDecimal(“123.122”).compareTo(new BigDecimal(“123.123”)) < 0 —> true就证明左边小于右边  
大于：new BigDecimal(“123.124”).compareTo(new  
BigDecimal(“123.123”)) > 0 —> true就证明左边大于右边

9、公共方法
------

以下是我在开发过程当中**两数相除求百分比**，写的一些简单的方法，感觉有用呢你可以直接拿去用。

```clike
	
	public static BigDecimal flotTwoDecimals(Double num) {
		if(!isBlankOrEmpty(num)) {
			BigDecimal bg = new BigDecimal(num.toString());
			Double f1 = bg.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();
		    return new BigDecimal(f1.toString());
		}
		return new BigDecimal(0);
	}
	
	
	public static BigDecimal integerDivideTwoDecimals(Integer divisor,Integer dividend) {
		if(!isBlankOrEmpty(divisor) && !isBlankOrEmpty(dividend)) {
			BigDecimal result = divisor==0||dividend==0?new BigDecimal(0):new BigDecimal(divisor*100)
						.divide(new BigDecimal(dividend), 2, BigDecimal.ROUND_HALF_UP);
			String resultString = result.toString();
			String substring = resultString.substring(resultString.indexOf(".")+1);
			if (substring.equals("00")) {
				String substring2 = resultString.substring(0, resultString.indexOf("."));
				return new BigDecimal(substring2);
			}
			return result;
		}
		return new BigDecimal(0);
	}
	
	
	public static BigDecimal flotDivideTwoDecimals(Double divisor,Double dividend) {
		if(!isBlankOrEmpty(divisor) && !isBlankOrEmpty(dividend)) {
			if (divisor==0 || dividend==0) {
				return new BigDecimal(0);
			}
			BigDecimal divisor1 = new BigDecimal(divisor.toString()).multiply(new BigDecimal(100));
			BigDecimal result = divisor1.divide(new BigDecimal(dividend.toString()), 2, BigDecimal.ROUND_HALF_UP);
			return result;
		}
		return new BigDecimal(0);
	}
	
	
    public static boolean isBlankOrEmpty(Object object) {
        if (null == object) {
            return true;
        }
        if (object.toString().equals("")) {
            return true;
        }
        return false;
    }

```