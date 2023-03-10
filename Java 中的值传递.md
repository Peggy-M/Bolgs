# Java 中的值传递

在张洪亮老师写的《深入理解 Java 核心技术》中，看到了关于`值传递`的解释，这里加入，自己的有关 C/C++ 指针这块的些许理解，做一个记录。

首先抛出一个问题，对于 Java 中对象之间的交互你有几种理解？

- 值传递与引用传递的区分条件是传递的内容，如果传递内容是值，就是值传递，如果内容是值引用就是引用传递
- Java 的传递就是引用传递
- 传递的如果是普通类型的值，就是值传递，传递的是对象，就是引用传递

其实在张洪亮老师指出，以上的观点都是错误的！！！

**为了验证以上错误确实是错误，我们可以参看以下几个代码进行分析。**

```java
/**
 * @author peggy
 * @data 2023/2/15 10:06
 */
public class Test {
    public static void main(String[] args) {
        Bank bank = new Bank();
        User user = new User("卡卡罗特",1000);
        bank.getBalance(user);
        System.out.println("main方法==》"+"用户:"+user.getName()+" 余额:"+user.getBalance());
    }
}

class Bank {
    public void getBalance(User user){
        System.out.println("getBalance方法==》"+"用户:"+user.getName()+" 余额:"+user.getBalance());
    }
}》
```

~~~ java
/**
 * @author peggy
 * @data 2023/2/15 10:03
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String name;
    private Integer balance;
}
~~~

> **getBalance方法==》用户:卡卡罗特 余额:1000**
>
> **main方法==》用户:卡卡罗特 余额:1000**

首先 main 方法中，将 User 对象作为实参，传递给了 Back 对象中 getBalance() 的形参，最终打印信息。

我们现在做一个修改，再次运行；

```java
class Bank {
    public void getBalance(User user){
        user.setBalance(500);
        System.out.println("getBalance方法==》"+"用户:"+user.getName()+" 余额:"+user.getBalance());
    }
}
```

> **getBalance方法==》用户:卡卡罗特 余额:500**
> **main方法==》用户:卡卡罗特 余额:500**

可以看到的是，我们在 getBalance 中修改了 user 形参的值，对于 main 方法中的实参，也同时发生了相同的改变。

所以可以断定，形参与实参其实都是指向同一个 User 对象的

那么问题就是，我们在传递的过程中传递的到底是什么呢？是值，还是对象引用，还是共享对象传递呢？

这个容易，我们只需要打印出形参与实参的 **<font color='cornflowerblue'>地址</font>**，即可验证传递的到底是什么

**<font color='orange'>上代码>>>></font>** 

我们这里取消掉代码中的 lombok 表达式

```java
/**
 * @author peggy
 * @data 2023/2/15 10:06
 */
public class Test {
    public static void main(String[] args) {
        Bank bank = new Bank();
        User user = new User("卡卡罗特",1000);
        bank.getBalance(user);
        System.out.println("main==》地址: "+user.hashCode());
        System.out.println("main方法==》"+"用户:"+user.getName()+" 余额:"+user.getBalance());
    }
}

class Bank {
    public void getBalance(User user){
        System.out.println("getBalance方法==》地址: "+user.hashCode());
        user.setBalance(500);
        System.out.println("getBalance方法==》"+"用户:"+user.getName()+" 余额:"+user.getBalance());
    }
}
```

> **getBalance方法==》地址: 666641942**
> **getBalance方法==》用户:卡卡罗特 余额:500**
> **main==》地址: 666641942**
> **main方法==》用户:卡卡罗特 余额:500**

可以发现 getBalance 与 main 方法中，指向的对象都是同一个，它们的 hashcode 相同。

所以其实 Java 在进行 <font color='cornflowerblue'>形参 </font>与 <font color='cornflowerblue'>实参</font>，又或者是在进行对象变量，在进行指向的过程中，都是传递的是一个地址值。

![image-20230215110530628](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230215110530628.png)

但如果用 C/C++ 去解释上面的代码得到的结果是相反的 

```c++
#include <iostream>
#include "Back.h"

int main() {
    User user = *new User("Kakarotto", 1000);
    Back back = *new Back();
    back.getBalances(&user);
    cout << "userName:" << user.getName() << " balance:" << user.getBalance() << endl;
    cout << "getBalance site " << &user << endl;
    return 0;
}
```

```c++
/**
* @author peggy
* @data 2023/2/15 11:13
*/

//
// Created by peggy on 2023/2/15.
//

#include "Back.h"
#include <iostream>

void Back::getBalances(User *user) {
    cout << "userName:" << user->getName() << " balance:" << user->getBalance() << endl;
    cout << "getBalance site " << user << endl;
}
```

```c++
/**
* @author peggy
* @data 2023/2/15 11:06
*/

//
// Created by peggy on 2023/2/15.
//

#include "User.h"

const string &User::getName() const {
    return name;
}

void User::setName(const string &name) {
    User::name = name;
}

int User::getBalance() const {
    return balance;
}

void User::setBalance(int balance) {
    User::balance = balance;
}

User::User() {}

User::User(const string &name, int balance) : name(name), balance(balance) {}
```

> **userName:Kakarotto balance:1000**
> **getBalance site 0xd3ab3ffb10**
> **userName:Kakarotto balance:1000**
> **getBalance site 0xd3ab3ffb10**

从输出结果结果可以看到，都是指向同一个地址

所以对于 C/C++ 中的值传递其实是不同的，getBalance 的执行，取得是实参的地址

![image-20230215141301566](https://peggy-note.oss-cn-hangzhou.aliyuncs.com/images/image-20230215141301566.png) 

所以可以看的是 Java 与 C/C++ 两者是有不同的

***开启掘金成长之旅！这是我参与「掘金日新计划 · 2 月更文挑战」的第 3 天***