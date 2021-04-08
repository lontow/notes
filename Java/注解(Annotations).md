## Annotations

注解，一种元数据，提供关于程序的信息。但不会直接影响被注解的程序

**用处：**

* 给编译器提供信息 可用来检测错误和忽略警告
* 编译时和部署时处理 相应的工具可通过处理注解生成相应的代码，文件等等
* 运行时处理

### 基础

#### 格式

```java
@Entity//例如 @Override

//包括多个元素的注解
@Author(
   name = "Benjamin Franklin",
   date = "3/27/2003"
)

//包含一个元素的注解
@SuppressWarnings(value = "unchecked")
@SuppressWarnings("unchecked")

//应用多个注解
@Author(name = "Jane Doe")
@EBook
class MyClass { ... }

//重复注解（SE8之后）
@Author(name = "Jane Doe")
@Author(name = "John Smith")
class MyClass { ... }
```

#### 注解位置

可放在类，域，方法和其他元素的声明之前。

SE8之后，可被用作类型。（Type Annotations）

### 注解声明

很多注解用来替换代码中的注释

例如：

```java
public class Generation3List extends Generation2List {

   // Author: John Doe
   // Date: 3/17/2002
   // Current revision: 6
   // Last modified: 4/12/2004
   // By: Jane Doe
   // Reviewers: Alice, Bill, Cindy

   // class code goes here

}
```

使用注解添加相同的元数据，首先要定义注解类型(Annotation type)。语法如下：

```java
@interface ClassPreamble {
   String author();
   String date();
   int currentRevision() default 1;
   String lastModified() default "N/A";
   String lastModifiedBy() default "N/A";
   // Note use of array
   String[] reviewers();
}
```

注解类型的定义与接口定义相似，但使用`@interface`作为标识。注解类型是*interface*的一种形式。

上述注解定义中包含注解类型元素。元素看起来像是方法，但可以定义默认值(可选)

在注解定义之后，使用注解的方式如下（将注释进行替换）：

```java
@ClassPreamble(
    author = "lontow",
    date = "3/17/2002",
    currentRevision = 6,
    lastModified = "4/12/2004",
    lastModifiedBy = "lontow",
    //数组注解
    reviewers = {"Alice","Bob","Cindy"}    
)
public class Generation3List extends Generation2List{
    //code
}
```

 **注意：**

如果想让注解内容出现在Javado生成的文档中，需要在注解定义前添加`@Documented`注解：

```java
// import this to use @Documented
import java.lang.annotation.*;

@Documented
@interface ClassPreamble {

   // Annotation element definitions
   
}
```

### 预定义的注解类型

#### Java Language 使用的注解类型

`java.lang`预定义的注解类型：

- **@Deprecated** 指明标记的元素是过时的，被弃用了。当使用被其标记的元素时，编译器将产生警告。同时，被弃用的元素在 Java-doc 注释中，也要同时使用`@deprecated`(**首字母小写**)。

```java
 // Javadoc comment follows
    /**
     * @deprecated
     * explanation of why it was deprecated
     */
    @Deprecated
    static void deprecatedMethod() { }
}
```

- **@Override** 告诉编译器此方法是对其超类的元素的覆盖。（可选）若其标记的方法未正确覆盖，编译器会产生错误。
- **@SuppressWarnings** 告诉编译器不产生指定的警告。

```java
	// 使用了被弃用的函数 
   // 告诉编译器不产生警告
   @SuppressWarnings("deprecation")
    void useDeprecatedMethod() {
        // deprecation warning
        // - suppressed
        objectOne.deprecatedMethod();
    }
```

每个编译器警告属于一个类别。Java Language 有两个类别：`deprecation`和`unchecked`。当使用之前未使用泛型（之后又使用了）的代码时，会产生`unchecked` 警告。同时，关闭多种警告，可以使用以下语法:

```java
@SuppressWarnings({"unchecked", "deprecation"})
```

- **@SafeVarargs** 当其应用在方法或者构造函数时，断言代码不执行潜在的不安全关于 varargs 的参数操作。当这个注解类型被使用时，则不会产生相关varargs 的unchecked警告。
- **@FunctionalInterface** （SE8 及之后）表明这个类型声明是一个函数接口

#### 应用到其他注解的注解（元注解）

应用到其他注解的注解叫元注解。`java.lang.annotation`定义了如下几种元注解：

- **@Retention** 指明注解如何储存。
  1. RetentionPolicy.SOURCE —— 注解只存在于源代码级别，编译器忽略
  2. RetentionPolicy.CLASS——编译保存使用，JVM忽略
  3. RetentionPolicy.RUNTIME——JVM保存，可在整个运行时环境中使用
- **@Documented** 指明无论何时，使用指定的的注解的那些元素，会被写进文档。(javadoc)
- **@Target** 限制注解可用于的java元素类型。主要的类型有：
  - `ElementType.ANNOTATION_TYPE` 注解类型
  - `ElementType.CONSTRUCTOR` 构造器类型
  - `ElementType.FIELD` 域或者属性
  - `ElementType.LOCAL_VARIABLE` 本地变量
  - `ElementType.METHOD`方法
  - `ElementType.PACKAGE` 包
  - `ElementType.PARAMETER` 方法参数
  - `ElementType.TYPE` c类
- **@Inherited** 只能用于类声明。指明该注解可从父类继承。在子类不存在某注解时，会从超类中搜索
- **@Repeatable** (SE8及之后)指明该注解可对相同的声明语句或类型语句重复多次应用。

### 类型注解和可插拔类型系统(SE8)

### 重复注解（SE8）