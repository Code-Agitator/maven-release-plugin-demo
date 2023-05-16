# maven-release-plugin demo

核心配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <!--  prepare发布之前执行的mvn命令   -->
                    <preparationGoals>clean verify</preparationGoals>
                    <!--  生成的tag格式  这里 @{} 而不是 ${} 可以防止project.version被其他方式覆盖 -->
                    <tagNameFormat>release-@{project.version}</tagNameFormat>
                    <!--   手动push -->
                    <pushChanges>false</pushChanges>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <scm>
        <!--      git https  -->
        <connection>scm:git:https://github.com/Code-Agitator/maven-release-plugin-demo.git</connection>
        <!--      git https  -->
        <developerConnection>scm:git:https://github.com/Code-Agitator/maven-release-plugin-demo.git
        </developerConnection>
        <!--      git url  -->
        <url>https://github.com/Code-Agitator/maven-release-plugin-demo</url>
    </scm>
</project>
```

## 假设当前版本为： **1.0.0-SNAPSHOT**
执行
```shell
mvn release:prepare
```
### 这个命令默认情况下会做以下几个事情
* 检查是否存在为commit的代码
* 检查是否存在SNAPSHOT的依赖
* 修改POM的project.version 成为一个不带SNAPSHOT的release版本(默认有三种version生成策略)
* 修改SCM中对应的TAG
* 运行单元测试
* 提交被修改的pom
* 根据SCM给commit log 打一个对应的 tag
* 更换本地pom文件为一个新的version(根据策略,默认是在最后一个版本号+1 并且加上SNAPSHOT)
* 最后根据配置执行 completionGoals

