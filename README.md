# maven-release-plugin

### 我是什么？

一个用于发布项目的maven插件，用于节省大量重复，枯燥的工作。发布一个项目只需要两步：

* 准备(prepare)
* 发布(perform)

### 我和git什么关系

git是一个版本控制(version controller)，关注的事代码版本控制。maven-release-plugin是一个版本管理(version management)
，关注于项目版本的管理。该插件的意义正如官网所说：减少枯燥的重复工作，提供了版本管理的默认实现以及很多的个性化配置

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

# 开干

### 假设当前版本为： **1.0.0-SNAPSHOT**

执行

```shell
mvn release:prepare
```

### 这个命令默认情况下会做以下几个事情

* 检查是否存在为commit的代码
* 检查是否存在SNAPSHOT的依赖
* 修改POM的project.version 成为一个不带SNAPSHOT的release版本(默认有三种version生成策略\[see 其他])
* 修改SCM中对应的TAG
* 运行单元测试
* 提交被修改的pom
* 根据SCM给commit log 打一个对应的 tag
* 更换本地pom文件为一个新的version(根据策略,默认是在最后一个版本号+1 并且加上SNAPSHOT)
* 最后根据配置执行 **completionGoals**

### 此时

* 此时我们的 **1.0.0-release** 版本已经被commit到本地并且打上了对应的tag
* 同时，我们的 **project.version** 递增了一个SNAPSHOT版本
* 项目更目录会生成两个文件 pom.xml.releaseBackup
* release.properties 用于后续的操作

### 发布

* 发布前要检查，如果在插件中配置了 pushChanges 为false
* 那么prepare的时候不会自动push任何代码,只是commit到了本地
* 然而perform做的事情就是拉代码和deploy,所以会导致拉不到代码而失败
* 所以需要手动push代码和tag
* (git push origin --tags/git push origin \<tagName>)
* (在idea中在push的时候选择左下角的 ALL TAG 即可)
* 执行

```shell
mvn release:perform
```

### 做了以下事情

* 根据SCM的URL和TAG 拉取最新的tag代码到 workingDirectory(默认: target/checkout)
* 对拉下来的代码执行 **deploy / site-deploy** 发布到仓库
* 构建成功的话就会删除根目录的 pom.xml.releaseBackup / release.properties 文件

### 此时

* 一个 **1.0.0-SNAPSHOT** 发布到仓库并且打一个tag为 **1.0.0**
* 修改version为 **1.0.1-SNAPSHOT** 并将 **1.0.0** deploy到私有仓库的过程就完成了

### 大版本更新的方式

1. 手动修改
    * 手动修改pom.xml的version,将 **1.0.1-SNAPSHOT** 改成 **1.2.0-SNAPSHOT**
2. 使用 mvn versions:set
    * 执行
   ```shell
   mvn versions:set -DnewVersion=1.2.0-SNAPSHOT 
   ```

* **区别：在多模块的项目中，mvn versions:set 可以处理包括依赖该模块的其他模块中版本号变化**



## 其他

### 提交代码到git而不想deploy

* 在perform过程中如果只是想提交代码到git而不想deploy可以有两种方式

1. 直接吧 pom.xml.releaseBackup release.properties 清除
   ```shell
   mvn release:clean
   ```

2. 或者使用不deploy的perform
   ```shell
   # goals 就是这个命令完成的目标  默认为 deploy / site-deploy 为空就是啥也不干
   mvn release:perform -Dgoals=""
   ```

* 由此可见 从版本更新到发布的过程中是需要和 **maven-deploy-plugin** 配合使用

### 版本号生成策略

1. default: 去掉SNAPSHOT作为release版本，递增一个版本号作为新的SNAPSHOT版本
2. OddEvenVersionPolicy: 将偶数版本作为release，奇数作为开发版
3. SemVerVersionPolicy: 语义化版本号：主版本号.次版本号.修订号 => breaking change.new feature.bug fix......







