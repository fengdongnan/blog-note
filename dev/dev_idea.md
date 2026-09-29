---
title: 开发环境配置 - idea  
description:  
date: 2026-07-26 
tags:  
- 配置  
draft: false  
---

## idea配置

idea全局搜索失效

https://www.cnblogs.com/chengho/p/14165962.html



#### 开启鼠标滚轮调节代码字体大小

 进入 Editor -> General，勾选 Change font size with Ctrl+Mouse Wheel 

#### 开启显示代码行号和方法分隔线

进入 Editor -> General -> Appearance，勾选 Show method separators（在方法之间绘制一条虚线分隔，增强代码可读性）。

#### 多层标签页, 标签页自动关闭

进入设置：File -> Settings -> Editor -> General -> Editor Tabs

- 开启多行标签页: Show tabs in 改为 Multiple rows
- 自动关闭不常用标签页: 同一页面下 Closing Policy 的 Tab limit 修改为 15

#### idea 提示不区分大小写

File -> Settings -> Editor -> General -> Code Completion，取消勾选 Match case

#### 取消 import 自动合并

File -> Settings -> Editor -> Code Style -> Java -> Imports，将 Class count to use import with '*' 和 Names count to use static import with '*' 改为 999

#### 自动删除无效 import

1.File -> Settings -> Editor -> General -> Auto Import。

2.在右侧 Java 区域中，勾选 Optimize imports on the fly (for current project) 选项

（补充：如果想在保存文件时自动清除，可依次展开 Tools -> Actions on Save，勾选 Optimize imports 后保存即可。）

#### 注释颜色设置

1.File -> Settings ->  Editor -> Color Scheme -> Java ->  Comments

2.根据需要点击具体的注释类型，例如 Line comment（单行注释）、Block comment（多行注释）或 JavaDoc（文档注释）。

3.在最右侧取消勾选 Foreground 右侧的 Inherit values from 选项，然后点击颜色方块选择



## idea插件

Rainbow Brackets - 彩虹括号