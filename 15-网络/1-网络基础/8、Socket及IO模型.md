#### Socket及IO模型

程序根据Socket文件的FD：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523205349980.png" alt="image-20210523205349980" style="zoom: 33%;" />

同步多线程模型（BIO）:

![image-20210523211806888](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523211806888.png)

工作线程，read的时候会去内核中的socket中获取，如果没有数据到达会阻塞。

NIO模型：

![image-20210523212347531](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523212347531.png)

工作线程不会去read，当数据准备好的时候，由selector通知工作线程。是一种reactor模型。

```java
public class NIOServer {

    ServerSocketChannel ssc;

    public void listen(int port) throws IOException {
        ssc = ServerSocketChannel.open();
        ssc.bind(new InetSocketAddress(port));
        // Reactive
        ssc.configureBlocking(false);

        Selector selector = Selector.open();
        ssc.register(selector, ssc.validOps(), null);
        ByteBuffer buffer = ByteBuffer.allocate(1024 * 16);

        for (; ; ) {
            int numOfKeys = selector.select();

            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectedKeys.iterator();

            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                // 刚开始
                if (key.isAcceptable()) {
                    SocketChannel chanel = ssc.accept();
                    if(chanel == null){
                        continue;
                    }

                    // kernel -> mmap(buffer) -> channel -> user(buffer)
                    // 向selector注册，selector就可以通知我们，下次 通过key就可以获取到channel
                    chanel.configureBlocking(false);
                    chanel.register(selector, SelectionKey.OP_READ);
                }
                // 某次读写
                else{
                    SocketChannel channel = (SocketChannel) key.channel();
                    // _ _ _ _ _ _ _ _
                    //         P(position)
                    //         L(limit)
                    buffer.clear();// position 清零，操作非常快，并没有真正清除数据
                    channel.read(buffer);
                    String request = new String(buffer.array());

                    // Logic...

                    buffer.clear();
                    buffer.put("HTTP/1.1 200 ok\n\nHello NIO !".getBytes());

                    // H T T P / 1 ... ! _ _ _ _
                    //                   P/L
                    // P                 L
                    buffer.flip();// 翻转一下，方便下面写的时候从头开始读
                    channel.write(buffer);
                    channel.close();
                }
            }

        }

    }

    public static void main(String[] args) throws IOException {
        NIOServer nioServer = new NIOServer();
        nioServer.listen(8888);

    }
}
```

