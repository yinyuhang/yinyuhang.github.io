---
layout: post
title: "为什么 Jdk 会自动为注解创建代理类"
subtitle: "搬运"
date: 2019-02-08 15:11:05
catalog: true
tag: 
    - 翻译
    - Java
---
### 问题
[原文地址](https://stackoverflow.com/questions/54285043/does-jdk-automatically-generate-proxy-classes-for-custom-annotations)
```
@Retention(RetentionPolicy.RUNTIME)
@Target(value={ElementType.TYPE})
@Documented
public @interface MyAnnotation {
}
```
```
@MyAnnotation
public class Main {
    public static void main(String[] args) {
        Class<Main> cls = Main.class;
        Annotation[] annotations = cls.getDeclaredAnnotations();
        Arrays.stream(annotations).forEach(an -> {
            if (an instanceof MyAnnotation) {
                System.out.println("proxy");
            } else {
                System.out.println("???");
            }
            System.out.println(an.getClass().getName());
        });
    }
}
```
我在 Windows Centos7 和 Ubuntu上都得到了同样的结果：
```
proxy
com.sun.proxy.$Proxy1
```
我的问题是：这是我的环境配置有问题还是JDK会自动生成一个代理类？如果是后者，为什么？

---------------------------

### 回答
参见 [9.6. Annotation Type](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.6)
> 一个注解的声明指定了一个新的注解类型，同样也是一种特殊的接口。

而接口无法直接实例化，所以从反射的API上获取的信息来看，不管是用代理、匿名类或者其他方法，必须有个对象去实现这个接口。具体要参考不同的JVM实现。

-------------------------

### 译者注
值得注意的是，这里使用了 ```getClass()``` 来获取类信息。Jvm必须生成一个实例才能调用该方法。如果使用 ``` MyAnnotation.class ``` 获取类信息的话，得到的结果将是 ``` interface com.shark.test.MyAnnotation ```
另外打个马后炮，其实这个问题，在查阅文档后，完全可以逻辑推出。首先，已知的条件有：
* 非静态方法必须通过某个非 null 实例调用(否则就是熟悉的 NullPointerException 了哈哈哈)
* 注解是一种特殊的接口
推出-> 那么JVM是如何获取一个接口的实例的
推出-> 通过接口来获取实例，已知的Java技术有动态代理，匿名内部类

这里因为没有仔细看文档，对条件2并不知情，所以有种JVM对注解做了特殊处理，但是原因不明的想法，就钻进死胡同了QAQ
