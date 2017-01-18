---
title: 将自己的库发布到maven
tags:
  - maven
toc: true
date: 2016-10-27 22:39:43
categories: 学习总结
---

### 什么是maven?
官网网站：[Apache Maven Project][1]
<!--more-->

> Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.


> Apache Maven是一个软件项目管理和理解工具。基于项目对象模型（POM）的概念，Maven可以管理一个项目的构建，从主要的信息模块报告和收集资料。

maven是一个项目管理工具，它包含了一个项目对象模型（Project Object Model），一组标准集合，一个项目生命周期，一个依赖管理系统，maven可以很方便的管理项目报告，生成站点，管理JAR文件等。

### 发布library到maven
maven作为一个中心仓库，在Android开发中，可以将自己写的lib发布到maven，并使用Sonatype Nexus进行管理。
仓库信息：
```java
Repository ID: airimos
Repository Name: airimos
Repository Type: hosted
Repository Policy: Release
Repository Format: maven2
Contained in groups: 
```
```xml
<distributionManagement>
  <repository>
    <id>airimos</id>
    <url>http://maven.airimos.com/nexus/content/repositories/airimos</url>
  </repository>
</distributionManagement>
```

 1.使用Android studio，在自己库的src同级目录下的`build.gradle`中进行一些配置，添加以下代码：

```groovy
apply plugin: 'com.android.library'
apply plugin: 'maven'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
}


tasks.withType(JavaCompile){
    options.encoding = "UTF-8"
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())

def isReleaseBuild() {
    return VERSION_NAME.contains("SNAPSHOT") == false
}

def getRepositoryUsername() {
    return hasProperty("NEXUS_USERNAME") ? properties.getProperty("NEXUS_USERNAME") : ""
}

def getRepositoryPassword() {
    return hasProperty("NEXUS_PASSWORD") ? properties.getProperty("NEXUS_PASSWORD") : ""
}

afterEvaluate { project ->
    uploadArchives {
        repositories {
            mavenDeployer {

                pom.groupId = properties.getProperty("GROUP")
                pom.artifactId = properties.getProperty("POM_ARTIFACT_ID")
                pom.version = properties.getProperty("VERSION_NAME")
                repository(url: properties.getProperty("RELEASE_REPOSITORY_URL")) {
                    authentication(userName: properties.getProperty("NEXUS_USERNAME"), password: properties.getProperty("NEXUS_PASSWORD"))
                }
                snapshotRepository(url: properties.getProperty("SNAPSHOT_REPOSITORY_URL")) {
                    authentication(userName: properties.getProperty("NEXUS_USERNAME"), password: properties.getProperty("NEXUS_PASSWORD"))
                }
            }
        }
    }
    task sourcesJar(type: Jar) {
        from android.sourceSets.main.java.srcDirs
        classifier = 'sources'
        from android.sourceSets.main.java.sourceFiles
    }
    task javadoc(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }
    task javadocJar(type: Jar, dependsOn: javadoc) {
        classifier = 'javadoc'
        from javadoc.destinationDir
    }
    javadoc {
        options {
            encoding "UTF-8"
            charSet 'UTF-8'
            author true
            version true
        }
    }
    artifacts {
        archives javadocJar
        archives sourcesJar
    }
}
```

特别地，在开头处要添加`apply plugin: 'maven'`，否则在进行同步时报错。

 2.配置项目根目录下的`local.properties`文件

```groovy
sdk.dir=D\:\\Android\\Sdk
GROUP=com.uniview.usbcameralib
VERSION_NAME=1.0.0
POM_ARTIFACT_ID=usbcameralib
RELEASE_REPOSITORY_URL=http\://maven.airimos.com/nexus/content/repositories/airimos
SNAPSHOT_REPOSITORY_URL=http\://maven.airimos.com/nexus/content/repositories/airimosSnap
NEXUS_USERNAME=username
NEXUS_PASSWORD=password
```
这些配置的信息，会在第一步添加的代码读取到。

 - `GROUP：`将lib发布到maven的哪个仓库组中，这里的仓库名字为airimos，会发布到airimos下的com/uniview/usbcameralib中。
 - ` VERSION_NAME：` 库的版本号
 - `POM_ARTIFACT_ID：` 库的名称
 - `RELEASE_REPOSITORY_URL：`正式仓库地址
 - `SNAPSHOT_REPOSITORY_URL：` 不稳定仓库地址
 - `NEXUS_USERNAME ：`Sonatype Nexus的用户名
 - `NEXUS_PASSWORD：`Sonatype Nexus的密码

 3. 在Android studio右侧的`Gradle projects`中，双击 uploadArchives上传到maven服务器，将会进行构建，显示`BUILD SUCCESSFUL`则表示成功。
 4. 以后每次对库进行更新后，将库的版本号增加，在使用的项目中重新下载就好。

### 依赖上传的库

 1. 在新建的项目中使用库，在项目的根目录下的`build.gradle`配置：

```groovy
repositories {
        jcenter()
        //配置maven仓库地址
        maven { url 'http://maven.airimos.com/nexus/content/repositories/airimos/' }
    }
```

 2. 在src同级目录下的`build.gradle`进行依赖：

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
    //依赖该库
    compile 'com.uniview.usbcameralib:usbcameralib:1.0.0'
}
```
### 仓库类型
maven中的仓库分为两种：`snapshot快照仓库`和`release 发布仓库`。`snapshot快照仓库`用于保存开发过程中的不稳定版本，`release正式仓库`用来保存稳定的发行版本，当定义一个组件或者模块为快照版本时，只需要在pom文件中在该模块的版本号后加上`-SNAPSHOT（要大写）`即可。
```xml
<version>1.0.0-SNAPSHOT</version>
```
maven2会根据模块的版本号中是否带有`-SNAPSHOT`来判断是快照版本还是正式版本，如果是快照版本在部署时会自动发布到快照版本库中，而使用快照版本的模块，在不更改版本号的情况下，直接编译打包时，maven会自动从镜像服务器上下载最新的快照版本；如果是正式版本，那么在部署时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，编译打包时如果本地已经存在该版本的模块，则不会主动去镜像服务器上下载。
因此在开发阶段，可以将共用库的版本设置为快照版本，而依赖组件则引用快照版本进行开发，在共用库的快照版本更新后，也不需要修改pom文件的版本号来下载新的版本，直接执行相关编译、打包命令即可重新下载最新的快照库，方便开发。开发完毕后，就可将该库发布到release正式版本库中。



  [1]: http://maven.apache.org/
