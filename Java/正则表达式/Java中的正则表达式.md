# Java中的正则表达式
在`java.util.regex`包中主要包括一下三个类：
- **Pattern**
  Pattern对象是一个正则表达式的编译表示。Pattern类没有公共构造方法。要创建一个Pattern对象，
  必须首先调用其公共静态编译方法，该方法接受一个正则表达式作为它的第一个参数，返回一个Pattern实体对象。
- **Matcher**
  Matcher对象是对输入的字符串进行解释和匹配操作的引擎。与Pattern类一样，Matcher没有公共构造方法，需要调用
  Pattern对象的`matcher()`方法来获得一个Matcher对象。
- **PatternSyntaxException**
  PatternSyntaxException是一个非强制异常类，它表示一个正则表达式模式中的语法错误。

## Java中的正则表达式语法
在其它语言中，`\\`表示：
*我想要在正则表达式中插入一个普通的反斜杠，没有任何特殊含义*；

在Java中，`\\`表示：
*我要插入一个正则表达式的反斜线，其后的字符具有特殊意义*。

所以在其它语言（如`Perl`）中，一个反斜杠`\`就足矣具有转义的作用，但在Java中，正则表达式则需要有两个反斜杠才能被解析为转义符。也可以简单理解为在Java的正则表达式中，两个反斜杠`\\`代表其他语言中的一个反斜杠`\`，因此Java中表示一位数的正则表达式是`\\d`，而表示一个普通反斜杠的正则表达式是`\\\\`。

>根据 Java Language Specification 的要求，Java 源代码的字符串中的反斜线被解释为 `Unicode 转义`或`其他字符转义`。因此必须在字符串字面值中使用两个反斜线，表示正则表达式受到保护，不被 Java 字节码编译器解释。例如，当解释为正则表达式时，字符串字面值 `\b` 与单个退格字符匹配，而 `\\b` 与单词边界匹配。字符串字面值 `\(hello\)` 是非法的，将导致编译时错误；要与字符串 (hello) 匹配，必须使用字符串字面值 `\\(hello\\)`。

更详细的语法参考[正则表达式](正则表达式.md)

```java
public class RegexTest {
  public static void main(String[] args) {
    // 去除单词与 , 和 . 之间的空格
    String str = "Hello , World .";
    String pattern = "(\\w)(\\s+)([.,])";
    // $0 匹配 `(\w)(\s+)([.,])` 结果为 `o空格,` 和 `d空格.`
    // $1 匹配 `(\w)` 结果为 `o` 和 `d`
    // $2 匹配 `(\s+)` 结果为 `空格` 和 `空格`
    // $3 匹配 `([.,])` 结果为 `,` 和 `.`
    System.out.println(str.replaceAll(pattern, "$1"));//Hello java
    System.out.println(str.replaceAll(pattern, "$2"));//Hell  jav
    System.out.println(str.replaceAll(pattern, "$3"));//Hell, jav.
    System.out.println(str.replaceAll(pattern, "$1$2"));//Hello  java
    System.out.println(str.replaceAll(pattern, "$1$3"));//Hello, java.
    System.out.println(str.replaceAll(pattern, "$2$3"));//Hell , jav .
    System.out.println(str.replaceAll(pattern, "$1$2$3"));//Hello , java .
  }
}
```

当在`()`内的模式的开头加入`?:`，表示这个模式仅分组，不创建反向引用。
```java
@Test
public void test6(){
  String str = "regex.png";
  Pattern pattern = Pattern.compile("(jpg|png)");//分组且创建反向引用
  Matcher matcher = pattern.matcher(str);
  while (matcher.find()){
    System.out.println(matcher.groupCount());//1
    System.out.println(matcher.group());//png
    System.out.println(matcher.group(1));//png
  }
}

