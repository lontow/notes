# 基础 I/O

## I/O流

I/O流是一系列数据。可使用 输入流读取数据，一次一项，可使用输出流写数据，一次一项。

### 字节流

可使用字节流处理8位的字节。所有的字节流类都是`InputStream` 和`OutputStream`的后代。所有其他的流都是建立在字节流的基础之上的

#### 使用字节流

使用`FileInputStream`和`FileOutputStream` 实现拷贝 xanadu.txt 的内容，一次一个字节。

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class CopyBytes {
    public static void main(String[] args) throws IOException {

        FileInputStream in = null;
        FileOutputStream out = null;

        try {
            in = new FileInputStream("xanadu.txt");
            out = new FileOutputStream("outagain.txt");
            int c;//只是用 后8位

            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } finally {
            //避免资源泄露
            if (in != null) {
                in.close();
            }
            if (out != null) {
                out.close();
            }
        }
    }
}
```



### 字符流

Java 使用Unicode 存储字符。字符流能够**自动**将Unicode 转换为其他格式，或者将其他格式转换为Unicode（**自动字符集转换**）。对很多程序来说，字符流的兼容性要高于字节流。

所有的类都是`Reader`和`Writer`的后代。

#### 使用字符流

使用字符流实现文件内容拷贝：

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class CopyCharacters {
    public static void main(String[] args) throws IOException {

        FileReader inputStream = null;
        FileWriter outputStream = null;

        try {
            
            //使用FileReader 和 FileWriter
            inputStream = new FileReader("xanadu.txt");
            outputStream = new FileWriter("characteroutput.txt");

            int c;//使用后16位存储字符
            while ((c = inputStream.read()) != -1) {
                outputStream.write(c);
            }
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
}
```

字符流是字节流的包装。

另有`InputStreamWriter`和`OutputStreamReader`可以实现**字节-字符**转换。

#### 面向行的I/O

以 `\r\n`,`\r`或者`\n`结束为一行。

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.BufferedReader;
import java.io.PrintWriter;
import java.io.IOException;

