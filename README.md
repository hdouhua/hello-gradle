# Get Started

Gradle是一种开源自动化构建工具，支持多语言环境，受Ant、Maven思想的影响，集二者之大成，相比Ant的不规范，Maven的配置复杂、生命周期限制较多，Gradle既规范也更灵活，可以使用DSL（领域特定语言，如Groovy）编写构建，脚本更加精悍。

它的构建脚本语言 Groovy 或 kotlin 都是基于JVM的语言。

如果采用Groovy编写，构建脚本后缀为.gradle，在里面可以使用Groovy语法，如果采用Kotlin编写，构建脚本后缀为.gradle.kts，在里面可以使用Kotlin语法。

它的两个对象 Project 和 Task 

- 一个构建脚本就是一个 project ，任何一个 Gradle 构建都是由一个或多个 project 组成。类比于pom、jar。
- Gradle 中的最小执行单元，一个 project 可以有多个 task 。类比于一个method、function。

## 安装

```shell
> java --version

openjdk 17.0.6 2023-01-17
OpenJDK Runtime Environment (build 17.0.6+0-17.0.6b802.4-9586694)
OpenJDK 64-Bit Server VM (build 17.0.6+0-17.0.6b802.4-9586694, mixed mode)

> brew search gradle

> brew install gradle@7

> gradle -v

------------------------------------------------------------
Gradle 7.6.2
------------------------------------------------------------

Build time:   2023-06-30 15:42:51 UTC
Revision:     dab132169006b16e7ada4ab2456e0c9d6415b52a

Kotlin:       1.7.10
Groovy:       3.0.13
Ant:          Apache Ant(TM) version 1.10.11 compiled on July 10 2021
JVM:          17.0.6 (JetBrains s.r.o. 17.0.6+0-17.0.6b802.4-9586694)
OS:           Mac OS X 13.4.1 x86_64

> echo 'export PATH="/usr/local/opt/gradle@7/bin:$PATH"' >> ~/.zshrc
```

## Hello World

```shell
> gradle helloWorld

Task :helloWorld
Hello World!

BUILD SUCCESSFUL in 468ms
1 actionable task: 1 executed
```

## Groovy 语法（仅列出与 Java 的不同）

1. 分号可省略

1. 支持动态推导

   ```groovy
   def num1 = 1
   def num2 = 2
   def result = add(num1, num2)

   def add(a, b){
       return a + b
   }
   ```

1. 方法调用传参时可以不添加括号，方法不指定 return 语句时，最后一行默认为返回值

   ```groovy
   def result = add 3, 4
   
   def add(a, b){
       a + b
   }
   ```

1. 可用单、双、三引号来表示字符串。

   - 单引号表示普通字符串
   - 双引号表示的字符串可以使用取值运算符${}，而$在单引号只只是表示一个普通的字符
   - 三引号表示的字符串又称为模版字符串，它可以保留文本的换行和缩进格式，三引号同样不支持$

   ```groovy
   def world = 'world'
   def str1 = 'hello ${world}!'
   def str2 = "hello ${world}!"
   def str3 = '''hello
   &{world}!'''
   ```

1. Groovy会为类中每个没有使用可见性修饰符的字段生成 get/set 方法，我们访问这个字段其实是调用它的 get/set 方法。同时如果类中的方法以 get/set 开头，我们也可以像普通字段一样访问这个方法

   ```groovy
   class Person {
        def name
        private def country

        def setNationality(value){
            this.country = value
        }
        def getNationality(){
            return country
        }
   }

   def person = new Person()
   person.name = 'hdouhua'
   println "name: " + person.name
   person.nationality = "Chinese"
   println "country: " + person.country
   println "nationality: " + person.getNationality()
   ```

