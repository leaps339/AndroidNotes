# Object

## 目录

* [概述](#head1)
* [equals()](#head2)
  * [等价关系](#head3)
  * [equals()与==](#head4)
  * [实现](#head5)
* [hashCode()](#head6)
* [toString()](#head7)
* [clone()](#head8)
  * [深拷贝、浅拷贝](#head9)

## <span id="head1">概述

Object是Java类库中的一个特殊类，也是所有类的父类。也就是说，Java允许把任何类型的对象赋给Object类型的变量。当一个类被定义后，如果没有指定继承的父类，那么默认父类就是 Object 类。因此，以下两个类表示的含义是一样的。

```java
public class MyClass{…}

public class MyClass extends Object {…}
```

由于Java所有的类都是Object类的子类，所以任何Java对象都可以调用Object类的方法。常见方法：

```java
//返回对象运行时的实例类
public final native Class<?> getClass()
//返回该对象的散列码值
public native int hashCode()
//比较两个对象是否相等
public boolean equals(Object obj)
//创建并返回一个对象的拷贝（浅拷贝）
protected native Object clone() throws CloneNotSupportedException
//返回该对象的字符串表示
public String toString()
//激活等待在该对象的监视器上的一个线程
public final native void notify()
///激活等待在该对象的监视器上的全部线程
public final native void notifyAll()
//在其他线程调用此对象的notify()方法或者notifyAll()方法前，导致当前线程等待
//timeout：等待的最大时间
//nanos：额外时间，纳秒为单位，0-999999
public final void wait(long timeout, int nanos) throws InterruptedException
//当垃圾回收器确定不存在对该对象的更多引用时，对象垃圾回收器调用该方法
protected void finalize() throws Throwable {}
```

## <span id="head2">equals()

### <span id="head3">等价关系

equals()方法用来比较两个对象内容是否相等。在重写equals方法时，要遵循以下原则：

```java
//自反性
x.equals(x); //true
//对称性
x.equals(y) == y.equals(x); //true
//传递性
if (x.equals(y) && y.equals(z))
    x.equals(z); // true;
//一致性
x.equals(y) == x.equals(y); // true
//对任何不是null的对象x调用x.equals(null)结果都为false
x.equals(null); // false;
```

### <span id="head4">equals()与==

* 对于基本类型，==判断两个值是否相等，基本类型没有equals()方法
* 对于引用类型，==判断两个变量是否引用同一个对象，而equals()判断引用的对象是否等价（对象内部数据是否一致）。

```java
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```

### <span id="head5">实现

* 检查是否为同一个对象的引用，如果是直接返回true
* 检查是否为同一个类型，如果不是，直接返回false；
* 将Object对象进行转型；
* 判断每个关键域是否相等；

```java
public class EqualExample {
    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

## <span id="head6">hashCode()

hashCode返回散列值，而equals()是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价（hash冲突）。因此，重写equals()方法时应总是覆盖hashCode()方法，保证等价的两个对象散列值也相等。

hashCode()值通常应用于HashMap、HashSet等数据结构中，它们内部通过获取对象的hashCode()值作为初始key值进行散列存储，两个等价的对象其hashCode()值也应该是一样的，否则在HashMap、HashSet等数据结构中，就会被认为是不同的对象。

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2 没有正常的处理hashCode()值，被认为是两个不等价的对象
```

理想的散列函数应当具有均匀性，即不相等的对象应当均匀的分布在所有可能的散列值上。这就要求在设计具体的实现时需要把所有域的值都考虑进来，目前默认的方法是，可以将每个域都当成是R进制的某一位，然后组成一个R进制的整数。**R一般取31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与2相乘就相当于向左移一位**。

```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```

>一个数与 31 相乘可以转换成移位和减法: 31*x == (x<<5)-x，编译器会自动进行这个优化。

## <span id="head7">toString()

toString() 方法返回该对象的字符串，当程序输出一个对象或者把某个对象和字符串进行连接运算时，系统会自动调用该对象的 toString() 方法返回该对象的字符串表示。

Object 类的 toString() 方法返回“运行时类名@十六进制哈希值”格式的字符串，但很多类都重写了 Object 类的 toString() 方法，用于返回可以表述该对象信息的字符串。

```java
Demo d = new Demo();
System.out.println(d); //输出 Demo@15db9742
```

## <span id="head8">clone()

clone()方法会返回该对象的浅拷贝，对象内属性引用的对象置灰拷贝引用地址，而不会将引用的对象重新分配，相对应的深拷贝则会连引用的对象也重新创建。

>clone规则
>
>* 基本类型：拷贝其值
>* 对象：拷贝其地址引用

返回的是Object对象，我们必须强转才能得到我们需要的类型。clone()方法在Object中是Protected，因此不能在类外访问，克隆一个对象，需要对clone重写。

```java
class Student implements Cloneable {
    private int a;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

Student a = new Student();
try {
    Student b = (Student) a.clone();
} catch (Exception e) {
    // TODO: handle exception
}
```

需注意到clone声明会抛出CloneNotSupportException异常，如果你调用了某个类的clone()方法，而这个类没有实现Cloneable接口，那么就会抛出此异常。

有意思的，Cloneable接口内部没有任何抽象方法，它是一个标记接口。我们所重写的clone()是Object内部的方法，Cloneable接口只是规定，如果一个类没有实现Cloneable接口又调用了clone()方法，就会抛出CloneNotSupportException异常。

```java
public interface Cloneable {
}
```

### <span id="head9">深拷贝、浅拷贝

上面提到，clone()是一种浅拷贝方式创建的对象，与之相对的，就是深拷贝。

* 浅拷贝：原对象和拷贝对象不同，但对象内的成员变量引用的地址相同
* 深拷贝：原对象和拷贝对象不同，且对象内的成员变量引用的地址也不同。

```java
//浅拷贝
class Student implements Cloneable{
    Bage bage = new Bage();

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

class Bage {
    int num = 2;
}

//输出
Student a = new Student();
try {
    Student b = (Student) a.clone();
    a.bage.num = 3;//改变a中成员变量的内布值
    System.out.println(b.bage.num); //输出  3；b也随之改变
} catch (Exception e) {
    // TODO: handle exception
}
```

那么如何实现深拷贝呢，有两种方法：

* 成员变量也实现Cloneable接口，重写clone()方法

```java
class Student implements Cloneable {
    Bage bage = new Bage();

    @Override
    public Object clone() throws CloneNotSupportedException {
        Student result = (Student) super.clone();
        //手动将成员变量也进行clone()赋值
        result.bage = (Bage) bage.clone();
        return result;
    }
}

class Bage implements Cloneable {
    int num = 2;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

//输出
Student a = new Student();
try {
    Student b = (Student) a.clone();
    a.bage.num = 3;//改变a中成员变量的内布值
    System.out.println(b.bage.num); //输出  2；b不随a改变而改变
} catch (Exception e) {
    // TODO: handle exception
}
```

* 序列化与反序列化

```java
//实现serializable接口
class Student implements Serializable {
    Bage bage = new Bage();

    public Student myClone() {
        Student result = null;
        try {
            // 将对象序列化到流里
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
            objectOutputStream.writeObject(this);
            // 将流反序列化为对象
            ByteArrayInputStream inputStream = new ByteArrayInputStream(outputStream.toByteArray());
            ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
            result = (Student) objectInputStream.readObject();
        } catch (Exception e) {
            // TODO: handle exception
        }
        return result;
    }
}
//实现serializable接口
class Bage implements Serializable {
    int num = 2;
}

//输出
Student a = new Student();
try {
    Student b = a.myClone();
    a.bage.num = 3;//改变a中成员变量的内布值
    System.out.println(b.bage.num); //输出  2；b不随a改变而改变
} catch (Exception e) {
    // TODO: handle exception
}
```

## 参考

* [Java基础-知识点](https://www.pdai.tech/md/java/basic/java-basic-lan-basic.html#object-%E9%80%9A%E7%94%A8%E6%96%B9%E6%B3%95)
* [Java Object类详解](http://c.biancheng.net/view/6587.html)