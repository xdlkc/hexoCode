---
layout: 
title: java的class文件结构
date: 2019-11-18 16:07:10
tags:
categories: Java
---

(<深入分析Java Web技术内幕>笔记)
java语言在宣传时打出的名号就是"一次编译,到处运行", 也就是支持跨平台运行,其实这是java文件编译成class文件来实现的,那么class文件是怎么生成的?clss文件又有哪些内容?本篇笔记就是记录相关的学习内容的.

（历史文章归档）<!--more-->

## JVM指令集
先来一段几乎每个人入门一门语言都会写的代码:

```java
public class ClassLearn {
    public static void main(String[] args) {
        System.out.println("hello world!");
    }
}
```
就是一个类有一个main方法,方法内部打印了"hello world!",我们先将其编译为class文件,再运行:

```bash
// oolong是一种汇编语言,可以编译class文件为.j
java  COM.sootNsmoke.oolong.Gnoloo ClassLearn.class
```
会生成一个ClassLearn.j文件,查看一下文件内容:
![](http://upload-images.jianshu.io/upload_images/1890983-b965f406dda0f787.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以"."符号开头表示是一个基本的属性项,就是一个java的基本概念(一个类,方法,属性,对象或者一个接口)

### 与类相关的指令
* `.source ClassLearn.java`表示代码的源文件是ClassLearn.java
* `.class public super ClassLearn`表示这是一个类且公有的类名是ClassLearn
* `.super java/lang/Object`表示这个类的父类是Object

与类相关的指令:

| 指令       | 参数             | 解释                                           |
| ---------- | ---------------- | ---------------------------------------------- |
| checkcast  | class            | 检验类型转换(ClassCastException)               |
| getfield   | class/field desc | 获取类的实例域,并将值压入栈顶                  |
| getstatic  | class/field desc | 获取类的静态域,并将值压入栈顶                  |
| instanceof | class            | 检查对象是否是类的实例,是1压入栈顶,否0压入栈顶 |
| new        | class            | 创建一个对象,将其引入值压入栈顶                |


### 方法定义
* `.method public <init> ()V` 表示是一个公有方法,没有参数,返回值是`void`,`<init>`表示是构造方法
* `.method public static main([Ljava/lang/String;)V`与上一条类似,`[`表示数组,`L`表示是一个类形式而不是基本数据类型(int,char等),凡是`L`最后都以`;`结尾,表示类结束

与方法相关的JVM指令:

| 指令            | 操作数              | 解释                                     |
| --------------- | ------------------- | ---------------------------------------- |
| invokeinterface | class/method desc n | 调用接口方法                             |
| invokespecial   | class/method desc   | 调用超类构造方法,实例初始化方法,私有方法 |
| invokestatic    | class/method desc   | 调用静态方法                             |
| invokevirtual   | class/method desc   | 调用实例方法                             |

### 类的属性的定义
数据类型:

| 数据类型 | 表示方式 |
| -------- | -------- |
| 数组     | [        |
| 类       | L;       |
| byte     | B        |
| boolean  | Z        |
| char     | C        |
| double   | D        |
| float    | F        |
| int      | I        |
| long     | J        |
| short    | S        |
| void     | V        |

修饰属性:

| 修饰符    | 说明         |
| --------- | ------------ |
| public    | 公有         |
| private   | 私有         |
| protected | 子类和包可见 |
| static    | 静态         |
| final     | 不可修改     |
| volatile  | 弱引用       |
| transient | 临时属性     |

## class文件头的表示形式
看一看之前的代码字节码形式是怎么样的:

```java
java COM.sootNsmoke.oolong.DumpClass ClassLearn.class
```
输出:

```java
000000 cafebabe          magic = ca fe ba be
000004 0000              minor version = 0
000006 0034              major version = 52
000008 001d              29 constants
00000a 0a0006000f           1. Methodref class #6 name-and-type #15
00000f 0900100011           2. Fieldref class #16 name-and-type #17
000014 080012               3. String #18
000017 0a00130014           4. Methodref class #19 name-and-type #20
00001c 070015               5. Class name #21
00001f 070016               6. Class name #22
000022 010006               7. UTF length=6
000025 3c696e69743e                       <init>
00002b 010003               8. UTF length=3
00002e 282956                             ()V
000031 010004               9. UTF length=4
000034 436f6465                           Code
000038 01000f               10. UTF length=15
00003b 4c696e654e756d6265725461626c65     LineNumberTable
00004a 010004               11. UTF length=4
00004d 6d61696e                           main
000051 010016               12. UTF length=22
000054 285b4c6a6176612f6c616e672f537472   ([Ljava/lang/Str
000064 696e673b2956                       ing;)V
00006a 01000a               13. UTF length=10
00006d 536f7572636546696c65               SourceFile
000077 01000f               14. UTF length=15
00007a 436c6173734c6561726e2e6a617661     ClassLearn.java
000089 0c00070008           15. NameAndType name #7 descriptor #8
00008e 070017               16. Class name #23
000091 0c00180019           17. NameAndType name #24 descriptor #25
000096 01000c               18. UTF length=12
000099 68656c6c6f20776f726c6421           hello world!
0000a5 07001a               19. Class name #26
0000a8 0c001b001c           20. NameAndType name #27 descriptor #28
0000ad 01000a               21. UTF length=10
0000b0 436c6173734c6561726e               ClassLearn
0000ba 010010               22. UTF length=16
0000bd 6a6176612f6c616e672f4f626a656374   java/lang/Object
0000cd 010010               23. UTF length=16
0000d0 6a6176612f6c616e672f53797374656d   java/lang/System
0000e0 010003               24. UTF length=3
0000e3 6f7574                             out
0000e6 010015               25. UTF length=21
0000e9 4c6a6176612f696f2f5072696e745374   Ljava/io/PrintSt
0000f9 7265616d3b                         ream;
0000fe 010013               26. UTF length=19
000101 6a6176612f696f2f5072696e74537472   java/io/PrintStr
000111 65616d                             eam
000114 010007               27. UTF length=7
000117 7072696e746c6e                     println
00011e 010015               28. UTF length=21
000121 284c6a6176612f6c616e672f53747269   (Ljava/lang/Stri
000131 6e673b2956                         ng;)V
000136 0021              access_flags = 33
000138 0005              this = #5
00013a 0006              super = #6
00013c 0000              0 interfaces
00013e 0000              0 fields
000140 0002              2 methods
                         Method 0:
000142 0001                 access flags = 1
000144 0007                 name = #7<<init>>
000146 0008                 descriptor = #8<()V>
000148 0001                 1 field/method attributes:
                            field/method attribute 0
00014a 0009                    name = #9<Code>
00014c 0000001d                length = 29
000150 0001                    max stack: 1
000152 0001                    max locals: 1
000154 00000005                code length: 5
000158 2a                      0 aload_0
000159 b70001                  1 invokespecial #1
00015c b1                      4 return
00015d 0000                    0 exception table entries:
00015f 0001                    1 code attributes:
                               code attribute 0:
000161 000a                       name = #10<LineNumberTable>
000163 00000006                   length = 6
                                  Line number table:
000167 0001                       length = 1
000169 00000008                      start pc: 0 line number: 8
                         Method 1:
00016d 0009                 access flags = 9
00016f 000b                 name = #11<main>
000171 000c                 descriptor = #12<([Ljava/lang/String;)V>
000173 0001                 1 field/method attributes:
                            field/method attribute 0
000175 0009                    name = #9<Code>
000177 00000025                length = 37
00017b 0002                    max stack: 2
00017d 0001                    max locals: 1
00017f 00000009                code length: 9
000183 b20002                  0 getstatic #2
000186 1203                    3 ldc #3
000188 b60004                  5 invokevirtual #4
00018b b1                      8 return
00018c 0000                    0 exception table entries:
00018e 0001                    1 code attributes:
                               code attribute 0:
000190 000a                       name = #10<LineNumberTable>
000192 0000000a                   length = 10
                                  Line number table:
000196 0002                       length = 2
000198 0000000a                      start pc: 0 line number: 10
00019c 0008000b                      start pc: 8 line number: 11
0001a0 0001              1 classfile attributes
                         Attribute 0:
0001a2 000d                 name = #13<SourceFile>
0001a4 00000002             length = 2
0001a8 000e                 sourcefile index = #14
Done.

```
看起来特别长,但是真正需要理解的并不是很多,先看头部信息:

```java
000000 cafebabe          magic = ca fe ba be
000004 0000              minor version = 0
000006 0034              major version = 52
```
第一行是一个标识符,表示是一个标准的class文件,由一个32位无符号整数表示,`cafebabe`是其16进制表示形式,如果不是这个,JVM就不认为这是一个class文件,也就不会加载,后面两个字节表示说最大版本和最小版本的范围,从最初的Java到Java8是45.3~53.0,JVM在加载时也会检查这个是否符合条件

## 常量池
根据输出显示,共有29个常量,但是看显示只有1~28,是因为0是保留常量,每个常量都由3个字节描述:
![](http://upload-images.jianshu.io/upload_images/1890983-e007abd5d081112a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第一个表示这个常量是什么类型的,JVM中定义了12种类型常量,每个都会对应一个数值
![](http://upload-images.jianshu.io/upload_images/1890983-16ae4e70fdd98b81.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/1890983-fe990fcd019fb8c1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来分别说明几个较重要的

### UTF8常量
字符编码格式,存储多个字节长度的字符串值(类名,方法名等)
![](http://upload-images.jianshu.io/upload_images/1890983-06efaeb87d03d0cb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Fieldref,Methodref常量
描述Class中的属性项和方法的
![](http://upload-images.jianshu.io/upload_images/1890983-9e07d353290d84fd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第一个字节09代表是Fieldref类型,10则代表Methodref类型;后面两个字节代表属于哪个类;最后两个字表示常量的name和type

### Class常量
表示该类的名称,会指向另一个UTF8类型的常量(之前提到了UTF8就是表示类名和方法名等的字符串)
![](http://upload-images.jianshu.io/upload_images/1890983-3999ff561b51620a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
利用之前的输出

```java
00008e 070017               16. Class name #23
```
第一个字节07就代表该常量是Class类型,后面两个字节0017指向第23个常量,而第23个常量是:

```java
0000cd 010010               23. UTF length=16
0000d0 6a6176612f6c616e672f53797374656d   java/lang/System
```
就是一个UTF8常量,存储的是`java/lang/System`,也就是类名

### NameAndType常量
为了表示`Methodref`和`Fieldref`的名称和类型描述做进一步说明存在的,名称也是通常由`UTF8`常量描述,类型也由`UTF8`描述
![](http://upload-images.jianshu.io/upload_images/1890983-b2393039979543bc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
0007指向第7个常量,表示`Methodref`或`Fieldref`的名称,而0008则表示`Methodref`的返回类型或`Fieldref`参数类型

##类信息
常量列表后面就是类本身的信息描述了,如访问控制,名称,类型,是否有父类,是否实现了接口等
```java
000136 0021              access_flags = 33
000138 0005              this = #5
00013a 0006              super = #6
00013c 0000              0 interfaces
```
![](http://upload-images.jianshu.io/upload_images/1890983-75cd28215401e710.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
两个字节只使用了5bit,`ACC_PUBLIC`代表类是否是public类(是为1,否为0),`ACC_FINAL`代表是否是final类,`ACC_SUPER`代表是否存在父类(除了Obejct其他类都有父类),`ACC_INTERFACE`代表是否是接口类,`ACC_ABSTRACT`代表是否是抽象类;后面两个字节0005代表该类名称,指向第5个常量;后面的0006代表父类的名称,指向第6个常量;0000代表没有实现接口类

## Fields和Methods定义

```java
00013e 0000              0 fields
000140 0002              2 methods
                         Method 0:
000142 0001                 access flags = 1
000144 0007                 name = #7<<init>>
000146 0008                 descriptor = #8<()V>
000148 0001                 1 field/method attributes:
                            field/method attribute 0
00014a 0009                    name = #9<Code>
00014c 0000001d                length = 29
000150 0001                    max stack: 1
000152 0001                    max locals: 1
000154 00000005                code length: 5
000158 2a                      0 aload_0
000159 b70001                  1 invokespecial #1
00015c b1                      4 return
00015d 0000                    0 exception table entries:
00015f 0001                    1 code attributes:
                               code attribute 0:
000161 000a                       name = #10<LineNumberTable>
000163 00000006                   length = 6
                                  Line number table:
000167 0001                       length = 1
000169 00000008                      start pc: 0 line number: 8

```
Method1与Method0类似,就不粘过来了
前4个字节表示该类有几个属性,几个方法,后面就是每个属性和类的具体定义了
方法当然也有访问控制,也由两个字节定义:
![](http://upload-images.jianshu.io/upload_images/1890983-b95ced1b692abbe8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
与类描述基本一致,就不列举说明了

接下来4个字节:

```java
000144 0007                 name = #7<<init>>
000146 0008                 descriptor = #8<()V>
```
定义了方法的名称和类型描述(NameAndType)

接下来是具体的代码实现的定义了,包括代码长度(最长64K字节长度)

接下来:

```java
000150 0001                    max stack: 1
000152 0001                    max locals: 1
```
定义了该方法使用的栈的深度及本地常量的最大个数,会在JVM类加载做检查,超过这两个值,会拒绝加载

接下来:

```java
000154 00000005                code length: 5
000158 2a                      0 aload_0
000159 b70001                  1 invokespecial #1
00015c b1                      4 return
```
表示这个方法的代码对应的JVM指令,00000005代表命令有5个字节

接下来:

```java
00015d 0000                    0 exception table entries:
```
定义了使用的异常,0000代表没有定义抛出的异常

接下来是方法存在的一些代码属性的描述,描述的是代码本身的一些额外信息

```java
00015f 0001                    1 code attributes:
                               code attribute 0:
000161 000a                       name = #10<LineNumberTable>
000163 00000006                   length = 6
                                  Line number table:
000167 0001                       length = 1
000169 00000008                      start pc: 0 line number: 8
```
0001代表只有一行对应关系,指向0000和0008,0000指向运行时的行指针,0008指向实际源代码的行数,每个都是两个字节,所以可知Java代码最大行数到65535,超过class就没办法表示出来了

## Javap生成class文件
之前所有介绍的内容都是靠oolong生成的class文件的二进制表示,还可以通过JDK自带的javap来生成class文件,而且更易理解:

```java
javap -verbose ClassLearn >  ClassLearn.txt
```
查看输出:

```java

  1 Classfile /Users/lkc/Desktop/Code/Learn/out/production/TechCollect/ClassLearn.class
  2   Last modified 2017-8-7; size 672 bytes
  3   MD5 checksum 733971818278fd4709847bba1a9720bd
  4   Compiled from "ClassLearn.java"
  5 public class ClassLearn
  6   minor version: 0
  7   major version: 52
  8   flags: ACC_PUBLIC, ACC_SUPER
  9 Constant pool:
 10    #1 = Methodref          #6.#26         // java/lang/Object."<init>":()V
 11    #2 = String             #27            // hello,world!
 12    #3 = Fieldref           #28.#29        // java/lang/System.out:Ljava/io/PrintStream;
 13    #4 = Methodref          #30.#31        // java/io/PrintStream.println:(Ljava/lang/String;)V
 14    #5 = Class              #32            // ClassLearn
 15    #6 = Class              #33            // java/lang/Object
 16    #7 = Utf8               <init>
 17    #8 = Utf8               ()V
 18    #9 = Utf8               Code
 19   #10 = Utf8               LineNumberTable
 20   #11 = Utf8               LocalVariableTable
 21   #12 = Utf8               this
 22   #13 = Utf8               LClassLearn;
 23   #14 = Utf8               main
 24   #15 = Utf8               ([Ljava/lang/String;)V
 25   #16 = Utf8               i
 26   #17 = Utf8               I
 27   #18 = Utf8               args
 28   #19 = Utf8               [Ljava/lang/String;
 29   #20 = Utf8               str
 30   #21 = Utf8               Ljava/lang/String;
 31   #22 = Utf8               StackMapTable
 32   #23 = Class              #34            // java/lang/String
 33   #24 = Utf8               SourceFile
 34   #25 = Utf8               ClassLearn.java
 35   #26 = NameAndType        #7:#8          // "<init>":()V
 36   #27 = Utf8               hello,world!
 37   #28 = Class              #35            // java/lang/System
 38   #29 = NameAndType        #36:#37        // out:Ljava/io/PrintStream;
 39   #30 = Class              #38            // java/io/PrintStream
 40   #31 = NameAndType        #39:#40        // println:(Ljava/lang/String;)V
 41   #32 = Utf8               ClassLearn
 42   #33 = Utf8               java/lang/Object
  43   #34 = Utf8               java/lang/String
 44   #35 = Utf8               java/lang/System
 45   #36 = Utf8               out
 46   #37 = Utf8               Ljava/io/PrintStream;
 47   #38 = Utf8               java/io/PrintStream
 48   #39 = Utf8               println
 49   #40 = Utf8               (Ljava/lang/String;)V
 50 {
 51   public ClassLearn();
 52     descriptor: ()V
 53     flags: ACC_PUBLIC
 54     Code:
 55       stack=1, locals=1, args_size=1
 56          0: aload_0
 57          1: invokespecial #1                  // Method java/lang/Object."<init>":()V
 58          4: return
 59       LineNumberTable:
 60         line 8: 0
 61       LocalVariableTable:
 62         Start  Length  Slot  Name   Signature
 63             0       5     0  this   LClassLearn;
 64
 65   public static void main(java.lang.String[]);
 66     descriptor: ([Ljava/lang/String;)V
 67     flags: ACC_PUBLIC, ACC_STATIC
 68     Code:
 69       stack=2, locals=3, args_size=1
 70          0: ldc           #2                  // String hello,world!
 71          2: astore_1
 72          3: iconst_0
 73          4: istore_2
 74          5: iload_2
 75          6: iconst_3
 76          7: if_icmpge     23
 77         10: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
 78         13: aload_1
 79         14: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
 80         17: iinc          2, 1
 81         20: goto          5
 82         23: return
 83       LineNumberTable:
 84         line 10: 0
  85         line 11: 3
 86         line 12: 10
 87         line 11: 17
 88         line 14: 23
 89       LocalVariableTable:
 90         Start  Length  Slot  Name   Signature
 91             5      18     2     i   I
 92             0      24     0  args   [Ljava/lang/String;
 93             3      21     1   str   Ljava/lang/String;
 94       StackMapTable: number_of_entries = 2
 95         frame_type = 253 /* append */
 96           offset_delta = 5
 97           locals = [ class java/lang/String, int ]
 98         frame_type = 250 /* chop */
 99           offset_delta = 17
100 }
101 SourceFile: "ClassLearn.java"
```
三个地方LineNumberTable,LocalVariableTable,StackMapTable

### LineNumberTable
LineNumberTable下面有多个line a:b,每个line表示一行,a表示在类文件的第几行,b表示在JVM指令中的pc偏移量
![](http://upload-images.jianshu.io/upload_images/1890983-dfad25390c85da30.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
拿line 14:23举例,14就对应return这一句,23代表偏移地址,也就是指:

```java
82         23: return
```
可以发现是一致的

### LocalVariableTable

```java
 89       LocalVariableTable:
 90         Start  Length  Slot  Name   Signature
 91             5      18     2     i   I
 92             0      24     0  args   [Ljava/lang/String;
 93             3      21     1   str   Ljava/lang/String;
```
可以很容易看出,LocalVariableTable由五部分组成,Start和Length表示变量有效作用域的偏移地址;Start表示变量被赋值到某个Slot中的指令的下一条指令的偏移地址;Length表示变量作用域总共占用的指令数对应的偏移量,所以作用域就是[Start,Start+Length];Slot表示该变量占用的Slot编号;Name表示该变量的名称;Signature表示该变量的类型
拿出一个来举例:

```java
 90         Start  Length  Slot  Name   Signature
 91             5      18     2     i   I
```
这里就表示变量"i"的类型是int,作用域是[5,23],也就是这一段:

```java
 74          5: iload_2
 75          6: iconst_3
 76          7: if_icmpge     23
 77         10: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
 78         13: aload_1
 79         14: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
 80         17: iinc          2, 1
 81         20: goto          5
 82         23: return
```
与实际代码比较,也就是整个for循环体直到方法结束,所以符合我们预期;还需要提一句的是其实Slot是可以出现重复的,只要实际代码中作用域不重合Slot的编号是可以复用的.

### StackMapTable
这里在我阅读本书时发现没有StackMapTable,因此特地查了下资料
这里是由Java6引入的一个叫做栈图的概念,Java7中开始强制使用,实例说明:

```java
 94       StackMapTable: number_of_entries = 2
 95         frame_type = 253 /* append */
 96           offset_delta = 5
 97           locals = [ class java/lang/String, int ]
 98         frame_type = 250 /* chop */
 99           offset_delta = 17
```
`number_of_entries`代表frame的个数,这里也就是2个,分别是`frame_type = 253 /* append */` 和 `frame_type = 250 /* chop */`, 这里注释的`append`就代表栈中局部变量或类增加1个,`chop`相反代表减少一个,还有一个`same`代表没有变化;offset_delta代表每个frame的增量偏移量,第一个的偏移量就是offset_delta本身,其他帧的偏移量计算方法是前一帧的偏移量加上此帧的offset_delta+1;目前仅了解这么多,想详细了解StackMapTable相关或者其他的class文件信息可以参照< Java虚拟机规范 se7 >