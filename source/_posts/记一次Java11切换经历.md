# 记一次Java11切换经历

### 系统切换(Arch Linux)

##### JDK下载

-  oracle官网下载[orcale-jdk](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html)(源码带文档注释)

- 源安装openjdk,运行`sudo pacman -S jdk11-openjdk `(idea查看源码反编译无注释),可运行`sudo pacman -S openjdk11-doc`增加稳定

##### JDK版本切换

通过源安装的open-jdk直接运行`sudo archlinux-java set java-8-openjdk`切换.

orcalce下载的jdk,通过将下载的jdk移动到`/usr/lib/jvm/`下,注意命名为java-\${JAVA_MAJOR_VERSION}-${VENDOR_NAME},这时通过`archlinux-java status`命令查看会发现新增的JDK,然后设置.

```bash
 ~ sudo cp -r app/jdk-11.0.4 /usr/lib/jvm/java-11-oraclejdk
 ~ archlinux-java status                         ✔  14:50:02
Available Java environments:
  java-11-oraclejdk (default)
  java-8-openjdk
 ~ sudo archlinux-java set java-11-oraclejdk     ✔  14:50:06
 ~ java -version                                 ✔  14:50:29
java version "11.0.4" 2019-07-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.4+10-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.4+10-LTS, mixed mode)
```

### IDEA配置

1.设置项目JDK

![](https://www.guohezuzi.cn/public/img/blog/java11-1.png)

2.设置模块JDK

![](https://www.guohezuzi.cn/public/img/blog/java11-2.png)

3.maven配置

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
        <release>11</release>
    </configuration>
</plugin>
```

### 新特性使用

1.Java 解释器直接执行 Java 源代码

写一个简单的程序,计算个人存活及死亡时间

```java
import java.time.LocalDate;
import java.time.temporal.ChronoUnit;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: zuzi
 * \* Date: 2019-07-29
 * \* Time: 上午9:24
 * \* Description: 向死而生
 * \
 */
public class LiveTowardsDeath1 {
    public static void main(String[] args) {
        liveTowardsDeath(1999,2,24,80);
    }

    private static void liveTowardsDeath(int year,int month,int day,int liveYear){
        LocalDate birth = LocalDate.of(year, month, day);
        LocalDate death = LocalDate.of(year+liveYear, month, day);
        long lifeDays = ChronoUnit.DAYS.between(birth, LocalDate.now());
        long deathDays = ChronoUnit.DAYS.between(LocalDate.now(), death);
        System.out.println("已存活: "+lifeDays+"天");
        System.out.println("距离死亡: "+deathDays+"天");
    }
}

```

运行 

```bash
~ java LiveTowardsDeath1.java                   ✔  15:29:42
已存活: 7461天
距离死亡: 21759天
```

2.作为脚本运行

修改.java文件为.sh并将内容修改如下:

```bash
#!java --source 11

import java.time.LocalDate;
import java.time.temporal.ChronoUnit;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: zuzi
 * \* Date: 2019-07-29
 * \* Time: 上午9:24
 * \* Description: 向死而生
 * \
 */
public class LiveTowardsDeath0 {
    public static void main(String[] args) {
        liveTowardsDeath(1999,2,24,80);
    }

    private static void liveTowardsDeath(int year,int month,int day,int liveYear){
        LocalDate birth = LocalDate.of(year, month, day);
        LocalDate death = LocalDate.of(year+liveYear, month, day);
        long lifeDays = ChronoUnit.DAYS.between(birth, LocalDate.now());
        long deathDays = ChronoUnit.DAYS.between(LocalDate.now(), death);
        System.out.println("已存活: "+lifeDays+"天 ");
        System.out.println("距离死亡: "+deathDays+"天");
    }
}

```

运行

```bash
 ~ chmod u+x LiveTowardsDeath.sh                 ✔  15:34:45
 ~ ./LiveTowardsDeath.sh                         ✔  15:35:25
已存活: 7461天 
距离死亡: 21759天
```

### 参考

[arch-linux文档](https://wiki.archlinux.org/index.php/Java_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
