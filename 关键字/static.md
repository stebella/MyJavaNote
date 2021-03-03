# static关键字

资料：[Java中的static关键字解析](https://www.cnblogs.com/dolphin0520/p/3799052.html)

static表示‘**唯一**’这种特性，这种唯一体现在，修饰了成员变量或者成员方法，则在所有实例中唯一。

从存储区域来看，static可以修饰方法或者成员变量，如果仅用static修饰的成员变量，是存放在静态存储区中的，如果是static final修饰的成员变量，其实是一个唯一不变的常量，存放在常量存储区中。所以final强调‘**不变**’这一特性。

## static方法

thinking in java中有一句话：“static方法就是没有this的方法。在static方法内部不能调用非静态方法，反过来是可以的。而且可以在没有创建任何对象的前提下，仅仅通过类本身来调用static方法。这实际上正是static方法的主要用途。”

简而言之就是：**方便在没有创建对象的情况下来进行调用（方法/变量）**。

static方法一般称作静态方法，由于静态方法不依赖于任何对象就可以进行访问，因此对于静态方法来说，是没有this的，因为它不依附于任何对象，既然都没有对象，就谈不上this了。并且由于这个特性，在静态方法中不能访问类的非静态成员变量和非静态成员方法，**因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用**。

关于构造器是否是static方法可参考：http://blog.csdn.net/qq_17864929/article/details/48006835

构造方法不是静态方法，但是静态的。个人理解在静态方法中this是找不到实例的，但是在构造方法中可以使用this找到当前对象，而在没有实例化之前可以调用的方法是静态方法，所以构造方法是静态的。

## static变量

static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量**被所有的对象所共享**，在**内存中只有一个副本**，它**当且仅当在类初次加载时会被初始化**。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。

static成员变量的初始化顺序按照定义的顺序进行初始化。

## static代码块

static关键字还有一个比较关键的作用就是，用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照**static块的顺序**来执行每个static块，并且只会执行一次。

为什么说static块可以用来优化程序性能，是因为它的特性:**只会在类加载的时候执行一次**。下面看个例子：

```java
class Person {
    private Date birthDate;

    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }

    boolean isBornBetween() {
        Date startDate = Date.valueOf("1946");
        Date endDate = Date.valueOf("1964");
        return birthDate.compareTo(startDate) >= 0
                && birthDate.compareTo(endDate) < 0;
    }
}
```

isBornBetween方法判断是否在1946-1964年出生，这样每次调用该方法都会创建两个Date对象，startDate和endDate，造成控件浪费。抓住**判断出生的两个年份是固定的，是静态的**，所以可以在类加载时初始化该固定部分一次，以后就不用初始化了，于是可以这样写：

```java
class Person {
    private Date birthDate;
    private static Date startDate, endDate;

    static{
        startDate = Date.valueOf("1946-02-02");
        endDate = Date.valueOf("1964-02-02");
    }

    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }

    boolean isBornBetween() {
        return birthDate.compareTo(startDate) >= 0
                && birthDate.compareTo(endDate) < 0;
    }
}

public class TestStatic {
    public static void main(String[] args) {
        Person person = new Person(Date.valueOf("1948-02-02"));
        System.out.println(person.isBornBetween());
    }
}
```

抓住了判断年份的这两个对象是固定的这一特性，在类加载时仅初始化这两个对象一次。此外需要注意：**static是不允许用来修饰局部变量**

## static修饰类

参考资料：[static关键字修饰类](https://www.cnblogs.com/a8457013/p/8078826.html)， [静态内部类何时初始化](https://www.cnblogs.com/maohuidong/p/7843807.html)， [java 内部类和静态内部类的区别](https://www.cnblogs.com/aademeng/articles/6192954.html)

首先搞清楚一个问题：类何时被加载？答案是：**静态内部类和普通类一样，在使用的时候才会被加载**。在加载的过程中，如果发现有静态代码，先执行静态代码，且仅执行一次，下一次使用该类，如实例化一个对象，也不会再执行静态代码。静态内部类的单例就利用的这种原理。

java里面static一般用来修饰成员变量或函数。但有一种特殊用法是用static修饰内部类，**普通类是不允许声明为静态的，只有内部类才可以。**

被static修饰的内部类可以直接作为一个普通类来使用，而不需实例一个外部类，代码如下：

```java
public class OutClassTest {
    static int a;

    int b;

    public static void test() {
        System.out.println("outer class static function");
    }

    public static void main(String[] args) {
        OutClassTest oc = new OutClassTest();
        // new一个外部类
        OutClassTest oc1 = new OutClassTest();
        // 通过外部类的对象new一个非静态的内部类
        OutClassTest.InnerClass no_static_inner = oc1.new InnerClass();
        // 调用非静态内部类的方法
        System.out.println(no_static_inner.getKey());

        // 调用静态内部类的静态变量
        System.out.println(OutClassTest.InnerStaticClass.static_value);
        // 不依赖于外部类实例,直接实例化内部静态类
        OutClassTest.InnerStaticClass inner = new OutClassTest.InnerStaticClass();
        // 调用静态内部类的非静态方法
        System.out.println(inner.getValue());
        // 调用内部静态类的静态方法
        System.out.println(OutClassTest.InnerStaticClass.getMessage());
    }

    private class InnerClass {
        // 只有在静态内部类中才能够声明或定义静态成员
        // private static String tt = "0";
        private int flag = 0;

        public InnerClass() {
            // 三.非静态内部类的非静态成员可以访问外部类的非静态变量和静态变量
            System.out.println("InnerClass create a:" + a);
            System.out.println("InnerClass create b:" + b);
            System.out.println("InnerClass create flag:" + flag);
            //
            System.out.println("InnerClass call outer static function");
            // 调用外部类的静态方法
            test();
        }

        public  String getKey() {
            return "no-static-inner";
        }
    }

    private static class InnerStaticClass {
        // 静态内部类可以有静态成员，而非静态内部类则不能有静态成员。
        private static String static_value = "0";

        private int flag = 0;

        public InnerStaticClass() {
            System.out.println("InnerClass create a:" + a);
            // 静态内部类不能够访问外部类的非静态成员
            // System.out.println("InnerClass create b:" + b);
            System.out.println("InnerStaticClass flag is " + flag);
            System.out.println("InnerStaticClass tt is " + static_value);
        }

        public int getValue() {
            // 静态内部类访问外部类的静态方法
            test();
            return 1;
        }

        public static String getMessage() {
            return "static-inner";
        }
    }

    public OutClassTest() {
        // new一个非静态的内部类
        InnerClass ic = new InnerClass();
        System.out.println("OuterClass create");
    }

}

/**
 * 总结： 
 * 1.静态内部类可以有静态成员(方法，属性)，而非静态内部类则不能有静态成员(方法，属性)。
 * 2.静态内部类只能够访问外部类的静态成员,而非静态内部类则可以访问外部类的所有成员(方法，属性)。
 * 3.实例化一个非静态的内部类的方法：
 *  a.先生成一个外部类对象实例
 *  OutClassTest oc1 = new OutClassTest();
 *  b.通过外部类的对象实例生成内部类对象
 *  OutClassTest.InnerClass no_static_inner = oc1.new InnerClass();
 *  4.实例化一个静态内部类的方法：
 *  a.不依赖于外部类的实例,直接实例化内部类对象
 *  OutClassTest.InnerStaticClass inner = new OutClassTest.InnerStaticClass();
 *  b.调用内部静态类的方法或静态变量,通过类名直接调用
 *  OutClassTest.InnerStaticClass.static_value
 *  OutClassTest.InnerStaticClass.getMessage()
 */
```



## 常见面试题

[初始化顺序总结 - 静态变量、静态代码块、成员变量、构造函数](https://www.cnblogs.com/abcjun/articles/4687539.html)

**子类继承父类时的初始化顺序：**

1.首先初始化父类的static变量和块，它们的级别是相同的，按照代码中出现的顺序初始化

2.初始化子类的static变量和块，它们的级别是相同的，按照代码中出现的顺序初始化

3.初始化父类的普通成员变量，调用父类的构造函数

4.初始化子类的普通成员变量，调用子类的构造函数

看题如下：

```java
package test_static;

/**
 * 装载Test类的时候，先判断有无父类，没有父类。
 * 在装载时，先初始化，且只初始化一次静态代码。
 * 静态代码包括静态成员变量和静态代码块，按代码顺序初始化。
 * 然后初始化普通成员变量紧接着构造方法。
 * 这里需要开始装载Person1类，先静态代码块
 * 装载Person1类后，先初始化Person1类的普通成员变量，然后构造方法
 * @author Administrator
 *
 */
public class Test {
    Person1 person = new Person1("Test");//3
    static{
        System.out.println("test static");//1打印test static
    }

    public Test() {
        System.out.println("test constructor");//6打印test constructor
    }

    public static void main(String[] args) {
        //程序入口
        //MyClass类加载
        new MyClass();
    }
}

class Person1{
    static{
        System.out.println("person static");//4打印person static
    }
    public Person1(String str) {
        System.out.println("person "+str);//5打印person Test，7打印person MyClass
    }
}

/**
 * MyClass类加载的过程中
 * 是否有继承自父类：
 * 如果继承自父类，先装载父类，于是先装载Test
 * 父类装载完之后，装载子类，先初始化静态代码。
 * 父类和子类的静态代码都初始完成后，初始化父类普通成员变量，紧接着构造方法。
 * 然后初始化子类普通成员变量和构造方法，这里又装载一个Person1类
 * 由于Person1类已经装载过了，所以不会走静态方法
 * @author Administrator
 *
 */
class MyClass extends Test {
    Person1 person = new Person1("MyClass");
    static{
        System.out.println("myclass static");//2打印 myclass static
    }

    public MyClass() {
        System.out.println("myclass constructor");//8打印myclass constructor
    }
}
```

运行结果：

test static 
myclass static 
person static 
person Test 
test constructor
person MyClass
myclass constructor

总结：**先初始化父类的静态代码—>初始化子类的静态代码–>初始化父类的非静态代码（普通成员变量）—>初始化父类构造函数—>初始化子类非静态代码（普通成员变量）—>初始化子类构造函数**

精简版：**静态代码–>普通成员变量–>构造方法，如果有父类先按此规则初始化父类。**

















子: