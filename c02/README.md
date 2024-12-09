# 多项目

[参考 c02](build.gradle)

Gradle 的 Project 接口为我们提供了 allprojects 和 subprojects 方法，在根项目的 build.gradle 中使用这两个方法可以全局的为所有子项目进行配置。

- allprojects 的配置包括根项目
- subprojects 的配置不包括根项目

