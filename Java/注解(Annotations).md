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

