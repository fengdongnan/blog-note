---
title: 技术-BigDecimal 使用须知  
description: 在 Java 开发中，`BigDecimal` 是处理高精度计算（特别是涉及货币、税率、财务等敏感数据）的首选工具，但因为它的工作机制与基本数据类型截然不同，使用不当非常容易踩坑。  
date: 2026-07-01  
tags:  
- 技术  
draft: false  
---

在 Java 开发中，`BigDecimal` 是处理高精度计算（特别是涉及货币、税率、财务等敏感数据）的首选工具，但因为它的工作机制与基本数据类型截然不同，使用不当非常容易踩坑。

以下是使用 `BigDecimal` 时最核心的 **8 个注意事项**：

## 1.  严禁使用 `double` 构造函数创建对象

绝对不要直接将 `double` 类型的浮点数传入构造函数，因为 `double` 本身精度丢失的问题会被带入 `BigDecimal`。

- **错误写法**：

  ```java
  BigDecimal number = new BigDecimal(0.1); 
  // 实际存储的值会变成 0.1000000000000000055511151231257827021181583404541015625
  ```

- **正确写法**：

  ```java
  // 方式一：使用 String 构造函数
  BigDecimal number1 = new BigDecimal("0.1");
  
  // 方式二：使用 BigDecimal.valueOf()（底层本质也是转成 String）
  BigDecimal number2 = BigDecimal.valueOf(0.1);
  ```

## 2.除法必须显式指定精度和舍入模式（RoundingMode）

使用 `divide()` 方法时，如果结果是无限循环小数（例如 $1 \div 3$），没有指定舍入规则的话，Java 会直接抛出 **`ArithmeticException`** 崩溃。

- **错误写法**：

  ```java
  BigDecimal a = new BigDecimal("1");
  BigDecimal b = new BigDecimal("3");
  BigDecimal result = a.divide(b); // 报错：ArithmeticException: Non-terminating decimal expansion
  ```

- **正确写法**：

  ```java
  // 明确保留几位小数（scale），以及采用什么舍入模式（如四舍五入 HALF_UP）
  BigDecimal result = a.divide(b, 2, RoundingMode.HALF_UP); // 结果：0.33
  ```

## 3.比较数值大小用 `compareTo()`，而不是 `equals()`

`BigDecimal` 的 `equals()` 方法极其严格，**不仅比较数值，还会比较 `scale`（精度/小数位数）**。

- **错误认知**：

  ```java
  BigDecimal num1 = new BigDecimal("1.0");
  BigDecimal num2 = new BigDecimal("1.00");
  
  boolean isEqual = num1.equals(num2); // false！因为 scale 一个是 1，一个是 2
  ```

- **正确写法**：

  ```java
  // compareTo 返回 0 代表数值大小相等
  boolean isSameValue = num1.compareTo(num2) == 0; // true
  ```

> **比较规范**：
>
> - `a.compareTo(b) > 0` 表示 $a > b$
> - `a.compareTo(b) == 0` 表示 $a = b$
> - `a.compareTo(b) < 0` 表示 $a < b$

## 4.`BigDecimal` 是不可变对象（Immutable）

就像 `String` 一样，所有对 `BigDecimal` 的数学计算（`add`, `subtract`, `multiply`, `divide` 等）**都不会改变原对象本身**，而是返回一个新的 `BigDecimal` 对象。

- **错误写法**：

  ```java
  BigDecimal total = new BigDecimal("10");
  total.add(new BigDecimal("5")); // 计算结果被丢弃了，total 依然是 10
  ```

- **正确写法**：

  ```java
  BigDecimal total = new BigDecimal("10");
  total = total.add(new BigDecimal("5")); // 必须接收返回值，total 变成 15
  ```

## 5.慎用 `toString()` 输出，推荐 `toPlainString()`

当 `BigDecimal` 的精度较大或数值太小/太大时，`toString()` 可能会默认格式化为**科学计数法**（例如 `0E-8` 或 `1E+7`），这可能不符合前端展示或接口入参的要求。

- **科学计数法输出**：

  ```java
  new BigDecimal("0.00000000").toString(); // 输出 "0E-8"
  ```

- **明确控制输出格式**：

  ```java
  // 1. 保留原始不带科学计数法的格式
  new BigDecimal("0.00000000").toPlainString(); // 输出 "0.00000000"
  
  // 2. 去除末尾无意义的 0 后输出
  new BigDecimal("1.2300").stripTrailingZeros().toPlainString(); // 输出 "1.23"
  ```

## 6.优先使用内置常量

对于一些常用的基础数字（如 `0`, `1`, `10`），建议直接使用 `BigDecimal` 预置的常量，无需重复创建对象，性能更好。

- **推荐写法**：
  - `BigDecimal.ZERO`
  - `BigDecimal.ONE`
  - `BigDecimal.TEN`

## 核心总结速查表

| **操作需求**   | **避坑原则**                    | **正确姿势**                                       |
| -------------- | --------------------------------- | ---------------------------------------------------- |
| **创建对象**   | `new BigDecimal(0.1)`             | `new BigDecimal("0.1")` 或 `BigDecimal.valueOf(0.1)` |
| **进行除法**   | `a.divide(b)`                     | `a.divide(b, 2, RoundingMode.HALF_UP)`               |
| **判断相等**   | `a.equals(b)`                     | `a.compareTo(b) == 0`                                |
| **执行计算**   | `a.add(b)` 忽略返回值             | `a = a.add(b)`                                       |
| **转为字符串** | `a.toString()` (可能带科学计数法) | `a.toPlainString()`                                  |