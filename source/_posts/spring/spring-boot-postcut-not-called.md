---
title: kafka-snapshot-file
date: 2020-10-27 17:47:09
tags:
---
# spring boot @PostConstruct 不生效的问题
  工程里有依赖一个工程的其他jar，jar里有一个component 类，类里有一个初始化的方法，加了@PostConstruct 注解，但是怎么试都没生效
  网上查找了一阵，都是说Component没注解，或者是jdk版本的问题
  JDK版本看了下，jdk8，spring2.4，理论上是应该没问题的
  
  jdk版本问题，是如果是jdk1.9，spring又比较低，需要增加
  ```xml
       <dependency>
             <groupId>javax.annotation</groupId>
             <artifactId>javax.annotation-api</artifactId>
         </dependency>
  ```
  
  的依赖，或者升级spring
  
  但是排查后发现跟jdk没关系
  
  最后发现是因为包路径的问题，因为没有显式的注解scan，约定是application目录下的包会扫描，但是引入的jar跟我们工程的包路径不一致。
  jar里的是 com.clc.aaaaa.DemoComponent
  我工程里的application路径为：com.clc.bbbbb.DemoApplication
  所以扫描不到，
  将我工程里的application路径为提到外一层的包，改成com.clc.DemoApplication即可
  或者显式申明
    ```java
    @ComponentScan(value="com.clc.bbbbb")
    ```
  

