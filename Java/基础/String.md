# String

## 目录

* [不可变](#head1)
* [不可变的好处](#head2)
  * [缓存hashcode](#head3)
  * [安全性](#head4)
  * [线程安全](#head5)
  * [字符串常量池](#head6)
  * [String.intern()](#head7)
* [编译优化](#head8)

## <span id="head1">不可变

String被声明为final，它不可被继承。内部使用char数组存储数据，并且也被声明为final，说明该变量初始化后不能再指向别的引用。并且String内部并没有提供改变char数据的方法，因此可以保证String不可变。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

## <span id="head2">不可变的好处

### <span id="head3">缓存hashcode

String类的hash值在Java中是经常会被使用的，比如HashMap、HashSet等。String不可变特性也保证了了hash值得不变，所以只需计算一次，就可以将hash值缓存起来。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其他的键对象。

```java
/** Cache the hash code for the string */
private int hash; // Default to 0
```

### <span id="head4">安全性

字符串被广泛用作许多java类的参数，例如网络连接、打开文件等。譬如，数据库的用户名、密码都是以字符串的形式传入来获得数据库的连接，或者在socket编程中，主机名和端口都是以字符串的形式传入。因为字符串是不可变的，所以它的值是不可改变的，否则黑客们可以钻到空子，改变字符串指向的对象的值，造成安全漏洞，引起很严重的安全问题。

### <span id="head5">线程安全

String不可变性天生具备线程安全，可以在多个线程中安全的使用，同一个字符串实例可以被多个线程共享。

### <span id="head6">字符串常量池

正式因为字符串的不可变性，所以有了字符串常量池。如果一个Sting对象已经被创建过了，那么就会从String Pool中取得引用。

### <span id="head7">String.intern()

使用String.intern()可以保证相同内容的字符串变量引用同一个内存对象。
看下面一段代码：

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
System.out.println(s1.intern() == s3);  // true
String s4 = "bbb";
String s5 = "bbb";
System.out.println(s4 == s5);           // true
```

s1和s2采用new String()的方式新建了两个不同的对象，而s3是通过s1.intern()方法取得一个对象引用。intern()首先把s1引用得对象放到String Pool（字符串常量池）中，然后返回这个对象得引用。因此s1和s3引用得是同一个字符串常量池对象。

有意思的是，如果采用"bbb"双引号的方式创建字符串实例，会自动的将新建的对象放入String Pool中。

通过javap查看字节码，可能会更清楚的看到有什么不同：

```java
javap
String s1 = new String("aaa");
String s3 = s1.intern();
String s4 = "bbb";

public static void main(java.lang.String[]) throws java.lang.Exception;
    Code:
       0: new           #2                  // class java/lang/String
       3: dup                               
       4: ldc           #3                  // String aaa //将aaa从常量池中推送至栈顶
       6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V  //调用String的构造方法，参数就是上一步取出来的常量值
       9: astore_1                          
      10: aload_1                           
      11: invokevirtual #5                  // Method java/lang/String.intern:()Ljava/lang/String; //调用栈顶对象的native方法，并返回
      14: astore_2                          
      15: ldc           #6                  // String bbb //将bbb常量值从常量池中推送至栈顶
      17: astore_3                          
      18: return
```

可以看到，不同之处就在于aaa从常量池中取出后，又生成了一个新的对象引用。

* HotSpot中字符串常量池保存哪里？永久代？方法区还是堆区？

1. 运行时常量池（Runtime Constant Pool）是虚拟机规范中是方法区的一部分，在加载类和结构到虚拟机后，就会创建对应的运行时常量池；而字符串常量池是这个过程中常量字符串的存放位置。所以从这个角度，字符串常量池属于虚拟机规范中的方法区，它是一个逻辑上的概念；而堆区，永久代以及元空间是实际的存放位置。
2. 不同的虚拟机对虚拟机的规范（比如方法区）是不一样的，只有 HotSpot 才有永久代的概念。
3. HotSpot也是发展的，由于一些问题在新窗口打开的存在，HotSpot考虑逐渐去永久代，对于不同版本的JDK，实际的存储位置是有差异的，具体看如下表格：

| JDK版本 | 是否有永久代，字符串常量池放在哪里？| 方法区逻辑上规范，由哪些实际的部分实现的？|
|---|---|---|
| jdk1.6及之前 | 有永久代，运行时常量池（包括字符串常量池），静态变量存放在永久代上  | 这个时期方法区在HotSpot中是由永久代来实现的，以至于这个时期说方法区就是指永久代  |
| jdk1.7       | 有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中；| 这个时期方法区在HotSpot中由永久代（类型信息、字段、方法、常量）和堆（字符串常量池、静态变量）共同实现  |
| jdk1.8及之后 	| 取消永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆中 	| 这个时期方法区在HotSpot中由本地内存的元空间（类型信息、字段、方法、常量）和堆（字符串常量池、静态变量）共同实现 |

## <span id="head8">编译优化

使用“+”连接常量字符串与常量字符串的时候，会将字符串全部加在一起存放。如果用“+”连接常量字符串与变量时，则是创建StringBuilder或StringBuffer来拼接。

```java
javap
int i= 10;
String s1 = "aaa" + i;
String s2 = "aaa" + "bbb";

public static void main(java.lang.String[]) throws java.lang.Exception;
    Code:
       0: bipush        10
       2: istore_1
       3: new           #2                  // class java/lang/StringBuilder
       6: dup
       7: invokespecial #3                  // Method java/lang/StringBuilder."<init>":()V
      10: ldc           #4                  // String aaa
      12: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder; //使用StringBuilder.append拼接aaa
      15: iload_1
      16: invokevirtual #6                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder; ////使用StringBuilder.append拼接10
      19: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      22: astore_2
      23: ldc           #8                  // String aaabbb //直接拼接成aaabbb
      25: astore_3
      26: return
```

## 参考

* [String](https://github.com/xfhy/Android-Notes/blob/master/Blogs/Java/%E5%9F%BA%E7%A1%80/String.md)
* [Java基础-知识点](https://www.pdai.tech/md/java/basic/java-basic-lan-basic.html#string)
* [为什么说String是线程安全的](https://www.cnblogs.com/651434092qq/p/11168608.html)