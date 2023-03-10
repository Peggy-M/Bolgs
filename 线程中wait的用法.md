# 线程中wait的用法

~~~ java
Thread thread_d = new Thread(() -> {
            synchronized (resourceA) {
                System.out.println("thread_d get resourceA Lock");
                for (int i = 0; i < 100; i++) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if(i==5) {
                        try {
                            //让该线程等待阻塞 5 秒中后重新获取锁对象
                            resourceA.wait(1000*5);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println("thread_d ===> " + i);
                }
            }
        }, "thread_d");

        Thread thread_e = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (resourceA) {
                System.out.println("tread_e get resourceA Lock");
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if(i==6){
                        try {
                            resourceA.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    System.out.println("thread_e ===> " + i);
                }
            }
        }, " thread_e");
~~~



如果启动上面两个线程会发生什么现象呢？

首先线程 thread_d 启动，1秒钟后线程 thread_e 启动，并且线程 thread_d 优先获取锁共享变量锁 resourceA ，在 5 秒中后，线程 thread_d 释放锁资源处于阻塞状态，线程 thread_e 获得锁资源，在 5 秒中后，线程 thread_b 释放锁资源，同时进入阻塞状态。此时线程 thread_d 已经被唤醒处于就绪状态，等待抢夺 thread_b 释放的 共享变量锁 resourceA ，当 thread_d 重新抢夺获取资源后，线程 thread_d 重新进入运行状态。



## 关于 wait 的几种方法

~~~ java
    public final void wait() throws InterruptedException {
        wait(0);
    }
~~~

~~~ java
    public final native void wait(long timeout) throws InterruptedException;
~~~

```java
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
```

比较有意思的是 `wait(long timeout ,int nanos) ` 这个方法

官方给出的解释是，为了更加精准的控制并发的时间后面的 参数 nanos 只要 满足大于 0 并且小于 999999 这个条件，（1毫秒 = 1000 微秒 = 1000 000 纳秒）就在原先的等待时间上加 1 毫秒。

早起的 jdk 中关于 wait(long timeout ,int nanos) 方法

~~~ java
public final void wait(long timeout, int nanos) throws nterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    // nanos 单位为纳秒,  1毫秒 = 1000 微秒 = 1000 000 纳秒
    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                "nanosecond timeout value out of range");
    }

    // nanos 大于 500000 即半毫秒  就timout 加1毫秒
    // 特殊情况下: 如果timeout为0且nanos大于0,则timout加1毫秒
    if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
        timeout++;
    }

    wait(timeout);
}
~~~

【思考】

其实个人感觉，这里的 nanos 其实很没有意思，就目前常用的 1.8 JDK 中 wait 方法 nanos 参数的判断范围为（0-999999），那么只要传入的参数值在这个范围内，原先的延时时间都是加 1 毫秒，这就显得 nanos 这个参数无任何实质性的意义了。