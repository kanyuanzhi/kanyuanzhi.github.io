---
title: 《Head First模式设计》学习笔记
tags: 设计模式
---

# Design Patterns Learning

《Head First模式设计》中的代码实践
<!--more-->

## 设计原则

- 找出应用中可能需要变化之处，把他们独立出来，不要和那些不要变化的代码混在一起。
- 针对接口编程，而不是针对实现编程。
- 多用组合，少用继承。
- 为了交互对象之间的松耦合而努力。
- 类应该对扩展开放，对修改关闭。

## 模式分类

### 策略模式
策略模式定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

### 观察者模式
观察者模式定义了对象之间一对多的关系，一个Subject对应多个Observe，Subject用一个共同的接口来更新Observes。
Observe利用Subject的接口向Subject注册，Subject利用Observe的接口通知Observe。
主题和观察者之间用松耦合的方式结合，主题不知道观察者的细节，只知道观察者实现了观察者接口。
```java
public interface Oberver{
    public void update();
}

public interface Subject{
    public void registerObserver(Observer observer);
    public void removeObserver(Observer observer);
    public void notifyObserver();
}
```
实现Subject的主题中维护一个与其相关的观察者列表：
```java
private ArrayList<Observer> observers;
```
![image](https://github.com/kanyuanzhi/design-patterns-learning/raw/master/docs/observe.png "观察者模式UML图")

### 装饰者模式