@Test
public void test7(){
  String str = "regex.png";
  Pattern pattern = Pattern.compile("(?:jpg|png)");//分组但不创建反向引用
  Matcher matcher = pattern.matcher(str);
  while (matcher.find()){
    System.out.println(matcher.groupCount());//0
    System.out.println(matcher.group());//png
    System.out.println(matcher.group(1));//报错：java.lang.IndexOutOfBoundsException: No group 1
  }
}
```
## Matcher类的方法
### 索引方法
索引方法提供了有用的索引值，精确表明输入的字符串中在哪个位置能找到匹配内容。
| 方法 | 作用 |
| :-- | :-- |
| public int `start()` | 返回以前匹配的初始索引 |
| public int `start(int group)` | 返回在以前匹配操作期间，由给定组所捕获的子序列的初始索引 |
| public int `end()` | 返回最后匹配字符之后的偏移量 |
| public int `end(int group)` | 返回在以前匹配操作期间，由给定组所捕获的子序列的最后字符之后的偏移量 |

### 研究方法
用于检查输入的字符串并返回一个布尔值，标识是否找到该模式。
| 方法 | 作用 |
| :-- | :-- |
| public boolean `find()` | 尝试查找与该模式匹配的输入序列的下一个子序列 |
| public boolean `find(int start)` | 重置此匹配器，然后尝试查找匹配该模式、从指定索引开始的输入序列的下一个子序列 |
| public boolean `matches()` | 尝试将`整个区域`与模式进行匹配 |
| public boolean `lookingAt()` | 尝试将从`区域开头`开始的输入序列与该模式进行匹配 |

其中，`matches()`和`lookingAt()`方法都用于尝试匹配一个输入序列模式，它们的不同：
- matches()：要求整个字符串与模式都匹配；
- lookingAt()：不要求匹配整个字符串，但需要从第一个字符开始匹配。
```java
@Test
public void test8(){
  String reg = "foo";
  String str1 = "foooooooooooooooooooooo";
  String str2 = "oooooooooofoooooooooooo";

  Pattern p = Pattern.compile(reg);
  Matcher m1 = p.matcher(str1);
  Matcher m2 = p.matcher(str2);

  System.out.println("m1.lookingAt() is: "+m1.lookingAt());//true
  System.out.println("m2.lookingAt() is: "+m2.lookingAt());//false
  System.out.println("m1.matches() is: "+m1.matches());//false
  System.out.println("m2.matches() is: "+m2.matches());//false
}
```

### 替换方法
替换输入字符串里的文本的方法
| 方法 | 作用 |
| :-- | :-- |
| public String `replaceAll(String replaceContext)` | 替换模式与给定替换字符串相匹配的输入序列的每个子序列 |
| public String `replaceFirst(String replaceContext)` | 替换模式与给定替换字符串匹配的输入序列的第一个子序列 |
| public `static` String `quoteReplacement(String s)` | 返回指定字符串的字面替换字符串 |
| public StringBuffer `appendTail(StringBuffer sb)` | 把最后一次匹配到内容之后的字符串追加到sb中 |
| public Matcher `appendReplacement(StringBuffer sb, String replaceContext)` | 把匹配到的内容替换为replaceContext，并且把从上次替换的位置到这次替换位置之间的字符串也拿到，然后加上这次替换后的结果一起追加到sb里 |
```java
@Test
public void test9(){
  String reg = "dog";
  String str1 = "The dog say kid. All dogs say kids.";
  String replace = "cat";

  Pattern p = Pattern.compile(reg);
  Matcher m = p.matcher(str1);
  System.out.println(str1);//The dog say kid. All dogs say kids.
  System.out.println(m.replaceAll(replace));//The cat say kid. All cats say kids.
  System.out.println(m.replaceFirst(replace));//The cat say kid. All dogs say kids.
}

@Test
public void test10(){
  String reg = "a*b";//匹配 b 前面没有a或有多个a的模式
  String str1 = "aabfooaabfooabfoobkkk";
  String replace = "-";

  Pattern p = Pattern.compile(reg);
  Matcher m = p.matcher(str1);
  StringBuffer sb = new StringBuffer();

  //替换所有
  while(m.find()){
      m.appendReplacement(sb, replace);
  }
  m.appendTail(sb);
  System.out.println(sb.toString());//-foo-foo-foo-kkk
}

@Test
public void test11(){
  String reg = "a*b";//匹配 b 前面没有a或有多个a的模式
  String str1 = "aabfooaabfooabfoobkkk";
  String replace = "-";

  Pattern p = Pattern.compile(reg);
  Matcher m = p.matcher(str1);
  StringBuffer sb = new StringBuffer();

  //替换第一个
  if(m.find()){
      m.appendReplacement(sb, replace);
  }
  m.appendTail(sb);
  System.out.println(sb.toString());//-fooaabfooabfoobkkk
}
```
