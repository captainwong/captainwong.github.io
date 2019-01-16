---
layout: post
title:  "加速Android开发"
subtitle: "fuck gfw"
date:   "2019-01-16"
author: "cj"
tags:
    android
    AndroidStudio
    gradle
    maven
    google
    aliyun
---

# 加速 `Android` 开发

即使有梯子，速度依然不让人满意。

## 加速 `dl.google.com`

我以前都是开着代理更新 AS，速度不咋样。后来搜索知道了原来 `dl.google.com` 在国内有服务器啊！

打开[多个地点Ping服务器,网站测速 - 站长工具](http://ping.chinaz.com/)，检测 `dl.google.com`，挑选物理坐标距离最近、相同同运营商、延迟最低的 IP地址。

例如我这是西安移动网络，结果只有 `203.208.50.73`，写到 `C:\Windows\System32\drivers\etc\hosts` 内：

`203.208.50.73 dl.google.com`

再次更新AS或SDK时速度飞起。

## 加速 `gradle`

1. 下载

    即使开着代理，让 AS 自动更新 Gradle 也巨慢无比，虽然手动下载还是很快的。开代理下载好适用的版本。这个每次等 AS 提示更新 Gradle 的时候看要用哪个版本就下载哪个。比如我这次要更新的链接：
    [http://services.gradle.org/distributions/gradle-4.10.1-all.zip](http://services.gradle.org/distributions/gradle-4.10.1-all.zip)。
    
    下载好后放到 `C:\Users\capta\.gradle\wrapper\dists\gradle-4.10.1-all\455itskqi2qtf0v2sja68alqd` 文件夹内（capta是用户名）。
    
    至于 `455itskqi2qtf0v2sja68alqd` 是啥意思，我搜了半天没找到相关的结果，Google 搜索技巧还是不到家啊。。。
    
    按笨办法来，让 AS 先自动更新一会，会生成这个文件夹，退出 AS；进入这个文件夹后把其他文件都删除，将手动下载好的 `gradle-4.10.1-all.zip` 放到这里，手动解压下，再启动 AS 即可。

2. 阿里云加速

    使用阿里云的[公共代理库](https://help.aliyun.com/document_detail/102512.html?spm=a2c40.aliyun_maven_repo.0.0.36183054OD162U)加速。参考[大家都是怎样处理Gradle中的这个文件下载慢的问题的？](https://www.zhihu.com/question/37810416)

    在 `C:\Users\capta\.gradle` 内新建 `init.gradle`，写入如下内容：
    ```gradle
    allprojects{
        repositories {
            def ALIYUN_REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public'
            def ALIYUN_JCENTER_URL = 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
            all { ArtifactRepository repo ->
                if(repo instanceof MavenArtifactRepository){
                    def url = repo.url.toString()
                    if (url.startsWith('https://repo1.maven.org/maven2')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                        remove repo
                    }
                    if (url.startsWith('https://jcenter.bintray.com/')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                        remove repo
                    }
                }
            }
            maven {
                url ALIYUN_REPOSITORY_URL
                url ALIYUN_JCENTER_URL
            }
        }
    }
    ```

## Fuck GFW!