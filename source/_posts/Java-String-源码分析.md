---
title: Java String 源码分析
categories: Java
tags:
  - Java
  - 源码
description: Java8新特性简单介绍
toc: true
author: NewGr8Player
date: 2019-08-26 01:35:45
updated: 2019-08-26 15:06:50
---

# 定义
```Java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {}

```

从定义可以看出
- `String` 被 `final` 修饰(不允许被继承)

实现了如下接口：

- `Serializable`（可以序列化和反序列化）
- `Comparable`（可以进行自定义的字符串比较）
- CharSequence（一个可读序列。此接口对许多不同种类的 char 序列提供统一的只读访问。StringBuilder 和StringBuffer也实现了这个接口）

# 属性

```Java
/** 用于存放字符串的数组 */
private final char value[];

```

> 由于被final修饰，故String对象一旦初始化后，其即为不可变（指其内容不能被修改，其引用还是可以指向其他的内容）。
> `String` 是基于 `char[]` 实现的
> 频繁拼接字符串建议使用StringBuffer、StringBuilder、StringJoiner等

# 构造方法

## 构造方法

### 无参构造器

```Java
public String() {
    this.value = "".value;
}

```

> 即 `String str = new String();` 时，变量`str`的值为空字符串。

### 一个字符串类参数构造
```Java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}

```

> 这个方法会产生两个字符串对象，分别为参数`original`与新创建的字符串对象。

### 使用字符数组构造
```Java
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}

```

**带有offset与count的重载**

```Java
public String(char value[], int offset, int count) {
    if (offset < 0) {
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset <= value.length) {
           this.value = "".value;
           return;
       }
   }
     // Note: offset or count might be near -1>>>1.
     if (offset > value.length - count) {
         throw new StringIndexOutOfBoundsException(offset + count);
    }
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}

```

### 使用字节数组构造

```Java
String(byte bytes[]);
String(byte bytes[], int offset, int length);

```

**需要制定字符集的构造方法**

```Java
String(byte bytes[], Charset charset);
String(byte bytes[], String charsetName);
String(byte bytes[], int offset, int length, Charset charset);
String(byte bytes[], int offset, int length, String charsetName);

```

### 使用`StringBuffer`或者`StringBuider`构造

> 很少用，因为`StringBuffer::toString()`或者`StringBuider::toString()`直接就可以生成一个String对象

### 特殊的构造方法

```Java
/*
 * Package private constructor which shares value array for speed.
 * this constructor is always expected to be called with share==true.
 * a separate constructor is needed because we already have a public
 * String(char[]) constructor that makes a copy of the given char[].
 */
String(char[] value, boolean share) {
    // assert share : "unshared not supported";
    this.value = value;
}

```

> 这个方法由Java7提供，直接饮用value[]的地址，并且`share`只能为`true`
> 提供这个方法的原因：性能好，节约空间，安全

## 一般方法
### equals()方法

```Java
public boolean equals(Object anObject) {
    // 判断是否为同一对象
    if (this == anObject) {
        return true;
    }
    // 判断 anObject 是不是 String 类型
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        // 判断长度是否相等
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 循环比较每一个 char 值
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}

```

> 一般默认是比较是否是同一个对象，此处默认重写，比较是否比较的是内容相同的两个字符串
> 1. 比较是否是同一个对象
> 2. 比较是否是字符串对象
> 3. 循环比较每一位是否相同

### hashCode()方法

```Java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        // 数学公式：s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]*31^0
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}

```

> 常规hashCode方法

### substring()方法

```Java
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
   // 将原来的 char[] 中的值逐一复制到新的 String 中
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}

```

> Java 7 提供了一种新的实现方式牺牲一些性能，避免内存泄露。 Java 6 时，是同一个数组中操作。Java 7 是创建一个新的数组。

### 替换方法

> replaceFirst、replaceAll、replace区别：
> replace 的参数是 char 和 CharSequence，即可以支持字符的替换，也支持字符串的替换；
> replaceAll 和 replaceFirst 的参数是 regex，即基于规则表达式的替换（如果所用的参数据不是基于规则表达式的，则与replace()替换字符串的效果一样。）

```Java
/**
 * value 为存储当前字符串对象的字符串数组名
 * 优化了比较次数
 * @param oldChar 将被替换掉的字符
 * @param newChar 新字符
 * @return
 */
public String replace(char oldChar, char newChar) {
    //如果新字符和老字符相同，直接返回当前字符串对象
    if (oldChar != newChar) {
        int len = value.length; //获取当前字符串长度
        int i = -1;
        char[] val = value; //目的：为了防止 getfield（获取指定类的实例域，并将其值压入到栈顶）这个操作码的执行
        //目的：找出将被替换的字符(oldChar)所在字符串数组中第一个字符的下标位置
        while (++i < len) {
            // 当找到第一个将被替换的字符时，跳出循环；i 就记录了下标位置
            // 如果没有找到，i==len 将不会进入下面的，直接返回当前字符串
            if (val[i] == oldChar) {
                break;
            }
        }
        if (i < len) {
            //将第一个将被替换的字符(oldChar)之前的不需要被替换的字符赋值给新数组buf
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            // 从第一个将被替换的字符下标(i)开始，对字符进行替换操作
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    return this;
}

```

