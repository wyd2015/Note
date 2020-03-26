使用idea编译Java项目时，idea重启后，项目老是恢复到jdk1.5版本。一劳永逸的办法是
通过修改maven的配置文件，指定Java项目默认使用的jdk版本。

`*\apache-maven-3.5.0\conf\setting.xml`

在`<profiles></profiles>`标签里添加配置，然后maven update即可。
```xml
<profiles>
  <profile>
    <id>jdk-1.8</id>
    <activation>
      <activeByDefault>true</activeByDefault>
      <jdk>1.8</jdk>
    </activation>
    <properties>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
      <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
  </profile>
</profiles>
```