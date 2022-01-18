# 【多线程高并发】Java线程间通信的方式

**Java线程间通信的方式：**

- 共享变量；
- 等待/通知；
- 管道流；

## 1.共享变量

- volatile修饰的变量，线程间可见，可使用这种变量作为线程间传递消息的媒介；

## 2.等待/通知

- 同步代码中利用锁对象的wait和notify方法来进行通信；

## 3.管道流

- 管道输入流/输出流  和普通的文件输入/输出流 不同之处在于，它主要用于线程之间的数据传输，而传输的媒介为内存；
- 管道输入/输出流主要包括了如下4种具体实现：PipedOutputStream、PipedInputStream、PipedReader和PipedWriter，前两种面向字节，而后两种面向字符。

```java
package com.demo.other;

import java.io.PipedReader;
import java.io.PipedWriter;

public class Piped {
    public static void main(String[] args) throws Exception {
        PipedWriter out = new PipedWriter();
        PipedReader in = new PipedReader();
// 将输出流和输入流进行连接，否则在使用时会抛出IOException
        out.connect(in);
        Thread printThread = new Thread(new Print(in), "PrintThread");
        printThread.start();
        int receive = 0;
        try {
            while ((receive = System.in.read()) != -1) {
                out.write(receive);
            }
        } finally {
            out.close();
        }
    }


    static class Print implements Runnable {
        private PipedReader in;
        public Print(PipedReader in) {
            this.in = in;
        }

        @Override
        public void run() {
            int receive = 0;
            try {
                while ((receive = in.read()) != -1) {
                    System.out.print((char) receive);
                }
            } catch (Exception ex) {
            }
        }
    }
}

```

