---
title: 'Lamda表达式'
Date: 2019-05-18 11:23:43
tag: 'java'
---
## 一、组成
Lamda表达式允许把函数作为一个方法的参数。一个lamda表达式由  
`参数列表`、`->`、`函数体`三部分组成。
## 二、功能
lamda表达式实现了接口里的有且唯一的一个抽象方法，这种接口称为`函数式接口`。

每种表达式的写法其实都是某个函数式接口的实现类。
## 三。 函数式接口
用`@FunctionalInterface`修饰的接口，该接口有且仅有一个抽象方法。
函数式接口中，可以存在三种方法：
1. 有且仅有一个的抽象方法，lamda表达式使用的方法；
2. 默认方法：使用`default`修饰，有默认实现；
3. 静态方法： 使用`static`修饰，有默认实现。
```java
public interface PersonCallback {
  //函数式接口中的抽象方法，有且仅有一个
  void callback();

  //在接口中用default修饰的方法称为默认方法
  //接口中的默认方法一定要有默认实现（方法体），接口实现者可以继承它，也可以覆盖它。
  default void sayName(){
    System.out.println("tell me your name ...");
  }

  //接口中的静态方法，一定有默认实现
  static void testStatic(){
    System.out.println("tell me your static name ...");
  }
}
```
`@FunctionalInterface`注解可以起到校验的作用。
```java
public class API8 {
  public static void main(String[] args) {
    /*new API8().test("wcg", new PersonCallback() {
      @Override
      public void callback() {

      }
    });*/
    new API8().test("ak47", ()->{
      System.out.printf("---哒哒哒---");
    });

    PersonCallback callback = ()->{
      System.out.println("args = []");
    };
    callback.callback();
  }

  private void test(String name, PersonCallback callback){
    System.out.printf(name);
    callback.callback();
  }
}
```