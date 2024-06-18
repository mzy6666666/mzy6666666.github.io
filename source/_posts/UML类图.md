---
title: UML类图
date: 2024-06-18 14:48:20
tags: uml
---

## UML类图

> **类图基础属性**
>
> +表示public
>
> -表示private
>
> #表示protected
>
> ~表示default
>
> _表示static
>
> 斜体表示抽象
>
> **类图之间关系**
>
> ![image-20240607165802362](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607165802362.png)
>
> 1. 继承：继承表示是一个类（称为子类、子接口）继承另外的一个类（称为父类、父接口）的功能，并可以增加它自己的新功能的能力。
>
>    表示方法：继承使用空心三角形+实线表示。
>
>    ![image-20240607170103069](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607170103069.png)
>
> 2. 实现：实现表示一个class类实现`interface`接口（可以是多个）的功能。
>
>    表示方法：使用空心三角形+虚线表示
>
>    ![image-20240607170209590](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607170209590.png)
>
> 3. 依赖：对于两个相对独立的对象，当一个对象负责构造另一个对象的实例，或者依赖另一个对象的服务时，这两个对象之间主要体现为依赖关系。
>    表示方法：依赖关系用虚线箭头表示
>
>    ![image-20240607170316317](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607170316317.png)
>
> 4. 关联：对于两个相对独立的对象，当一个对象的实例与另一个对象的一些特定实例存在固定的对应关系时，这两个对象之间为关联关系。
>
>    表示方法：关联关系用实线箭头表示
>
>    ![image-20240607170403521](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607170403521.png)
>
> 5. 聚合：表示一种弱的‘拥有’关系，即`has-a`的关系，体现的是A对象可以包含B对象，但B对象不是A对象的一部分。 两个对象具有各自的生命周期。
>
>    表示方法：聚合关系用空心的菱形+实线箭头表示
>
>    ![image-20240607170518709](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607170518709.png)
>
> 6. 组合：组合是一种强的‘拥有’关系，是一种`contains-a`的关系，体现了严格的部分和整体关系，部分和整体的生命周期一样。
>    表示方法：组合关系用实心的菱形+实线箭头表示，还可以使用连线两端的数字表示某一端有几个实例
>
>    ![image-20240607170613673](https://mzy777.oss-cn-hangzhou.aliyuncs.com/img/image-20240607170613673.png)


