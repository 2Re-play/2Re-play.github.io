---
layout: post
comments: true
title:  "SPRING"
subtitle: "Bean, Bean Scope, Spring Annotation(의존성 주입)"
categories: KeyPoint
tags: KeyPoint
author: jaehyun
background: '/img/posts/06.jpg'
---
* content
{:toc}

---

### Bean 이란?

---

<div>
<img width="400" height = "250" alt="img2" src="https://user-images.githubusercontent.com/22942313/66982192-9d192980-f0f0-11e9-9ad1-fd5b24c21d75.png">
</div>

Spring을 사용하다보면 bean 이라는 용어가 많이 나오는데 bean에 대해 알아보자.

* 상속을 받지않고 가장 기본적인 field와 getter, setter만 가지고 있는 객체가 bean이다.
* Spring에서 POJO(plain old java object)는 beans라고 불린다.
* bean을 생성하기 위해서는 xml config file의 설정을 통해 Spring container에 의해 생성된다.
* xml config file에서는 bean의 lifecycle과 dependency를 설정할 수 있다.
* Spring container에서 객체를 생성해주기 때문에 개발자가 new 생성자로 객체를 생성할 필요가 없다.
* container에 의해 생성된 bean에 접근하기 위해서는 getBean("[bean_id]")로 통해 bean에 대한 reference 값을 가져올 수 있다.

### Spring에서 Bean의 속성

---

* Key attributes
  - class(required) : 클래스의 이름을 속성에 반드시 넣어줘야한다.
  - id : 해당 bean의 유니크한 값
  - scope : 객체의 scope (singleton, prototype) **singleton에 대해서 추가공부**
  - constructor-arg : bean을 생성할 때 constructor에게 넘겨줄 인자, 이를 통해 의존성을 주입한다.
  - property : bean을 생성할 때 setter메소드를 호출하고 setter의 인자값으로 value로 넘겨준다.
  - init, destroy method

  
```xml
<!-- A simple bean definition -->
<bean id="..." class="...">
</bean>

<!-- A bean definition with scope-->
<bean id="..." class="..." scope="singleton">
</bean>

<!-- A bean definition with property -->
<bean id="..." class="...">
	<property name="message" value="Hello World!"/>
</bean>

<!-- A bean definition with initialization method -->
<bean id="..." class="..." init-method="...“ >
</bean>
```  
  

### Spring Bean의 Scope

---
  
  
| Bean Scope | 설명 |  
|:--------:|:--------:|
| singleton | bean을 단 하나만 생성하여 값을 공유한다(default scopt) |
| prototype | 매번 새로운 요청 마다 bean을 생성하며 값을 공유하지 않는다. |
| request | HTTP request의 하나의 bean  |
| session | HTTP session의 하나의 bean |
| global-session | gobal HTTP session의 하나의 bean |  



* Bean은 2가지의 type을 가지고있다.
  - singleton
  - prototype
* 'singleton'으로 설정한 객체는 Spring container에서 **단 한번** 생성된다.
  - singleton 객체는 언제 생성되고 언제 수거될까?
    - Spring container가 생성될 때 singleton 객체가 생성되고 container가 종료될 때 garbage collercter에 의해 수거된다.

* 'prototype'으로 설정한 객체는 요청 마다 새로운 객체를 생성한다.
  - 더 이상 사용하지 않는 prototype 객체는 garbage collercter가 메모리를 회수한다.

* 객체의 scope를 정의하지 않을 경우 scope default값이 **singleton**으로 객체가 생성된다.

### Dependency Injection methods

---
의존성주입을 하는 방법은 2가지가 있다.

* 생성자를 통해 의존성 주입
* property의 setter를 통해 의존성 주입

### Spring Annotation을 사용하여 설정

---

* Spring 2.5버전 이후로 등장하여 요즘 많이 사용하는 방법이다.
* xml configuration size를 줄이기 위한 목적으로 annotation이 나왔다.
* bean wired을 위해 xml을 통해서 설정하는 방법의 대체재로 사용된다.
* bean의 configuration을 class 자체에 옮겨 class 안에서 직접 의존성을 주입한다.
* xml을 사용하게 되면 bean을 설정할 때 마다 계속해서 설정을 넣어줘야하는데 annotation으로 좀 더 명시적이고 보기좋은 코드를 만들어준다.
* xml과 annotation을 둘다 사용한다면 xml이 annotaion을 override하게 된다, 즉! xml이 우선순위다.

그렇다면 대표적인 Spring Annotation 몇가지에 대해 알아보자.

첫번째로 Spring Annotation을 사용하기 위해서 Spring Annotation을 활성화 시켜줘야한다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"
    ...
    xmlns:context="http://www.springframework.org/schema/context"
    ...

   <context:annotation-config/>
```
* xmlns:context="http://www.springframework.org/schema/context"
* <context:annotation-config/>

위 2개의 코드를 web.xml에 추가시켜 Spring Annotation을 활성화 시켰다.

#### @Required Annotation

setter 메소드에서 인자를 반드시 받아야할 경우 사용하는 어노테이션
```java
public class Boy {
 private String name;
 private int age;

