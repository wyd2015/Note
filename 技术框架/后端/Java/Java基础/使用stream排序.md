# jdk8中使用stream排序
## 对整型数进行排序
```java
public static void main(String[] args) {
		Integer[] arr = {1,43,123,2312,4343,21,3,5,7,8,9,11,23};
		List<Integer> list0 = Arrays.asList(arr);

		List<Integer> positiveList = list0.stream().sorted().collect(Collectors.toList());
		//List<Integer> positiveList = list0.stream().sorted(Comparator.naturalOrder()).collect(Collectors.toList());
		System.out.println(positiveList);

		List<Integer> reverseList = list0.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());
		System.out.println(reverseList);

		List<Person> list1 = new ArrayList<>();
		list1.add(new Person("张三", new BigDecimal("50.0")));
		list1.add(new Person("王五", new BigDecimal("25.0")));
		list1.add(new Person("李四", new BigDecimal("68.0")));
		list1.add(new Person("李四", new BigDecimal("17.0")));
		list1.add(new Person("张三", new BigDecimal("45.0")));

		List<Person> positive = list1.stream().sorted(Comparator.comparing(Person::getSalary)).collect(Collectors.toList());
		System.out.println(positive);

		List<Person> reverse = list1.stream().sorted(Comparator.comparing(Person::getSalary).reversed()).collect(Collectors.toList());
		System.out.println(reverse);
	}
```
