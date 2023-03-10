# jvm

```JAVA
import java.util.ArrayList;
import java.util.Random;

/**
 * @author peggy
 * @data 2023/2/6 9:08
 */
public class HeapMemoryTest {
    public void start() {
        ArrayList<ImageByteSpace> spaces = new ArrayList<>();
        while (true) {
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            spaces.add(new ImageByteSpace());
        }
    }

    public static void main(String[] args) {
        new HeapMemoryTest().start();
    }
}

class ImageByteSpace {
    byte[] data = new byte[new Random().nextInt(1024*200)];
}
```

![Snipaste_2023-02-06_09-56-38](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/Snipaste_2023-02-06_09-56-38.png)

![Snipaste_2023-02-06_09-53-52](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/Snipaste_2023-02-06_09-53-52.png)

![Snipaste_2023-02-06_09-55-57](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/Snipaste_2023-02-06_09-55-57.png)

- 设置堆空间的初始空间大小与最大空间大小以及显示堆空间的执行信息

  > -Xms5m -Xmx5m -XX:+PrintGCDetails

![image-20230206103932070](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230206103932070.png)

```java
import java.util.ArrayList;
import java.util.Random;

/**
 * @author peggy
 * @data 2023/2/6 9:08
 */
public class HeapMemoryTest {
    public void start() {
        ArrayList<String> spaces = new ArrayList<>();
        while (true) {
            String a="peggy";
            spaces.add(a); //循环生成新的字符串 占用堆空间的大小
            a+=a;
        }
    }

    public static void main(String[] args) {
        new HeapMemoryTest().start();
    }
}

class ImageByteSpace {
    byte[] data = new byte[new Random().nextInt(1024*200)];
}
```

![image-20230206104332067](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230206104332067.png)

可以看到的在前面的几次都是触发的为 YoungGen 回收之后的占用为 1024k —> 0k ，第一次触发 Full GC 的时候占用空间变化为 3287 K —> 1569 K ，第三次触发 Full GC 时占用的空间变化为 2938k —> 2017k 以及可回收的空间以及很少，随着字符串在堆中的不断产生，最终由于无法通过 Full GC 的回收来产生多余的空间，最终抛出 OutMemoryError 异常，JVM 停止。