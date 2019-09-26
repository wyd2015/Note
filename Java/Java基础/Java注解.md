---
title: Java注解
date: 2019-09-20 13:58
categories: Java
---
# Java自带注解
注解是Java1.5，JDK5.0引用的技术，与类，接口，枚举处于同一层次 。它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，用来对这些元素进行说明，注释。

Java自带三种注解
## Override
对覆盖超类中方法的方法进行标记，如果被标记的类并没有实际覆盖超类，则编译器会发出错误警告。

## Deprecated
对不应该再使用的方法添加注解，当编程人员使用这些方法时，将会在编译时显示提示信息.

## SuppressWarnings
告诉编译器忽略特定的警告信息，例如在泛型中使用原生数据类型。此注释常用的参数值有 :
- deprecation(忽略使用过时类或者方法)；
- unchecked(忽略执行了未检查装换时警告)；
- fallthrough(忽略switch直接指向到下一个case块没有break警告)；
- path(忽略类路径，源文件路径中有不存在路径时警告)；
- serial(忽略可序列化类中没有serialVersionUID时的警告)；
- finally(任何finally不能正常执行时的警告)；
- all(以上所有)。

# Java自定义注解
## 元注解
作用于注解之上的元数据或者元信息。也即注解的注解。其中`Documented`和`Inherited`是典型的`标识性注解`（`在注解内部没有成员变量`）。

### Documented
拥有这个注解的元素可以被javadoc此类的工具文档化，如果一种声明使用Documented进行注解，这种类型的注解被作为被标注的程序成员的公共API 。

### Inherited
指明该注解类型被自动继承。如果用户在当前类中查询这个元注解类型并且当前类的声明中不包含这个元注解类型，那么也将自动查询当前类的父类是否存在Inherited元注解，这个动作将被重复执行知道这个标注类型被找到，或者是查询到顶层的父类。

### Retention
定义了该Annotation被保留的时间有效范围，也就是定义了该注解的生命周期。Retention主要的参数类型包括以下几种：
- `RetentionPolicy.SOURCE`: 存在于源码中，编译时会被抛弃；
- `RetentionPolicy.CLASS`: 注解会被编译到class文件中，但会被JVM忽略；
- `RetentionPolicy.RUNTIME`: 运行时有效。JVM会读取注解，同时保存到class文件中。

### Target
说明了Annotation所修饰的对象范围，用于描述注解的使用范围。Target主要的参数类型有：
- `ElementType.TYPE`: 作用于类、接口、枚举，但不能是注解；
- `ElementType.FIELD`: 作用于字段，包含枚举值；
- `ElementType.METHOD`: 作用于方法，但不包含构造方法；
- `ElementType.PARAMETER`: 作用于方法的参数；
- `ElementType.CONSTRUCTOR`: 作用于构造方法；
- `ElementType.LOCAL_VARIABLE`: 作用于本地变量或catch语句；
- `ElementType.ANNOTATION_TYPE`: 作用于注解；
- `ElementType.PACKAGE`: 作用于包。

## @interface
用于定义注解接口来声明一个注解，其中的每一个方法都是声明其中需要的一个配置参数。

方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。

可以通过default来声明参数的默认值。

自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。
```java

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)   //类上的注解
public @interface ClassAnnotation {
  String value() default "I am Class Annotation";
}

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)  //属性上的注解
public @interface FieldAnnotation {
  String value() default "I am Field Annotation";
  String operate() default "operate";
}

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD) //方法上的注解
public @interface MethodAnnotation {
  String value() default "I am Method Annotation";
  String name() default "laobo";
}
```
## 注解的使用
```java
@ClassAnnotation  //类上注解
public class TestAnnotation {
  //属性注解
  @FieldAnnotation(value = "老伯", operate = "啥都不干！")
  public String operate;

  //方法注解
  @MethodAnnotation(value = "好人", name = "老伯")
  public void testMethod(){
    System.out.println("这个测试方法！");
  }
}
```