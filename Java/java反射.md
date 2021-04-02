## 反射

反射能够测试或者修改运行在JVM上应用程序的运行时行为。

* 优点：
  1. **功能扩展** 使应用程序可以使用外部的用户自定义的类
  2. **类浏览和可视开发环境** 
  3. **调试和测试工具** 调试类的私有成员。使用反射获取系统级的set api 来进行测试
* 缺点：
  1. **性能损失** 使用反射可能导致特定的JVM优化不能运行，且反射得到的操作比非反射操作要慢
  2. **安全限制** 当运行在 SecurityManager下时，反射可能获取不到运行时权限。
  3. **内部显露** 反射可访问私有成员，因而可能破坏移植性。

### 类

所有的类型要么是引用要么是元类型。类，枚举，数组和接口都是引用类型。对所有的对象类型，JVM实例化一个 java.lang.Class对象。该对象提供测试对象的类型信息和成员。同时它也提供创建类和对象的方法。

Class对象是所有反射api的入口。除了java.lang.reflect.ReflectPermission之外，所有的反射类都没有公开的构造函数。但可用通过Class对象的相应函数获得这些对象。

#### 获得Class 对象

##### Object.getClass()

如果能够获得一个对象的实例，则可以通过getClass()方法获得Class对象。如下：

```java
//获取 String Class 对象
Class c = "foo".getClass();

//System.out.console()返回与JVM相关联的唯一console对象
//返回 java.io.Console Class对象
Class c = System.out.console().getClass();

//返回 Enum Class对象
enum E{A,B}
Class c=A.getClass();

//arrays is Objects 
//返回一个由 byte组成的数组Class对象
byte[] bytes = new byte[1024];
Class c = bytes.getClass();

import java.util.HashSet;
import java.util.Set;


//返回一个java.util.HashSet Class对象
Set<String> s = new HashSet<String>();
Class c = s.getClass();
```

##### .class 语法

当未获得对象实例时，可通过 `.class`获得Class对象。对元类型（`boolean`, `byte`, `short`, `int`, `long`, `char`, `float`, and `double`.）也能使用这种方式。

```java
//元类型
boolean b;
Class c = b.getClass();   // compile-time error,元类型不能被解引用
Class c = boolean.class;  // correct

//
Class c = java.io.PrintStream.class;
//
Class c = int[][][].class;
```

##### Class.forName()

如果可以获得一个类的全验名称，则可以使用静态方法`Class.forName()`获得Class对象。但不能通过其获得元类型。数组名称，使用`Class.getName()`获得。`Class.getName()`可以对元类型使用。

```java
Class c = Class.forName("com.duke.MyLocaleServiceProvider");

//数组
Class cDoubleArray = Class.forName("[D");//一维

Class cStringArray = Class.forName("[[Ljava.lang.String;");//二维

Class c1= Class.forName(int[].class.getName());//ClassNotFoundException
```

##### 元类型封装类的TYPE域

```java
Class c =Double.TYPE;
Class c = Void.TYPE;
```

##### 返回类的方法

首先要获得Class对象

```java
//返回当前类的超类
Class.getSuperclass();

//返回所有当前类的公开类，接口，枚举的成员（包括继承来的）
Class<?>[] c = Character.class.getClasses();

//返回所有在当前类中显式声明的类，接口，和枚举
Class<?>[] c = Character.class.getDeclaredClasses();

//返回声明该成员的类
Class.getDeclaringClass()
java.lang.reflect.Field.getDeclaringClass()
java.lang.reflect.Method.getDeclaringClass()
java.lang.reflect.Constructor.getDeclaringClass()
//例如    
import java.lang.reflect.Field;

Field f = System.class.getField("out");
Class c = f.getDeclaringClass();//c将代表System.class
```

匿名类没有声明类，但是有包含类(Enclosing class)

```java
public class MyClass {
    static Object o = new Object() {
        public void m() {} 
    };
    static Class<c> = o.getClass().getEnclosingClass();//将返回null
    //获得包含类
    static Class<c> = o.getClass().getEnclosingClass();//将返回MyClass
}
```



### 成员

### 数组和枚举类型

