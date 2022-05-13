# Netty

### Netty的介绍

![img](https://my-typora-picgo.oss-cn-shanghai.aliyuncs.com/images/202205132145467.png)

**Netty**是 一个**异步事件驱动**的网络应用程序框架，用于**快速开发可维护的高性能协议服务器和客户端**。

Netty是由[JBOSS](https://baike.baidu.com/item/JBOSS)提供的一个[java开源](https://baike.baidu.com/item/java开源/10795649)框架，现为 [Github](https://baike.baidu.com/item/Github/10145341)上的独立项目。Netty提供异步的、[事件驱动](https://baike.baidu.com/item/事件驱动/9597519)的网络应用程序框架和工具，用以快速开发高性能、高可靠性的[网络服务器](https://baike.baidu.com/item/网络服务器/99096)和客户端程序。

也就是说，Netty 是一个基于NIO的客户、服务器端的编程框架，使用Netty 可以确保你快速和简单的开发出一个网络应用，例如实现了某种协议的客户、[服务端](https://baike.baidu.com/item/服务端/6492316)应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如：基于TCP和UDP的socket服务开发。

“快速”和“简单”并不用产生维护性或性能上的问题。Netty 是一个吸收了多种协议（包括FTP、SMTP、HTTP等各种二进制文本协议）的实现经验，并经过相当精心设计的项目。最终，Netty 成功的找到了一种方式，在保证易于开发的同时还保证了其应用的性能，稳定性和伸缩性。

## Netty的应用场景

1. 互联网行业：在分布式系统中，各个节点之间需要远程服务调用，高性能的RPC框架必不可少，Netty作为异步高性能的通信框架，往往作为基础通信组件被这些RPC框架使用
2. 典型的应用有：阿里的分布式服务框架Dubbo的RPC框架使用Dubbo协议进行节点通信，Dubbo协议默认使用Netty作为基础通信组件，用于实现各个进程节点之间的内部通信
3. Netty作为高性能的基础通信组件，提供了TPC/UDP和HTTP协议，方便定制和开发私有协议栈，账号登陆服务器
4. 大数据领域，经典的Hadoop的高性能通信和序列化组件（AVRO实现数据文件共享）的RPC框架，默认采用Netty进行跨界通信，它的Netty Service基于Netty框架二次封装实现

## I/O模型

1. I/O模型简单的理解：就是用什么样的通道进行数据的发送和接收，很大程度决定了程序通信的性能
2. Java共支持三种网络编程模型/IO模型：BIO、NIO、AIO
3. Java BIO：同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器就需要启动一个线程进行处理，如果这个链接不做任何事情会造成不必要的线程开销
4. Java NIO：同步非阻塞，服务器实现模式为一个线程处理多个请求（连接），即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求就进行处理
5. Java AIO：异步非阻塞，AIO引入了异步通信的概念，采用了Proactor模式，简化了程序编写，有效的请求才启动线程，它的特点是先由才做系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用

## Java BIO基本介绍

1. Java BIO就是传统的Java IO变成，其相关的类和接口在java.io
2. BIO（Blocking I/O）：同步阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善（实现多个客户连接服务器）。

### BIO的简单流程

1. 服务器端启动一个ServerSocket
2. 客户端启动一个Socket对ServerSocket进行通信，默认情况下服务器端需要对每个请求建立一个线程与其通讯
3. 客户端发出请求后，先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝；
4. 如果有响应，客户端线程会等待请求结束后才继续执行

BIO示例代码：

```java
public class BIOServer {
    public static void main(String[] args) throws IOException {
        //1. 创建线程池
        //2. 如果有客户端连接，就创建一个线程，与之通信
        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
        // 创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        System.out.println("服务器启动了");
        while (true) {
            // 监听，等待客户端连接
            System.out.println("等待连接...");
            final Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");

            // 创建一个线程与之通信
            newCachedThreadPool.execute(() -> {
                // 可以与客户端进行通讯
                handler(socket);
            });
        }
    }

    /**
     * 编写一个handler方法与客户端进行通讯
     */
    public static void handler(Socket socket) {

        try {
            System.out.println("线程信息 ID=" + Thread.currentThread().getId() +
                    "name=" + Thread.currentThread().getName());
            byte[] bytes = new byte[1024];
            // 通过socket获取一个输入流
            InputStream inputStream = socket.getInputStream();
            // 循环的读取客户端发送的数据
            while (true) {
                System.out.println("线程信息 ID=" + Thread.currentThread().getId() +
                        "name=" + Thread.currentThread().getName());
                System.out.println("read...");
                int read = inputStream.read(bytes);
                if (read != -1) {
                    //输出客户端发送的数据
                    System.out.println(new String(bytes, 0, read));
                } else {
                    break;
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            System.out.println("关闭和client的连接");
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Java NIO基本介绍

1. Java NIO全程 java non-blocking IO，是指JDK提供的新API。从JDK1.4开始，Java提供了一系列改进输入/输出的新特性，被统称为NIO（即New IO），是同步阻塞的
2. NIO相关类都放在java.nio包下，并且对原java.io包中的很多类进行了改写
3. NIO有三大核心部分：**Channel（通道）**，**Buffer（缓冲区）**，**Selector（选择器）**
4. NIO是面向缓冲区，或者面向块编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区前后移动，这就增加了处理过程中的灵活性，使得它可以提供非阻塞式的高伸缩性网络
5. Java NIO的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变得可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
6. HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.0大了好几个数量级。

Buffer的基本使用

```java
    public static void main(String[] args) {
        // 举例说明buffer的使用 (简单案例)

        // 创建一个Buffer，大小为5，可以存放5个int
        IntBuffer intBuffer = IntBuffer.allocate(5);
        // 向Buffer中 存放数据
        for (int i = 0; i < intBuffer.capacity(); i++) {
            intBuffer.put(i * 2);
        }

        // 如何从Buffer读取数据
        // 将Buffer转换，读写切换
        intBuffer.flip();
        while (intBuffer.hasRemaining()) {
            System.out.println(intBuffer.get());
        }
    }
```



### NIO与BIO的比较

- BIO以流的方式处理数据，而NIO以块的方式处理数据，块I/O的效率比流I/O高很多
- BIO是阻塞的，NIO是非阻塞的
- BIO基于字节流和字符流进行操作，而NIO基于Channel（通道）和Buffer（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector（选择器）用于监听多个通道的事件（比如：连接请求、数据到达等），因此使用单个线程就可以监听多个客户端通信

### NIO三大核心原理

Selector、Channel和BUffer的关系：![image-20220513221200058](https://my-typora-picgo.oss-cn-shanghai.aliyuncs.com/images/202205132255853.png)

1. 每个Channel都对应一个Buffer
2. Selector对应一个线程，一个线程对应多个Channel（连接）
3. 程序切换到哪个Channel是由事件决定的，**Event**就是一个重要的概念
4. Selector会根据不同的事件，在各个通道上切换
5. Buffer就是一个内存块，底层是由一个数组组成的
6. 数据的读取写入是通过Buffer，和BIO有本质的不同，BIO中为输入流或输出流，不能是双向的，NIO的Buffer是可以读也可以写的，需要flip切换
7. Channel也是双向的，可以返回底层操作系统的情况，比如Linux底层的操作系统通道就是双向的

### 缓冲区Buffer

基本介绍：

>缓冲区（Buffer）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象（含数组），该对象提供了一组方法，可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。

Buffer类及其子类：

1. Buffer子类一览：<img src="https://my-typora-picgo.oss-cn-shanghai.aliyuncs.com/images/202205132251988.png" alt="image-20220513225035633"  />

2. Buffer类定义了所有缓冲区都具有四个属性来提供关于其所包含的数据元素的信息：

   ```java
       // Invariants: mark <= position <= limit <= capacity
       private int mark = -1;
       private int position = 0;
       private int limit;
       private int capacity;
   ```

   | 属性     | 描述                                                         |
   | -------- | ------------------------------------------------------------ |
   | mark     | 标记                                                         |
   | position | 位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变值，为下次读写做准备 |
   | limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改的 |
   | capacity | 容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变 |

   ![image-20220513230233731](https://my-typora-picgo.oss-cn-shanghai.aliyuncs.com/images/202205132302764.png)

#### ByteBuffer

对于Java中的基本数据类型，除了boolean都有一个Buffer类型与之对应，最常用的就是ByteBuffer类（二进制数据），ByteBuffer主要有以下方法：

```java
public static ByteBuffer allocateDirect(int capacity);// 创建直接缓冲区
public static ByteBuffer allocate(int capacity);// 设置缓冲区的初始容量
public static ByteBuffer wrap(byte[] array);// 把一个数组放到缓冲区中使用
public static ByteBuffer wrap(byte[] array,int offset, int length);
// 构造初始化位置offset和上界length的缓冲区
public abstract byte get();// 从当前位置position上get，get之后，position会自动+1
public abstract byte get(int index);// 从绝对位置get
public abstract ByteBuffer put(byte b);// 从当前位置上添加，put之后，position会自动+1
public abstract ByteBuffer put(int index, byte b);// 从绝对位置上put
```