public class CopyLines {
    public static void main(String[] args) throws IOException {

        BufferedReader inputStream = null;
        PrintWriter outputStream = null;

        try {
            inputStream = new BufferedReader(new FileReader("xanadu.txt"));
            outputStream = new PrintWriter(new FileWriter("characteroutput.txt"));

            String l;//读一行
            while ((l = inputStream.readLine()) != null) {
                outputStream.println(l);//输出带换行的一行
            }
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
}
```



### 缓冲流

缓冲流可以提高效率。buffer 空的时候，才实际进行读操作，buffer 满的时候，实际进行写操作。

将上几节的未缓冲流转换为缓冲流可直接使用构造函数进行。

```java
inputStream = new BufferedReader(new FileReader("xanadu.txt"));
outputStream = new BufferedWriter(new FileWriter("character-output.txt"));
```

有四个类可用作包装：

- `BufferedInputStream`,`BufferedOutputStream` ->字节流
- `BufferedReader`,`BufferedWriter` ->字符流 

#### 刷新缓冲

flush 方法对所有的输出流都有效，但只对缓冲流有作用。有很多类支持自动刷新。例如：当PrintWriter调用println或者format时可以自动刷新。

### Scanning and Formatting

Scanner 负责切分输入数据，formatting负责组装数据。

#### Scanning

Scanner 的对象能够将输入数据根据它的数据类型分成独立的部分。

默认情况下，Scanner 使用空白字符(空格，tab,换行符等[^1])划分。下面的示例将读取文件xanadu.txt 的文件内容然后打印出来。

```java
import java.io.*;
import java.util.Scanner;

public class ScanXan {
    public static void main(String[] args) throws IOException {

        Scanner s = null;

        try {
            //使用 Scanner划分
            s = new Scanner(new BufferedReader(new FileReader("xanadu.txt")));

            while (s.hasNext()) {
                System.out.println(s.next());
            }
        } finally {
            if (s != null) {
                //虽然Scanner 不是流，仍需要使用关闭来关闭其底层使用的流
                s.close();
            }
        }
    }
}
```



设置不同的分割符，使用正则表达式指定，例如使用逗号后跟一些空白字符。

```java
s.useDelimiter(",\\s*");
```

ScanXan 将所有分割后的token当作字符串，但Scanner 支持将其转换为所有的java元类型(除了char)和BigInteger 和 BigDecimal.

**注意：**一定要指定locale，以确保java使用正确的分隔符和小数形式。

```
//文件内容如下
8.5
32,767
3.14159
1,000,000.1
```



```java
import java.io.FileReader;
import java.io.BufferedReader;
import java.io.IOException;
import java.util.Scanner;
import java.util.Locale;

public class ScanSum {
    public static void main(String[] args) throws IOException {

        Scanner s = null;
        double sum = 0;

        try {
            s = new Scanner(new BufferedReader(new FileReader("usnumbers.txt")));
            s.useLocale(Locale.US);//设置locale

            while (s.hasNext()) {
                if (s.hasNextDouble()) {
                    sum += s.nextDouble();
                } else {
                    s.next();
                }   
            }
        } finally {
            s.close();
        }

        System.out.println(sum);
    }
}
```

#### Formatting

PrintWriter(字符流) 和PrintStream(字节流,主要用于System.out和System.err) 实现了Formatting。两者实现了同一套转换内部数据到格式化输出的方法集。

- print 和 println 使用标准方式格式化。使用 toString 函数。
- format 更精细的格式化。使用格式化字符串[^2]进行格式化

```java
public class Root2 {
    public static void main(String[] args) {
        int i = 2;
        double r = Math.sqrt(i);
        
        System.out.format("The square root of %d is %f.%n", i, r);
        //将输出 The square root of 2 is 1.414214.
    }
}
```

所有的以`%`开始的，以一个或者两个 *conversion* 结束的标识符指定转换的形式。

- %d 十进制整数
- %f 浮点数
- %n 行结束符
- %x  十六进制
- %s 字符串
- %tB 本地月名

**注意：**\n 只会产生回车。%n可根据平台自动产生不同的换行。

格式标识符可包含额外的元素实现自定义的格式化输出。

```java
public class Format {
    public static void main(String[] args) {
        System.out.format("%f, %1$+020.10f %n", Math.PI);
        //输出 3.141593, +00000003.1415926536
    }
}
```

自定义格式化如下：

 ![自定义格式化](./imgs/io-spec.gif)

-  **Argument Index** 指明匹配参数列表中第几个参数（从1开始）
- **Flags** + 表示带符号，0表示填充0.- 右边填充，“，”使用本地分隔符。
- **Width** 输出值的最小宽度。默认是左边填充空格
- **Precision** 指定浮点数的精度。对%s表示最大长度若超出长度，将被右截断。



### I/O from the CommandLine

java可通过标准流和Console 实现命令行交互。

#### 标准流——字节流

- System.in 标准输入
- System.out标准输出——printStream 提供一系列字节流功能
- System.err标准错误——printStream 提供一系列字节流功能

#### Console

提供字节流输入输出方法。通过System.console 可获得Console 对象。Console 支持安全密码设置（readPassword）。

更改密码实例：

```java
import java.io.Console;
import java.util.Arrays;
import java.io.IOException;

public class Password {
    
    public static void main (String args[]) throws IOException {

        Console c = System.console();
        if (c == null) {
            System.err.println("No console.");
            System.exit(1);
        }

        String login = c.readLine("Enter your login: ");
        char [] oldPassword = c.readPassword("Enter your old password: ");

        if (verify(login, oldPassword)) {
            boolean noMatch;
            do {
                char [] newPassword1 = c.readPassword("Enter your new password: ");
                char [] newPassword2 = c.readPassword("Enter new password again: ");
                noMatch = ! Arrays.equals(newPassword1, newPassword2);
                if (noMatch) {
                    c.format("Passwords don't match. Try again.%n");
                } else {
                    change(login, newPassword1);
                    c.format("Password for %s changed.%n", login);
                }
                Arrays.fill(newPassword1, ' ');
                Arrays.fill(newPassword2, ' ');
            } while (noMatch);
        }

        Arrays.fill(oldPassword, ' ');
    }
    
    // Dummy change method.
    static boolean verify(String login, char[] password) {
        // This method always returns
        // true in this example.
        // Modify this method to verify
        // password according to your rules.
        return true;
    }

    // Dummy change method.
    static void change(String login, char[] password) {
        // Modify this method to change
        // password according to your rules.
    }
}
```



### 数据流

数据流支持元数据类型(boolean,int,double,short,long,float,double)和 String 的二进制I/O.所有数据流实现了DataInput 接口或者DataOutput 接口。`DaInputStream`和`DataOutputStream`分别实现了这两个接口。这两个类后都必须传入底层的字节流，然后构建。

下面实例实现将数据写入到文件 `invoicedata`，并将其读出：

```java
//文件名
static final String dataFile = "invoicedata";
//要写入的数据
static final double[] prices = { 19.99, 9.99, 15.99, 3.99, 4.99 };
static final int[] units = { 12, 8, 13, 29, 50 };
static final String[] descs = {
    "Java T-shirt",
    "Java Mug",
    "Duke Juggling Dolls",
    "Java Pin",
    "Java Key Chain"
};
//必须传入字节流对象来构建（同时使用了缓冲流）

//注意异常处理未实现
DataOutputStream out = new DataOutputStream(new BufferedOutputStream(
              new FileOutputStream(dataFile)));
//写入数据
for (int i = 0; i < prices.length; i ++) {
    out.writeDouble(prices[i]);//double
    out.writeInt(units[i]);//int
    out.writeUTF(descs[i]);//UTF-8  字符串
}
//输入流
//必须传入字节流对象来构建（同时使用了缓冲流）
//注意异常处理未实现
DataInputStream in = new DataInputStream(new
            BufferedInputStream(new FileInputStream(dataFile)));

double price;
int unit;
String desc;
double total = 0.0;

try {
    while (true) {
        //read 与 write 对应
        price = in.readDouble();
        unit = in.readInt();
        desc = in.readUTF();
        System.out.format("You ordered %d" + " units of %s at $%.2f%n",
            unit, desc, price);
        total += unit * price;
    }
} catch (EOFException e) {
    //所有的DataInput实现方法使用 EOFException 异常表示文件的结束
}
```



### 对象流

对象I/O的支持。通过实现 Serializable 接口实现支持。  

- ObjectInput  DataInput j的子接口---> ObjectInputStream  
- ObjectOutput DataOutput j的子接口--->ObjectOutputStream 

元类型的的I/O也有对象流的实现。因为对象流I/O可以混合元类型和对象流。

使用BigDecimal表示小数:

```java
import java.io.*;
import java.math.BigDecimal;
import java.util.Calendar;
 
public class ObjectStreams {
    static final String dataFile = "invoicedata";
 
    static final BigDecimal[] prices = { 
        new BigDecimal("19.99"), 
        new BigDecimal("9.99"),
        new BigDecimal("15.99"),
        new BigDecimal("3.99"),
        new BigDecimal("4.99") };
    static final int[] units = { 12, 8, 13, 29, 50 };
    static final String[] descs = { "Java T-shirt",
            "Java Mug",
            "Duke Juggling Dolls",
            "Java Pin",
            "Java Key Chain" };
 
    public static void main(String[] args) 
        throws IOException, ClassNotFoundException {
  
        ObjectOutputStream out = null;
        try {
            out = new ObjectOutputStream(new
                    BufferedOutputStream(new FileOutputStream(dataFile)));
 			//使用WriteObject写对象
            out.writeObject(Calendar.getInstance());
            for (int i = 0; i < prices.length; i ++) {
                out.writeObject(prices[i]);
                out.writeInt(units[i]);
                out.writeUTF(descs[i]);
            }
        } finally {
            out.close();
        }
 
        ObjectInputStream in = null;
        try {
            in = new ObjectInputStream(new
                    BufferedInputStream(new FileInputStream(dataFile)));
 
            Calendar date = null;
            BigDecimal price;
            int unit;
            String desc;
            BigDecimal total = new BigDecimal(0);
 			//使用 readObject读对象。若类型转换失败，抛出ClassNotFoundException 异常
            date = (Calendar) in.readObject();
 
            System.out.format ("On %tA, %<tB %<te, %<tY:%n", date);
 
            try {
                while (true) {
                    price = (BigDecimal) in.readObject();
                    unit = in.readInt();
                    desc = in.readUTF();
                    System.out.format("You ordered %d units of %s at $%.2f%n",
                            unit, desc, price);
                    total = total.add(price.multiply(new BigDecimal(unit)));
                }
            } catch (EOFException e) {}
            System.out.format("For a TOTAL of: $%.2f%n", total);
        } finally {
            in.close();
        }
    }
}
```

#### 复杂对象的输入和输出

一个对象中可能包含其他对象的引用吧，这些其他对象的引用可能又包含另一些对象的引用。readObject和writeObject会自动做这些事.

例如: a包含b和c的引用,b又包含d和e的引用。

![](./imgs/io-trav.gif)



如果在同一流中的两个对象同时包含同一对象的引用。流中只会包含该对象的一份拷贝，如果向流中写同一个对象两次，实际上只是写了两次引用。但如果在不同的流中，便会保存两个副本。

## 文件I/O（JDK7版本）

### Path

### 文件操作

------

[^1]:[`Character.isWhitespace`](https://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#isWhitespace-char-).包含所有的空白字符
[^2]: [`format string syntax`](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html#syntax) 提供了详细的语法，类似C