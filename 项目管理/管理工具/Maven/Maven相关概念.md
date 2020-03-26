## maven-构建生命周期
**maven的生命周期是为了对所有的构建过程进行抽象和统一。**

包括项清理、初始化、编译、打包、测试、部署等几乎所有的构建步骤。
| 阶段  | 处理 | 描述 |
| :---: | :---: | :--- |
| prepare-resources | 资源拷贝 | 本阶段可自定义需要拷贝的资源 |
| compile | 编译 | 本阶段完成源代码的编译 |
| package | 打包 | 本阶段根据 pom.xml 中的打包配置创建 jar/war 包 |
| install | 安装 | 本阶段在 本地/远程 仓库中安装工程包 |

当需要在某个特定阶段之前或之后执行目标时，可使用 `pre` 和 `post` 来定义这个目标。

当maven开始构建工程时，会按照所定义的阶段序列的顺序，依次执行每个阶段注册的目标。

Maven有以下三个标准的生命周期：
- clean：删除构建目录
- default/build：构建应用
- site：用于创建新的报告文档、部署站点等。

**目标** 表示一个特定的、对构建和管理工程有帮助的任务。它可能绑定了0或多个构建阶段。没有绑定任何构建阶段的目标可以在构建生命周期之外被直接调用执行。
执行的顺序依赖于目标和构建阶段被调用的顺序。

## maven坐标的组成
- groupId: 定义当前maven项目的隶属项目；
- artifactId: 定义实际项目中的一个模块；
- version: 定义当前项目的当前版本；
- packaging: 定义该项目的打包方式。

## 依赖管理
### 1. 依赖声明
```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
```
### 2. 依赖范围
`<scope>`标签：用于指定依赖的作用范围。
- compile: 默认值。适用于所有阶段。表明该jar包在编译、运行以及测试路径中均可见，并且会随着项目一起发布。如`log4j`;
- test: 只在测试时使用，用于编译和运行测试代码，不随项目发布。如`junit`；
- runtime: 无需参与项目的编译，不过后期的测试和运行周期需要参与。与compile相比，跳过了编译。如jdbc驱动，适用于运行和测试阶段；
- provided: 编译和测试时有效，但该依赖在运行时由服务器提供，且打包时也不会被包含进去。如 `servlet-api`;
- system: 类似provided，需要显示提供包含依赖的jar包，不会从maven仓库下载，而是从本地文件系统获取，需要添加systemPath的属性来定义获取路径。如`JDBC Driver`。