 @Required
 public void setName(String name){
  this.name = name;
 }

 @Required
 public void setAge(int age){
  this.age = age;
 }

 //   getters ...
}
```
위 코드에서 setName, setAge 메소드에 @Required 어노테이션이 설정되어 있기 때문에,
name과 age는 반드시 필수로 주입시켜줘야한다.

```xml
<bean id="boy" class="Boy">
 <property name="name" value="Rony"/>
 <property name="age" value="10"/>
</bean>
```
위의 의존성 주입은 @Required로 설정된 name과 age를 두개다 **필수**로 주입해줘야한다.
```xml
<!-- Property 'age' is required for bean 'boy' -->
<bean id="boy" class="Boy">
 <property name="name" value="Rony"/>
</bean>
```
위의 의존성 주입은 @Required로 설정된 age를 주입해주지 않았기 때문에 **에러**가 발생한다.

#### @Autowired Annotation

자동적으로 의존성주입을 시키기 위한 어노테이션

1) xml을 사용하여 의존성을 주입하는 방법
```java
public class Boy {
 private String name;
 private int age;

 //   getters and setters ...
}
```

```java
public class College {

 private Boy student;

 public void setStudent(Boy aboy){
  this.stuendt = aboy;
 }
}
```

```xml
<bean id="boy" class="Boy">
 <property name="name" value="Rony"/>
 <property name="age" value="10"/>
</bean>

<bean id="college" class="College">
  <property name="student" ref="boy"/>
</bean>
```

xml파일에서 <property name="student" ref="boy"/> property의 ref값에 주입시킬 bean의 id를 명시하여 의존성주입을 한다.

#### @Autowired
2) 어노테이션을 사용하여 의존성을 주입하는 방범

type에 의해서 자동적으로 의존성을 주입해준다.

```java
public class Boy {
 private String name;
 private int age;

 //   getters and setters ...
}
```

```java
public class College {

 @Autowired
 private Boy student;
}
```

```xml
<bean id="boy" class="Boy">
 <property name="name" value="Rony"/>
 <property name="age" value="10"/>
</bean>

<bean id="college" class="College">
</bean>
```
College 클래스의 student 필드에 **@Autowired** 어노테이션을 사용하여 Boy라는 type을 가진 bean을 주입한다.

##### 하지만!!, 만약 같은 type을 가진 bean이 여러개일때는 Spring container가 값을 어떻게 주입할까?? 이 경우에는 Spring Container는 값을 주입하지 않고 에러를 발생시킨다.  이 문제를 @Qualifier 어노테이션을 사용하여 해결할 수 있다.


#### @Qualifier

위에서 말한 상황에서 마법사같이 해결해줄 수 있는 어노테이션이 @Qualifier다.
같은 type의 객체가 1개 이상일 경우 어떠한 bean을 주입할지 모호하기 때문에
qulifier value를 통해 구분하여 자동적으로 주입하게 한다.


```java
public class Boy {
 private String name;
 private int age;

 //   getters and setters ...
}
```
```java
public class College {

@Autowired
@Qualifier(value="tony")
 private Boy student;

  //   getters ...
}
```

```xml
<bean id="boy1" class="Boy">
 <qualifier value="rony"/>
 <property name="name" value="Rony"/>
 <property name="age" value="10"/>
</bean>

<bean id="boy2" class="Boy">
 <qualifier value="tony"/>
 <property name="name" value="Tony"/>
 <property name="age" value="8"/>
</bean>

<bean id="college" class="College">
</bean>
```
College 클래스에서 @Qualifier(value="tony")로 어노테이션을 추가하였다.
Qualifier는 type으로 bean을 찾아 의존성을 주입하지 않고 <qualifier value="tony"/>의 value값으로 구분해서 주입한다.

#### @Resouce

@Autowired 어노테이션과 기능은 비슷하지만 다른 점이 있다.
@Autowired 는 bean의 type에 따라 의존성주입을 하지만, @Resource 어노테이션은 beand의 unique한 id값을 통해 해당 bean을 의존성주입한다.
그렇다면, 어차피 @Autowired와 기능도 똑같고 같은 type의 bean이 여러개일 때 걱정도 안해도 되는 @Resource 어노테이션보다 왜 @Autowired 어노테이션을 더 많이 사용할까?
그 답은 Spring에서는 대부분 singleton bean으로 단 하나의 객체만을 생성하기 때문에 @Autowired 어노테이션을 더 자주 사용한다.

```java
public class College {

  @Resource(name="boy1")
  private Boy student;

  //   getters and setters ...
}
```

```xml
<bean id="boy1" class="Boy">
 <property name="name" value="Rony"/>
 <property name="age" value="10"/>
</bean>

<bean id="boy2" class="Boy">
 <property name="name" value="Tony"/>
 <property name="age" value="8"/>
</bean>

<bean id="college" class="College">
</bean>
```

xml파일에서 bean의 unique한 id값을 @Resource(name="boy1")의 name값으로 bean의 id를 값으로 넘겨 해당 bean을 의존성주입한다.