1. Groovy中的闭包是用{参数列表 -> 代码体}表示的代码块

   - 当参数 <= 1个时，-> 箭头可以省略；
   - 当只有一个参数时，如果不指定参数名，默认以 it 为参数名。
   - 在 Groovy 中闭包对应的类型是 Closure ，所以闭包可以作为参数传递，同时闭包有一个 delegate 字段，通过 delegate 可以把闭包中的执行代码委托给任意对象来执行。
   - 没有指定闭包的 delegate ，默认为当前项目的 Project 对象。

   [参考](https://groovy-lang.org/closures.html)

   ```groovy
   { [closureParameters -> ] statements }
   ```

## Gradle构建的生命周期

init(初始化阶段) -> configure(配置阶段) -> execute(执行阶段)

1. init：初始化阶段主要是解析 `settings.gradle`，生成 Settings 对象，确定哪些项目需要参与构建，为需要构建的项目创建 Project 对象；
1. configure：配置阶段主要是解析 `build.gradle`，配置 init 阶段生成的 Project 对象，构建根项目和所有子项目，同时生成和配置在 build.gradle 中定义的 Task 对象，构造 Task 的关系依赖图，关系依赖图是一个有向无环图；
1. execute：根据 configure 阶段的关系依赖图执行 Task 。

Gradle 在上面的每一个阶段的开始和结束都会 hook 一些监听，暴露给开发者使用，方便开发者在 Gradle 的不同生命周期阶段做一些事情。

settings.gradle 和 build.gradle 分别代表 Settings 对象和 Project 对象，它们都有一个 Gradle 对象，我们可以在 Gradle 项目根目录的 settings.gradle 或 build.gradle 中获取到 Gradle 对象。

[【参考】](https://docs.gradle.org/current/javadoc/org/gradle/BuildListener.html)

## Task

1. Task 是 Gradle 中最小执行单元，它是一个接口，默认实现类为`DefaultTask`，Task 的创建必须要处于 Project 上下文中

1. 每个 Task 都定义了一些默认的属性(Property)， 比如 description 、 group 、 dependsOn 、 inputs 、 outputs 等

1. 自定义 Task

1. 支持增量式构建

通常每次执行时，每个action都会被执行，进行全量构建，其实Gradle支持增量式构建的Task，增量式构建就是当Task的输入和输出没有变化时，跳过action的执行，当Task输入或输出发生变化时，在action中只对发生变化的输入或输出进行处理，这样就可以避免一个没有变化的Task被反复构建，

## Gradle Wrapper

gradle init 执行时会同时执行 wrapper 任务，wrapper 任务会创建 gradle/wrapper 目录和其下的两个文件 gradle-wrapper.jar 和 gradle-wrapper.properties ；wrapper 任务还同时创建 gradlew 和 gradlew.bat 这两个脚本，它们统称为 Gradle Wrapper ，是对 Gradle 的一层包装。

1. gradlew：用于在linux平台下执行gradle命令的脚本；
1. gradlew.bat：用于在window平台下执行gradle命令的脚本；
1. gradle-wrapper.jar：包含Gradle Wrapper运行时的逻辑代码；
1. gradle-wrapper.properties：用于指定Gradle的下载位置和解压位置。

   其中的各个字段解释如下：

   | 字段名 | 解释 |
   | -- | -- |
   | distributionBase | 下载的 Gradle 压缩包解压后的主目录，默认 GRADLE_USER_HOME，为 ~/.gradle/ |
   | distributionPath | 下载的 Gradle 解压后的相对于 distributionBase 的路径，为 wrapper/ |
   | distsdistributionUrl | Grade 压缩包的下载地址，在此可以修改下载的 Gradle 的版本和版本类型。如 gradle-x-all.zip 表示 complete 版本， gradle-x-bin.zip 表示 binary 版本 |
   | networkTimeout | - |
   | zipStoreBase | 同 distributionBase 类似，表示存放下载的 Gradle 的压缩包的主目录 |
   | zipStorePath | 同 distributionPath 类似，表示存放下载的 Gradle 的压缩包的相对路径 |

Gradle Wrapper 的作用：
统一项目在不同用户电脑上的 Gradle 版本，同时不必让运行这个 Gradle 项目的人安装配置 Gradle 环境，提高了开发效率。

用户可以通过运行 gradlew 或 gradlew.bat 脚本来使用 gradle 命令。
例如，运行`gradle -v`命令，在linux平台下只需要运行`./gradlew -v`，在 windows 平台下只需要运行`gradlew -v`。（把gradle替换成gradlew）

## learning staff

- [installation](https://gradle.org/install/)
- [Gradle Guides](https://gradle.org/guides/)
- [Gladle构建系列](https://www.cnblogs.com/davenkin/tag/gradle/)
- [Gradle的快速入门学习](https://juejin.cn/post/6844904200120303623)
- [Gradle快速上手](https://www.it235.com/%E5%AE%9E%E7%94%A8%E5%B7%A5%E5%85%B7/Gradle/gradle.html)
