---
layout: post
title: Android Studio使用Gradle上传AAR至Maven
categories:
- Android
- Gradle
- 开源
date: 2015-02-11
---

这几天有个开源library要上传到Maven Central Repository，找到某大神的博客，他用[Maven](http://maven.apache.org/)上传的库，我依葫芦画瓢搞了两天，卧槽，可能是人品问题，mvn一直报一个莫名其妙的异常，Google了半天完全没有头绪，恨死那个Windows的黑框框了。

后来从国外某大神的开源project那学到到可以用[Gradle](http://www.gradle.org/)来上传AAR到maven central repository，终于可以和那个坑爹的黑框框说good bye了- -

<!-- more -->

## 前期工作

前期我们要做一些准备工作，包括：

1. 注册Sonatype帐号
2. 创建一个JIRA ticket

打开[Sonatype JIRA](https://issues.sonatype.org/)注册帐号，注册好之后打开[Create Issue](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)创建一个JIRA ticket，一个JIRA ticket对应一个项目。

其中Summary填写项目名，例如AndroidTagGroup；Description填写项目描述；Group Id必须是项目包名的父级，比如我的包名为me.gujun.android.taggroup，那么为了我所有的项目都可以发布，Group Id填写为me.gujun。

其它按照提示填写，完成后大概两个工作日左右，该issue会变成`RESOLVED`状态，表示可用，在可用前除了最后一步正式发布之外，其它都可以正常进行。


## 使用GnuPG生成密钥

发布release版本时需要对上传的文件加密和签名，GPG用于生成签名，管理密钥。

上传前我们需要做两件事：

1. 生成密钥对
2. 上传密钥

### 安装

下载地址：[GPG](https://www.gnupg.org/download/index.html)、[GPG for Windows](http://gpg4win.org/)

安装之后验证一下

```
$ gpg --version

gpg (GnuPG) 2.0.26 (Gpg4win 2.2.3)
libgcrypt 1.6.2
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
```

### 生成密钥

```
$ gpg --gen-key
```

除了姓名、邮箱、备注外其它都可以使用默认设置，最后需要输入一个passphrase，妥善保管这个口令，后面配置Gradle脚本时需要用到。

### 查看密钥

查看公钥：

```
$ gpg --list-keys

C:/Users/PingGe/AppData/Roaming/gnupg/pubring.gpg
-------------------------------------------------
pub   2048R/F874D485 2015-02-10
uid       [ultimate] JunGu (Sonatype) <2dxgujun@gmail.com>
sub   2048R/F27758E5 2015-02-10
```

输出的路径为公钥文件，F874D485为keyId，需要上传给服务器。

查看私钥：

```
$ gpg --list-secret-keys

C:/Users/PingGe/AppData/Roaming/gnupg/secring.gpg
-------------------------------------------------
sec   2048R/F874D485 2015-02-10
uid                  JunGu (Sonatype) <2dxgujun@gmail.com>
ssb   2048R/F27758E5 2015-02-10
```

私钥文件路径在配置Gradle脚本时需要用到。

### 上传公钥

```
$ gpg --keyserver hkp://pool.sks-keyservers.net --send-keys F874D485

gpg: sending key F874D485 to hkp server pool.sks-keyserver.net
```

把之前生成的公钥上传至服务器，系统需要你上传的公钥来验证发布时的文件。

F874D485为之前生成的公钥的keyId，一旦提交至一个key server，公钥会自动同步到其它key server。


## 配置Gradle脚本

Gradle脚本使用了开源项目[gradle-mvn-push](https://github.com/chrisbanes/gradle-mvn-push)，Thx Chris大神!

### 添加maven-push.gradle

把下面的脚本复制到工程目录，新建文件maven-push.gradle。

{% highlight groovy %}
/*
 * Copyright 2013 Chris Banes
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

apply plugin: 'maven'
apply plugin: 'signing'

def isReleaseBuild() {
    return VERSION_NAME.contains("SNAPSHOT") == false
}

def getReleaseRepositoryUrl() {
    return hasProperty('RELEASE_REPOSITORY_URL') ? RELEASE_REPOSITORY_URL
            : "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
}

def getSnapshotRepositoryUrl() {
    return hasProperty('SNAPSHOT_REPOSITORY_URL') ? SNAPSHOT_REPOSITORY_URL
            : "https://oss.sonatype.org/content/repositories/snapshots/"
}

def getRepositoryUsername() {
    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
}

def getRepositoryPassword() {
    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
}

afterEvaluate { project ->
    uploadArchives {
        repositories {
            mavenDeployer {
                beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }

                pom.groupId = GROUP
                pom.artifactId = POM_ARTIFACT_ID
                pom.version = VERSION_NAME

                repository(url: getReleaseRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }
                snapshotRepository(url: getSnapshotRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }

                pom.project {
                    name POM_NAME
                    packaging POM_PACKAGING
                    description POM_DESCRIPTION
                    url POM_URL

                    scm {
                        url POM_SCM_URL
                        connection POM_SCM_CONNECTION
                        developerConnection POM_SCM_DEV_CONNECTION
                    }

                    licenses {
                        license {
                            name POM_LICENCE_NAME
                            url POM_LICENCE_URL
                            distribution POM_LICENCE_DIST
                        }
                    }

                    developers {
                        developer {
                            id POM_DEVELOPER_ID
                            name POM_DEVELOPER_NAME
                        }
                    }
                }
            }
        }
    }

    signing {
        required { isReleaseBuild() && gradle.taskGraph.hasTask("uploadArchives") }
        sign configurations.archives
    }

    task apklib(type: Zip){
        appendix = extension = 'apklib'

        from 'AndroidManifest.xml'
        into('res') {
            from 'res'
        }
        into('src') {
            from 'src'
        }
    }

    task androidJavadocs(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath() .join(File.pathSeparator))
    }

    task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
        classifier = 'javadoc'
        from androidJavadocs.destinationDir
    }

    task androidSourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.srcDirs
    }

    artifacts {
        archives androidSourcesJar
        archives androidJavadocsJar
        archives apklib
    }
}
{% endhighlight %}

### 配置Project属性

在工程目录下的gradle.properties文件中设置属性，把属性修改成自己的。

{% highlight groovy %}
VERSION_NAME=1.0
VERSION_CODE=1
GROUP=me.gujun.android.taggroup

POM_DESCRIPTION=Android Library to display a set of tags
POM_URL=https://github.com/2dxgujun/AndroidTagGroup
POM_SCM_URL=https://github.com/2dxgujun/AndroidTagGroup
POM_SCM_CONNECTION=scm:https://github.com/2dxgujun/AndroidTagGroup.git
POM_SCM_DEV_CONNECTION=scm:https://github.com/2dxgujun/AndroidTagGroup.git
POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo
POM_DEVELOPER_ID=2dxgujun
POM_DEVELOPER_NAME=Jun Gu

SNAPSHOT_REPOSITORY_URL=https://oss.sonatype.org/content/repositories/snapshots
RELEASE_REPOSITORY_URL=https://oss.sonatype.org/service/local/staging/deploy/maven2
{% endhighlight %}

**注意：**VERSION_NAME后面加-SNAPSHOT表示发布的是版本快照。

GROUP设置成项目包名，注意，父级要和之前创建JIRA ticket时的Group Id一致。

### 配置Project构建脚本

之后修改工程目录下的build.gradle：

{% highlight groovy %}
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.0.1'
    }
}

allprojects {
    version = VERSION_NAME
    group = GROUP

    repositories {
        jcenter()
    }
}
{% endhighlight %}

### 配置Module属性

工程配置好之后，我们还需要给上传的Module配置（我们的工程中可能有多个Module需要上传到仓库中，要给每个Module添加配置）

给需要上传到Maven仓库的Module提供一个gradle.properties文件：

```
POM_NAME=Android TagGroup Library
POM_ARTIFACT_ID=library
POM_PACKAGING=aar
```

POM_ARTIFACT_ID设置成Module名。

这个组件对应的依赖已经浮出水面了：GROUP:POM_ARTIFACT_ID:VERSION_NAME

即

`me.gujun.android.taggroup:library:1.0`

### 配置Module构建脚本

修改Module目录的build.gradle，在最后加上：

```
apply from: '../maven-push.gradle'
```

### 配置全局属性

这个全局的Gradle配置文件默认在`USER_HOME/.gradle/gradle.properties`，没有的话可以新建一个。

在这里配置Maven服务器的用户名和密码，还需要配置之前生成的keyId, password和一个secretKeyRingFile，这个文件用来在上传release版本时对文件进行签名。

```
NEXUS_USERNAME=2dxgujun
NEXUS_PASSWORD=123456

signing.keyId=F874D485
signing.password=123456
signing.secretKeyRingFile=C:/Users/PingGe/AppData/Roaming/gnupg/secring.gpg
```

Windows的secretKeyRingFile路径如上所示，其它系统可以使用`gpg --list-secret-keys`命令查看。


## 部署release版本

所有的配置已经完成，现在可以上传了，在Android Studio的Terminal输入命令：

```
gradle uploadArchives
```

如果上传成功，最后你会看到控制台打印类似的信息：

```
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0.aar to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 19K from remote
Uploaded 19K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0.aar.asc to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 0K from remote
Uploaded 0K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0-sources.jar to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 7K from remote
Uploaded 7K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0-sources.jar.asc to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 0K from remote
Uploaded 0K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0.apklib to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 11K from remote
Uploaded 11K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0.apklib.asc to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 0K from remote
Uploaded 0K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0-javadoc.jar to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 34K from remote
Uploaded 34K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0-javadoc.jar.asc to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 0K from remote
Uploaded 0K
Uploading: me/gujun/android/taggroup/library/1.0/library-1.0.pom.asc to repository remote at https://oss.sonatype.org/service/local/staging/deploy/maven2
Transferring 0K from remote
Uploaded 0K
```

上传成功后，打开[Sonatype Nexus Professional](https://oss.sonatype.org/)登录，选择左侧*Build Promotion*菜单中的*Staging Repositories*选项，在出现的选项卡右上角的搜索框输入关键字，筛选出你上传的组件所在的repository。

部署时创建的repository会根据部署项目的groupId来命名，例如我的groupId为me.gujun，那么我的repository即为megujun-xxxx，后面的xxxx为4个数字，初次部署为1000，后面每次部署这个数字都会+1。选择这个repository，列表下面的面板会显示一些详细信息。

![OSS Sonatype](/media/2015/02/11/oss_sonatype.png)


## 正式发布

部署完成之后，上传的组件会存储在一个独立的临时staging repository，在正式发布之前如果你在测试时遇到任何问题，都可以删除这个staging repository，在修复之后重新部署。正式发布才会同步到maven central repository。

通常情况下正式发布操作需要手动完成。

首先打开[Sonatype Nexus Professional](https://oss.sonatype.org/)登录，打开*Staging Repositories*列表，筛选出之前部署的repository。

部署完成之后，这个repository会处于*Open*状态，你可以切换到*Content*标签页检查这个repository，如果确信没有问题，可以点击列表上面的*Close*按钮，这样会触发系统对这个repository进行评估。

如果你的组件不满足评估要求，*Close*操作会失败。

遇到这种情况，可以切换到*Activity*标签查看系统评估时出现的具体问题，修复问题，再尝试*Close*操作；如果需要重新部署，可以点击列表上面的*Drop*按钮删除这个repository，在本地修改之后，再重新部署。

成功close之后，可以点击*Release*按钮正式发布repository，组件会被移动到OSSRH的release repository，这个仓库会同步到maven central repository。

![Promote Release](/media/2015/02/11/promote_release.png)

**注意：**如果你是第一次发布，需要到之前创建的JIRA ticket评论一下，告诉他们你已经release了，需要同步下。

Gradle文件的配置可以参考我的项目：[AndroidTagGroup](https://github.com/2dxgujun/AndroidTagGroup)


## 遇到问题

androidJavadocs 错误: 编码GBK的不可映射字符。

解决方法：把中文注释替换成英文注释。

androidJavadocs task 错误：不允许使用自关闭元素。

解决方法：删除注释中`<br/>`、`<p/>`之类的标签，把整段注释内容使用`<p></p>`标签包裹起来。

Close时评估出现错误：Failed: Signature Validation

解决方法：重试上传GPG生成的keyId


## 备注

在正式发布时可能会出现403错误：

```
Release failed
Nexus returned an error: ERROR 403： Forbidden
```

这是因为之前创建的JIRA ticket的状态还未变成`RESOLVED`，等待可用既可。

<br/>
参考：

1. [Working with PGP signatures](http://central.sonatype.org/pages/working-with-pgp-signatures.html)
2. [Releasing the deployment](http://central.sonatype.org/pages/releasing-the-deployment.html)
3. [Release the deployment to the central repository](http://central.sonatype.org/pages/gradle.html#releasing-the-deployment-to-the-central-repository)

本文出自[2dxgujun](/)，转载时请注明出处及相应链接。