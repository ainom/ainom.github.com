---
layout: post
date: "2014-08-19 23:34:PM"
title: "Integer对象引发的思考"
description: "类内部提供了一个静态的IntegerCache，为了提高计算效率和速度，IntegerCache内部，缓存了[-128,127]区间内的256个整数对象，所以初始化时，如果值的空间在[-128,127]之间，会使用缓存，否则，重新定义对象。"
category: Java
tags: [Java]
---
{% include JB/setup %}
</br>最近在使用Integer时，用到==，遇到如下问题：

</br>Integer a=100,b=100;
</br>Integer x=200,y=200;

</br>a==b ? //true
</br>x==y ? //false

</br>解释是：
</br>类内部提供了一个静态的IntegerCache，为了提高计算效率和速度，IntegerCache内部，缓存了[-128,127]</br>区间内的256个整数对象，所以初始化时，如果值的空间在[-128,127]之间，会使用缓存，否则，重新定义对象。

</br>针对这个问题，小明同学有特意在这个解释的基础上做了一些深入的研究。以下是他的个人分析：
</br>源代码为：
</br>[java] view plaincopy在CODE上查看代码片派生到我的代码片
</br>public static void main(String[] args) {  
  
</br>        Integer a=100,b=100;  
</br>        Integer x=200,y=200;  
</br>        System.out.println(a==b);  
</br>        System.out.println(x==y);  
</br>    }  

</br>首先，我对这段代码执行后的.class进行了反编译，针对
</br>.class文件进行反编译，看到：
</br>[java] view plaincopy在CODE上查看代码片派生到我的代码片
</br>public static void main(String[] args)  
</br>  {  
</br>    Integer a = Integer.valueOf(100); Integer b = Integer.valueOf(100);  
</br>    Integer x = Integer.valueOf(200); Integer y = Integer.valueOf(200);  
</br>   System.out.println(a == b);  
</br>    System.out.println(x == y);  
</br>  }  
</br>原来关键在Integer.valueOf()这个方法，在jdk源码（我用的是jdk1.8.0，不过这个类应该没啥变化吧）中，该方法描述如下：
</br>[java] view plaincopy在CODE上查看代码片派生到我的代码片
</br>public static Integer valueOf(int i) {  
</br>if (i >= IntegerCache.low && i <= IntegerCache.high)  
</br>return IntegerCache.cache[i + (-IntegerCache.low)];  
</br>        return new Integer(i);  
</br>    }  

</br>Integer类中定义了静态的内部类IntegerCache，定义了不可变的low变量值为-128,代码：
</br>[java] view plaincopy在CODE上查看代码片派生到我的代码片
</br>static final int low = -128;  
</br>而high变量时在类加载时进行设置，首先去VM中获取关键字为“java.lang.Integer.IntegerCache.</br>high“的value值。默认为127，但如果虚拟机中缓存了该值，会取虚拟机中的值，但不会打过MAX_VALUE，详细代码如下：
</br>[java] view plaincopy在CODE上查看代码片派生到我的代码片
</br>// high value may be configured by property  
</br>           int h = 127;  
</br>           String integerCacheHighPropValue =  
</br>               sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");  
</br>           if (integerCacheHighPropValue != null) {  
</br>               try {  
</br>                   int i = parseInt(integerCacheHighPropValue);  
</br>                   i = Math.max(i, 127);  
</br>                   // Maximum array size is Integer.MAX_VALUE  
</br>                   h = Math.min(i, Integer.MAX_VALUE - (-low) -1);  
</br>               } catch( NumberFormatException nfe) {  
</br>                   // If the property cannot be parsed into an int, ignore it.  
</br>               }  
</br>           }  
</br>           high = h;  
</br>而这个数组cache的范围就是[low,high]，所以该初始化逻辑可以简单解释为：如果在缓存中则取指向缓存对象，否则定义新的对象，那么a,b(
</br>值为100)当然就都指向了相同的对象（是Integer哦）。