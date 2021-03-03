在Java中，final关键字可以用来修饰类、方法和变量（包括成员变量和局部变量）。下面就从这三个方面来了解一下final关键字的基本用法。

## 修饰类

当用final修饰一个类时，表明**这个类不能被继承**。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。final类中的成员变量可以根据需要设为final，但是要注意final类中的所有**成员方法都会被隐式地指定为final方法。**

在使用final修饰类的时候，要注意谨慎选择，除非这个类真的在以后**不会用来继承或者出于安全的考虑**，尽量不要将类设计为final类。

## 修饰方法

下面这段话摘自《Java编程思想》第四版第143页：

　　“使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。“

因此，如果只有在想**明确禁止该方法在子类中被覆盖**的情况下才将方法设置为final的。

注：**类的private方法会隐式地被指定为final方法**。

## 修饰变量

修饰变量是final用得最多的地方，首先了解一下final变量的基本语法：

> `注意`
>
> 对于一个final变量，如果是**基本数据类型**的变量，则其数值一旦在**初始化之后便不能更改**；如果是**引用类型的变量**，则在对其初始化之后便**不能再让其指向另一个对象**。

## 深入理解

- 类的final变量和普通变量有什么区别？

  > ```java
  > public class TestFinal {
  > public static void main(String[] args) {
  >        String a = "hello2"; 
  >        final String b = "hello";
  >        String d = "hello";
  >        String c = b + 2; 
  >        String e = d + 2;
  >        System.out.println((a == c));
  >        System.out.println((a == e));
  > }
  > }
  > ```
  >
  > 为什么第一个比较结果为true，而第二个比较结果为fasle。这里面就是final变量和普通变量的区别了，当final变量是基本数据类型以及String类型时，如果在编译期间能知道它的确切值，则编译器会把它当做编译期常量使用。也就是说在用到该final变量的地方，相当于直接访问的这个常量，不需要在运行时确定。这种和C语言中的宏替换有点像。因此在上面的一段代码中，**由于变量b被final修饰，因此会被当做编译器常量，所以在使用到b的地方会直接将变量b替换为它的值**。而对于变量d的访问却需要在运行时通过链接来进行。想必其中的区别大家应该明白了，不过要注意，只有在编译期间能确切知道final变量值的情况下，编译器才会进行这样的优化，比如下面的这段代码就不会进行优化：
  >
  > ```java
  > public class Test {
  >    public static void main(String[] args)  {
  >        String a = "hello2"; 
  >        final String b = getHello();
  >        String c = b + 2; 
  >        System.out.println((a == c));
  > 
  >    }
  > 
  >    public static String getHello() {
  >        return "hello";
  >    }
  > }
  > ```

- 被final修饰的引用变量指向的对象内容可变吗？引用可变吗？

  > ```java
  > public class Test {
  > public static void main(String[] args) {
  >     final int[] array = new int[]{1, 2, 3};
  >     //改变array的内容：允许
  >     array[1] = 4;
  >     for(int i=0; i<array.length; i++){
  >         System.out.println(array[i]);
  >     }
  >     //改变array的引用，使其指向一个新的对象，不允许
  >     //这里报错：
  >     //The final local variable array cannot be assigned. 
  >     //It must be blank and not using a compound assignment
  >     array = new int[3];
  > }
  > }
  > ```
  >
  > **内容可变，引用不可变(=, new 操作都不可行)**

- 形参用final修饰可以达到防止实参被修改的目的吗？

  > ```java
  > class Test1{
  >     public void test1(final int a){
  >         //The final local variable a cannot be assigned. 
  >         //It must be blank and not using a compound assignment
  >         a = 2;
  >     }
  > 
  >     public void test2(final StringBuffer sb){
  >         //The final local variable a cannot be assigned. 
  >         //It must be blank and not using a compound assignment
  >         sb = new StringBuffer("456");
  > 
  >         sb.append("456");
  >     }
  > }
  > 
  > public class TestFinalParameter {
  >     public static void main(String[] args) {
  >         Test1 test1 = new Test1();
  >         int a = 1;
  >         test1.test1(1);
  > 
  >         StringBuffer sb = new StringBuffer("123");
  >         test1.test2(sb);
  >         System.out.println(sb);
  >     }
  > }
  > ```
  >
  > 得到的结果是：123456
  >
  > 很显然，如果形参是基本类型变量，值确实不可变了。但如果形参是引用变量，那么同上，引用不可变，仍旧只能指向实参那个对象，但是内容却可变，可以使用append方法改变其内容。

  

  