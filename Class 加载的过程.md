# Class 加载的过程

## ❤️‍🔥爱恨分明的 Java 与 C

![image-20230320130047306](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320130047306.png)

🌵 对于 C/C++ 而言大家最直观的感觉是什么？

我的感觉是在写 C 程序的时候，无时无刻的都在关注的内存的是怎么分配，变量而地址是怎么定义的。如何保证我的变量按照 ARM 手册定义的变量地址，在我调用的时候，如何很好的控制我硬件的 I/O 引脚的输出，而用 C/C++ 去写程序能更好的理解底层硬件的工作。

但对于 Java 而言一切却变得轻松了许多，不用去管内存具体如何分配，也不用去管和硬件底层如何对接。我们只需要，面向业务，面向逻辑的去编写复合场景需求的程序并实现具体的功能即可，具有这样的能力，对于初学者就比如笔者应付简单的日常“搬砖”是完全够的。但显示往往就是，但业务达到一定的规模时，逻辑也变得更负责，此时需要的不单单只是日常处理简单“搬砖”这么容易的事了，还需要学会思考如何提升“搬砖”的效率以及质量和解决问题的能力。

这就需要一个合格的代码编写者，不能单单只是停留在使用工具的表面，而要理解自己手里工具的原理，即便是工具“坏了”也能自己修理，格物更需致知。

Java 之所以方便是因为前辈们的努力，我们站在了前辈们铺好的路上，所以一切看起来好像是那么的容易简单。

为了打开 Java 的黑盒，我们就从其机器与代码之间的桥梁 <font color='green'>**虚拟机**</font> 开始叭

## 🗄️ Java 从编码到执行

听说过，当年 Java 当年的口号吗？<font color='cornflowerblue'>**“Write once , run anywhere”**</font>，一次编译随处运行。那么当年的 Sun 公司是怎么实现它这句口号的“野心”的呢？

如果有过 Linux 经验的伙伴应该都都知道，我们需要安装一些软件的时候，绝大多是的情况都是到官网去下载源码，然后通过调用 GCC 工具进行编译连接成可执行的文件。

即便是我们在 Windows 下编写的正常运行 HelloWorld 源代码，也是需要传输到 Linux 下重新编译后才可执行的。所以可以看到的是，很显然对于 C/C++ 源文件，每一次到不同的平台上运行都是需要重新编译、链接这样的一些列操作的。

但如果你在 windows 环境下，自己编写一份 HelloWorld 源代码在本地 build 之后，会有一个 .class 文件，将该文件无论是在 windows 环境还是 Linux 环境下，只需要在具有 Jdk 的环境下执行相同的 java xxxx.class 就会打印出相同的内容。

那么为什么 Java 就可以不像 C/C++ 语言一样繁琐，在不同而平台上运行之前必须先进行编译一遍，然后再执行输出呢？

这一切的功劳都是来是源于我们的虚拟机，当我们只要将编译好的 .class 文件交付于我们的虚拟机，虚拟机就会按照指定的格式进行解析执行，通过与 C/C++ 的对接调用底层的硬件。

![image-20230320125702861](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320125702861.png)

## 🚧从跨平台的语言到跨语言的平台

JVM 或许我们只是单单觉得它绝大多数的情况下都是用于 Java 程序的运行，那它能不能兼容其他程序呢？其实是可以的，这里说兼容应该说是有点不恰当的，我们应该是说包纳能更形象一些。

![image-20230320135512991](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320135512991.png)

其实 JVM 发展到现如今，可以包纳的语言其实已经很多了，以致于你按照 JVM 的规范自己定义一套专属的语言，也是可以通过 JVM 运行的。

不信，我们接着看一个编译后的生产的 .class 文件的真面目到底是什么？

![image-20230320135929223](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320135929223.png)

编写一个空类，编译后再 target 目录下生成一份 .class 文件

![image-20230320140047144](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320140047144.png)

相比如源码图中可以看多了一个空参构造方法，当然这个 .class 文件并不是最原始的 .class ，我们通过安装的一个 IDEA 插件 <font color='cornflowerblue'>**BinEd-Binary**</font> 进制转化工具查看。

![image-20230320140337755](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320140337755.png)

![image-20230320140656150](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320140656150.png)

![image-20230320140809790](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230320140809790.png)

而当我们看到如图中所示的这个的时候，这才是编译后真正的 .class 源文件。它本身其实就是一组 0/1 编码的文件，而当 JVM 在执行的时候，就会按照编码的规律一次取出执行。

那么这么一堆看起来混乱的编码 JVM 具体是怎么识别的呢？这个就得从虚拟机的具体实现，这里就以为开源免费的 Hotspot 为例说明。

也就是说，我们可以按照某种规则，将自己任何一种语言进行处理，变成能被 JVM 识别的，class 文件即可，实现自定义的语言跨平台。

## 🧗‍♂️神秘编码的背后都是什么

