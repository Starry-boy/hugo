---
title: "Gradle入门"
description: "Gradle入门"
slug: "test"
image: "hutomo-abrianto-l2jk-uxb1BY-unsplash.jpg"
style:
    background: "#2a9d8f"
    color: "#fff"
categories:
    - Java
tags:
    - "Gradle"
    
date: 2024-06-02
math: true
---

# Gradle入门

1. gradle 与 maven 对比

   maven 侧重依赖管理、gradle 侧重构建

2. 下载gradle

   官方下载网址 https://gradle.org/next-steps/?version=8.7&format=all

3. 配置环境变量

   3.1 执行目录

   ```bash
   GRADLE_HOME={path}
   PAHT=%GRADLE_HOME%\bin  
   ```

   3.2 配置仓库地址缓存文件地址

   ```bash
   GRADLE_USER_HOME={path}
   ```

4. gradle 常用指令

   注意：gradle 的指令要在含有 build.gradle的目录执行

   | 常用指令             | 作用                   |
   | -------------------- | ---------------------- |
   | gradle clean         | 清空build目录          |
   | gradle classes       | 编译业务代码和配置文件 |
   | gradle test          | 编译测试代码，生成测试 |
   | gradle build         | 构建项目               |
   | gradle build -x test | 跳过测试构建项目       |

5. 修改 maven 下载源

   在gradle 安装目录的 init.d 目录下创建已 .gradle 结尾的文件

   使用mavenLocal() 时Gradle默认会按以下顺序去查找本地的maven仓库：USER_HOME/.m2/settings.xml >> M2_HOME/conf/settings.xml >> USER_HOME/.m2/repository。注意，环境变量要加入M2_HOME， 我们配环境时很多时候都是使用MAVEN_HOME或者直接在path中输入bin路径了，导致mavenLocal无法生效。

   ```groovy
   allprojects {
       repositories {
           // maven本地仓库 要求本地配置有 M2_HOME 的环境变量
           mavenLocal()
           maven {
               name "Alibaba" ; url "https://maven.aliyun.com/repository/public"
           }
           maven {
               name "Bstek"; url "https://nexus.bsdn.org/content/groups/public"
           }
       }
   
       buildscript {
           repositories {
               mavenLocal()
               maven {
                   name "Alibaba" ; url "https://maven.aliyun.com/repository/public"
               }
               maven {
                   name "Bstek"; url "https://nexus.bsdn.org/content/groups/public"
               }
               maven {
                   name "M2"; url "https://plugins.gradle.org/m2/"
               }
           }
       }
   }
   ```

6. 修改 gradle wrapper 版本

   ```bash
   gradle wrapper --gradle-version=8.7
   ```

7. 解决 gradle 下载慢的问题

   跟换gradle下载地址

   ```properties
distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.5-bin.zip
   ```

   

   

   

   

   

   

   

   

   

   

