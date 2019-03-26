# stream常见API

## 一、流的创建
1. 使用`Collection`接口的stream方法将任何集合转换为一个流
```java
String[] array = {"java", "jdk8", "stream"};
Arrays.asList(array).stream();
```
2. 如果有一个数组array，可以使用静态的Stream.of()方法创建流
```java
Stream<String> words = Stream.of(array);

// of方法具有可变长参数，可以构建具有任意数量引元的流
Stream<String> words = Stream.of("java", "jdk8", "stream");
```
3. 使用Array.stream(array, from, to)方法可以从数组中位于from(包含)和to(不包含)的元素中创建一个流
```java
Stream<String> words = Arrays.stream(array, 0, 2);
```
3.   n