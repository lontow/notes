### ByteBuf——Netty的数据容器

ByteBuf是对NIO 中ByteBuffer 的替换。

#### ByteBuf API

Netty 中处理数据的API 有两个组件：`abstract class ByteBuf` 和`interface ByteBufHolder`

优势如下：

* 可以扩展到用户定义的缓冲类型
* 通过内建的复合缓冲类型实现透明零拷贝
* 容量按需增长（像JDK 中的`StringBuilder`）
* 读写模式切换不需要调用ByteBuffer.flip()
* 读写使用不同的索引
* 支持方法链
* 支持引用计数
* 支持池



#### ByteBuf 类

##### 原理

ByteBuf  包含两个不同的索引：一个读索引（readerIndex），一个写索引（writerIndex）。readerIndex跟随读到的字节增加，writerIndex跟随写字节增加。

假如readerIndex 和writerIndex相等的时候，意味着没有可读的数据。当尝试随writerIndex之后的数据时，会抛出`IndexOutOfBoundsException`。read() 和write()函数会增加相应的索引，但是set() 和get()不会。

##### 使用模式

###### heap buffers

将数据存到 JVM 的堆中，基于数组。在池不再被占用的时候，分配和回收很快。与JDK的ByteBuffer相似。

```
Bytebuf heapBuf = ...;
if(heapBuf.hasArray()){//检测是否基于数组
	byte[] array = heapBuf.array();//获得数组引用
	int offset =heapBuf.arrayOffset() + heapBuf.readerIndex();//获得第一个字节位置
	int length= heapBuf.readableBytes();
	handleArray(array,offset，length);//处理逻辑
}
```

###### direct buffers

> 在 JDK 1.4 中新加入NIO (New Input/Output)类，引入了一种基于通道与缓冲区的IO方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在 Java堆中的DirectByteBuffer 对象作为对这块内存的引用进行操作。可以避免java堆和Native堆中来回复制数据。
>
> ——深入理解Java虚拟机（周志明）[^1]



直接内存中的内容存放在通常JVM 垃圾回收堆之外的内存。所以直接缓存很适合网络数据传输。如果数据保存在堆中，JVM 可能会在发送数据之前将数据拷贝到直接内存中。

最主要缺点是，内存的分配和回收较堆慢。

```
Bytebuf directBuf = ...;
if(!directBuf.hasArray()){//检测是否基于数组
	int length= directBuf.readableBytes();
	byte[] array = new byte[length];//创建数组
	directBuf.getBytes(directBuf.readerIndex(),array);//copy
	handleArray(array,offset，length);//处理逻辑
}
```

###### composite buffers

多个ByteBuf的抽象。可以添加和删除 ByteBuf实例。通过 ByteBuf的子类`Composite-ByteBuf`实现。

***注意：***ByteBuf实例可能同时包含直接和间接的分配方式。如其中只有一个实例，调用CompositeByteBuf.hasArray() 返回其组件的hasArray() 。否则，返回false。

有这样一个场景：假设一个消息由两部分组成。header 和 body。这两部分由不同的应用程序模块产生。应用程序能够重用 body,并加上自己的header ,以产生新的消息。这时，CompsiteByteBuf 可以避免buffer 不必要的拷贝。

```java
CompositeByteBuf messageBuf = unPooled.compositeBuffer();
ByteBuf headerBuf = ...;
ByteBuf bodyBuf = ...;
messgeBuf.addComponnets(headerBuf,bodyBuf);
......
messageBuf.removeComponet(0);//remove the header
for(ByteBuf buf:messageBuf){
	Sytem.out.println(buf.toString());
    //数据访问
    int length = comBuf.readableBytes();
    byte[] array = new byte[length];
    compBuf.getBytes(compBuf.readerIndex(),array);
    //handleArray
}
```

访问CompositeByteBuf 中的数据与访问 direct Buffers 相似。Netty优化了Socket IO.排除了JDK buffer实现中可能存在的性能和内存损失。

##### 字节级操作

###### 随机读取索引

可以像数组一样存取，不改变readerIndex 和writeIndex

```java
ByteBuf buffer = ...;
for (int i = 0; i < buffer.capacity(); i++) {
byte b = buffer.getByte(i);
System.out.println((char) b);
}
```

###### 顺序存取索引

ByteBuffer 在读和写之间必须调用flip()。下图展示了ByteBuf的内部结构：

![](./imgs/ByteBuf_interval.png)

+ 可丢弃字节

可使用discardReadBytes()将会回收上图的Discardable bytes。readerIndex 和WriterIndex都会减小。readerIndex变为零。由于涉及内存拷贝，因而需要少用。

+ 可读字节





------

[^1]:深入理解Java虚拟机（周志明）,第三版。2.2.7直接内存

