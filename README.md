
## 概述


**Java SPI**（Service Provider Interface）是一种 **服务发现机制**，用于实现模块化、可插拔式的设计。在 Java 中，它允许程序在运行时动态地加载和调用实现类，而不是在编译时硬编码依赖。这种机制在 **JDK 内置库** 和 **第三方库** 中被广泛使用，例如 JDBC 驱动加载、日志框架绑定（如 SLF4J 和 Logback）、序列化机制扩展等。


## **SPI 的核心概念**


1. **服务接口（Service Interface）**
定义服务的规范，提供一个接口或抽象类。
2. **服务提供者（Service Provider）**
一个实现了服务接口的具体类。
3. **服务加载器（Service Loader）**
用于动态加载实现服务接口的服务提供者类。


## **SPI 的工作机制**


Java SPI 的实现依赖于 `resources/META-INF/services` 文件夹中的描述文件。主要过程如下：


1. **定义服务接口：** 创建一个服务接口，定义公共方法。
2. **创建服务提供者：** 编写实现服务接口的具体类。
3. **配置服务提供者：** 在 `META-INF/services` 文件夹中，创建一个文件，文件名是服务接口的全限定类名，内容是服务提供者的全限定类名。
4. **通过 `ServiceLoader` 加载服务：** 使用 `java.util.ServiceLoader` 动态加载实现类。


## **Java SPI 示例**


我的文件结构定义如下：



```
/src/
    ├── test/
    	├── java/
    		├── spi/
    			├── example/
    				├── MyService		# SPI接口
    				├── SericeA			# SPI接口A实现
    				├── SericeB			# SPI接口B实现
    				├── SPIServiceLoader # SPI加载器
    ├── resources/
    	├── META-INF/
    		├── services/
    			├── spi.example.MyService	# 资源文件

```

### **定义服务接口**


创建一个服务接口 `MyService`：



```
package spi.example;

public interface MyService {
    void execute();
}

```

### 创建服务提供者


创建ServiceA、SeriviceB两个类，然后重写excute代码



```
package spi.example;

public class ServiceA implements MyService {
    @Override
    public void execute() {
        System.out.println("ServiceA is executing...");
    }
}

```


```
package spi.example;

public class ServiceB implements MyService {
    @Override
    public void execute() {
        System.out.println("ServiceB is executing...");
    }
}

```

### **配置服务提供者**


在 `resources/META-INF/services` 目录下，创建一个文件，文件名为 `spi.example.MyService`，内容为：



```
spi.example.ServiceA
spi.example.ServiceB

```

使用 `ServiceLoader` 加载服务


在主程序中，通过 `ServiceLoader` 动态加载实现类：



```
package spi.example;

import java.util.ServiceLoader;
public class SPIServiceLoader {
    public static void main(String[] args) {
        ServiceLoader loader = ServiceLoader.load(MyService.class);

        for (MyService service : loader) {
            service.execute();
        }
    }
}

```

### 运行结果


![image-20241126224757183](https://images.cnblogs.com/cnblogs_com/erosion2020/2432063/o_241126145514_spi1.png)


## SPI恶意代码执行


比如我们在spi.example包中新增一份恶意代码的MyService实现，如下：



```
package spi.example;

public class EvilService implements MyService{
    public EvilService(){
        try {
            System.out.println("EvilService constructor is executing...");
            Runtime.getRuntime().exec("calc");
        }catch (Exception ignore) { }
    }
    @Override
    public void execute() {
        System.out.println("EvilService is executing...");
    }
}

```

运行结果如下，可以看到恶意代码被执行：


![image-20241126225345244](https://images.cnblogs.com/cnblogs_com/erosion2020/2432063/o_241126145514_spi2.png)


## SPI 的缺点


* 性能问题： 每次调用 ServiceLoader 都需要扫描 META\-INF/services 下的文件，可能影响性能。
* 缺乏优先级支持： 多个服务提供者时，SPI 无法原生支持加载优先级。
* 安全性问题： 攻击者可能通过篡改 META\-INF/services 文件加载恶意类。


### 增强版 SPI


为了解决上述缺点，现代框架（如 Spring）提供了增强的服务发现机制。例如：


* Spring 使用 @Component 和 @Autowired 自动注入服务。
* Google Guice 和 Apache Dubbo 也扩展了类似的机制，支持更加灵活的依赖注入和服务加载。
* SPI 是 Java 生态中非常重要的机制，在理解其原理的基础上，可以结合实际场景选择更适合的方案。


  * [概述](#%E6%A6%82%E8%BF%B0)
* [SPI 的核心概念](#spi-%E7%9A%84%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5)
* [SPI 的工作机制](#spi-%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6)
* [Java SPI 示例](#java-spi-%E7%A4%BA%E4%BE%8B)
* [定义服务接口](#%E5%AE%9A%E4%B9%89%E6%9C%8D%E5%8A%A1%E6%8E%A5%E5%8F%A3)
* [创建服务提供者](#%E5%88%9B%E5%BB%BA%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E8%80%85)
* [配置服务提供者](#%E9%85%8D%E7%BD%AE%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E8%80%85)
* [运行结果](#%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C)
* [SPI恶意代码执行](#spi%E6%81%B6%E6%84%8F%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C):[Flowercloud 机场订阅加速](https://flowercloud6.com)
* [SPI 的缺点](#spi-%E7%9A%84%E7%BC%BA%E7%82%B9)
* [增强版 SPI](#%E5%A2%9E%E5%BC%BA%E7%89%88-spi)

   \_\_EOF\_\_

       - **本文作者：** [Erosion2020](https://github.com)
 - **本文链接：** [https://github.com/erosion2020/p/18571153](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
