# Java反射
## 一、什么是反射？
通过反射，可以在运行时获得程序或集合中每一个类型的成员和成员的信息。  
**JVM在运行时才动态加载类或调用方法、访问属性，不需要事先知道运行对象是谁！**
## 二、反射的用途
- 在**运行时**判断任意一个对象所属的类：`isInstance()`；
- 在**运行时**构造任意一个类的对象；
- 在**运行时**判断任一个类所具有的成员变量和方法（甚至是私有方法）；
- 在**运行时**调用任意一个对象的方法。
## 三、反射的使用
### 3.1 获得Class对象
1. 使用Class类的 `forName(String className)` 静态方法，在JDBC中常用于加载数据库驱动；
```java
Class.forName("driver");
```
2. 直接获取某个对象的 `class`；
```java
Class<?> classInt = int.class;
Class<?> classInteger = Integer.TYPE;
```
3. 调用某个对象的 `getClass()` 方法。
```java
StringBuilder sb = new StringBuilder("asdfg");
Class<?> c = sb.getClass();
```
### 3.2 判断是否是某个类的实例
一般使用 `instanceof` 关键字来判断是否属于某个类的实例。通过反射可以使用Class对象的 `isInstance()`方法判断是否为某个类的实例。
```java
public native boolean isInstance(Object object);
```
### 3.3 创建实例
通过反射来生成对象主要有两种方式：  
- 使用Class对象的newInstance()方法创建Class对象对应类的实例。
```java
Class<?> c = String.class;
Object obj = c.newInstance();
```
- 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法创建实例。
```java
Class<?> c = String.class;
Constructor constructor = c.getConstructor(String.class);
Object obj = constructor.newInstance("2333");
System.out.println(obj);
```
### 3.4 获取方法
- `getDeclaredMethods()`：返回类或接口声明的所有方法，包括public、protected、private、default权限的方法，但不包括继承的父类方法。
```java
public Method[] getDeclaredMethods() throws SecurityException
```
- `getMethods()`：返回某个类的所有公共权限的方法，包括其继承的父类的public方法。
```java
public Method[] getMethods() throws SecurityException
```
- `getMethod(String name, Class<?>... parameterTypes)`：返回一个特定方法，其中，name为方法名，parameterTypes为方法的参数对应的Class对象。
```java
public Method getMethod(String name, Class<?>... parameterTypes)
```
### 3.5 获取构造器信息
通过Class类的 `getConstructor()` 方法得到Constructor类的一个实例，而Constructor类有一个newInstance方法可以创建一个对象实例:
```java
//根据传入的参数来调用对应的 Constructor创建对象实例
public T newInstance(Object ... initargs)
```
### 3.6 获取类的成员变量（字段）信息
- `getField()`：访问共有的成员变量；
- `getDeclaredField()`：访问所有已声明的成员变量，但不能得到父类的成员变量。
### 3.7 调用类的方法
通过调用 `invoke()`方法实现：
```java
public Object invoke(Object obj, Object... args)
```
实例：
```java
public class test1 {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> klass = MethodClass.class;
        //创建methodClass的实例
        Object obj = klass.newInstance();
        //获取methodClass类的add方法
        Method method = klass.getMethod("add",int.class,int.class);
        //调用method对应的方法 => add(1,4)
        Object result = method.invoke(obj,1,4);
        System.out.println(result);
    }
}
class MethodClass {
    public final int fuck = 3;
    public int add(int a,int b) {
        return a+b;
    }
    public int sub(int a,int b) {
        return a+b;
    }
}
```
### 3.8 利用反射创建数组
```java
public static void main(String[] args){
    Class<?> c = Class.forName("java.lang.String");
    Object arr = Array.newInstance(c, 25);
    
    //向数组里添加元素
    Array.set(arr, 0, "Hello");
    Array.set(arr, 1, "!");
    Array.set(arr, 2, "How");
    Array.set(arr, 3, "are");
    Array.set(arr, 4, "you");
    Array.set(arr, 5, "?");

    System.out.println(Array.get(arr, 3));
}
```