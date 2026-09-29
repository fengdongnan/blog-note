---
title: Enum 枚举类的 equals 问题  
description: 下午正好在看 java中 equals 相关的博客，到晚上就刚好遇到 equals 的判断问题了
date: 2026-07-20  
tags:  
- 技术  
draft: false  
---


下午正好在看 java中 equals 相关的博客，到晚上就刚好遇到 equals 的判断问题了

## 场景重现

```java
if (!XXXEnum.ORDER.equals(context.getCalculationScene())) {
    return false;
}
```

这里进行枚举场景的判断, 因为项目中原本的场景枚举判断都是这么写的, 就没有在意, 但是在debug的过程中发现即使 context.getCalculationScene() 的值是 ORDER, equals() 也会返回false, 导致拦截返回 false;  而预期是不应该命中这个条件的

下一步进行debug, 查看两个对比枚举值的具体地址和属性值, 可以看到属性值都是相同的, 但是地址值不一致


![image-20260722215339552](https://raw.githubusercontent.com/fengdongnan/image-bed/main/img/20260813203732814.png)

![image-20260722215250648](https://raw.githubusercontent.com/fengdongnan/image-bed/main/img/20260813203751195.png)
再回想equals 这个方法的实现, **默认是进行地址对比, 想要进行属性值对比,  要么在类中自己重写 equals() , 要么类中使用 Lombok 自动重写equals()**, 我跟进这个枚举类中都没找到上述二者, 难道是 Enum 像 String 一样自己重写了 equals ? 查看 Enum 源码:

![](https://raw.githubusercontent.com/fengdongnan/image-bed/main/img/20260813204001831.png)



还是没有, 虽然重写了, 但还是采用 == 进行比较

再看项目中其他用到这个枚举类的地方都是这么用的, 项目总不可能错着跑了这么久吧, 所以还是自己的问题. 终于在点进枚举类进行反跟调用处时找到了原因:

![image-20260722215534089](https://raw.githubusercontent.com/fengdongnan/image-bed/main/img/20260813204034259.png)

![image-20260722215620343](https://raw.githubusercontent.com/fengdongnan/image-bed/main/img/20260813204050727.png)

一个枚举只能跟到我调用的这里, 另一个枚举则可以跟到整个项目的其他调用处, 原来是因为两个项目中维护了同一个枚举类, 而我这里的import 错误import了其他项目中的包, 这样情理上首先说的通了, 至少我和大家保持一致了,但是根本原因呢? 为什么不进行equals 重写枚举类就可以直接使用 equals 进行属性比较? 这点在将 Enum 的 equals() 实现丢给 ai 后得到了答案

## Enum的特殊实现

> 它揭示了 Java 枚举（Enum）在底层的一个核心特性：**枚举比较 `.equals()` 和双等号 `==` 没有任何区别，底层直接就是 `this == other`**。
>
> 1.为什么用 `==` 就够了？
>
> 在 JVM 中，每个枚举常量都是**单例对象（Singleton）**。同一 JVM 内，无论你使用多少次某个枚举值，它在堆内存中都只有一个唯一的实例。因此，**比较引用地址（即 `==`）即可判定两者是否相等**。
>
> 2.为什么声明为 `final`？
>
> 方法前加上了 `final` 关键字，确保了任何自定义的枚举类型**都无法重写（Override）`equals` 方法**，强行保证了所有枚举的等值逻辑一致。

故, 答案明了, 因为 Enum采用的**单例模式**,  所以正常情况Enum 全局共享一个 A 对象, 因此直接采用 == 地址比较也就没问题. 而我这次遇到的import 外包中的枚举类, 等于是局外的另一个枚举类 B, 进行 == 比较自然也就不相等, 这点也体现在上述 debug 时二者的属性值相同, 但是**地址不同**

## 建议

开发时枚举比较优先使用 `==`，而不是 `.equals()`。这样在编写代码时编译器就会报错（无法比较两个不同类型的枚举），而不用等到运行时排查

```java
if (!(XXXEnum.ORDER == paramBO.getCalculationScene())) {
            return false;
}
```

原写法报错:

```json
Operator '==' cannot be applied to 'A.contract.enums.XXXEnum', 'B.contract.enums.XXXEnum'
```

