---
title: Java基础之转型
date: 2016-05-20 14:43:49
tags: [java,java基础]
categories: [java,java基础]
---
上转型对象
父类的声明指向子类对象（最无争议的说法）
父类 对象名 = new 子类构造方法();
(1)该对象可以调用子类重写父类的方法(本质)
(2)该对象不能调用子类独有的方法
(3)上转型对象可以强制转化成子类对象 (进而访问子类独有的方法)
父类 :     Person
子类 :Teacher    Student
Person person = new Teacher();
Studnet student= (Student)person;
在企业开发的时候,当别人给你传递一个对象的时候,如果对象的类型,不是很确定,要先测试一下，instanceof :java中的一个关键字,专门用来进行对象 类型的测试,跟强制类型转化,经常结合使用
    if(person instanceof Student2){
      Student2 p2 =(Student2) person;
