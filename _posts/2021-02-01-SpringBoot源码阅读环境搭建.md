---
layout: post
title:  SpringBoot系列之源码阅读环境搭建
date: 2021-02-01 21:47:00 +0800
categories: SpringBoot源码解析
tag: springboot
---

* content
{:toc}

前言                         {#前言}
===================================
这周一领导突然发微信说让我准备这周四的部门间知识分享，由于平时没怎么总结，突然间不知道分享什么。情急之下想到了项目使用的SptringBoot框架。便准备讲下boot的启动及自动化配置原理。
在准备的过程中突发奇想，想整理出关于boot的一系列知识总结。顺便养写总结的习惯。


源码下载                      {#源码下载}
=======================================
先在github找到spingboot的官方项目，将其fork到自己的github，同时注册个gitee的账号并将boot的源码同步到gitee上，这样下载比较快。gitee是国内免费的开源代码托管仓库，类似于github
![gitee导入github代码]({{ '/images/springboot-gitee.jpg' | prepend: site.baseurl  }})



idea导入源码					{#idea导入源码}
==========================================
使用idea下载代码，特别说明下代码下载完成后会自动编译，此时需要手动停止。下载后代码处于master分支。需要选择relese分支进行切换，relese分支属于发布分支相对稳定。我选择的是v2.3.4-RELEASE。使用` git checkout -b v2.3.4-RELEASE v2.3.4-RELEASE` 进行切换。方便日后提交相关注释及代码。
此版本使用使用gradle进行编译。同时需要将仓库修改为国内仓库，首选阿里云。
- 默认使用系统自带gradle，使用项目内的配置,下图为源码编译的配置。
![gradle配置]({{ '/images/springboot-gradle.jpg' | prepend: site.baseurl  }})
- 在项目根目录的settings.gradle配置文件中搜索`io.spring.gradle-enterprise-conventions` 并注掉对应的依赖，否则会报错
- 找到项目使用的.gradle文件夹并在文件夹下新建init.gradle文件内容为：
```
# 配置所有项目使用的仓库
allprojects{
    repositories {
        def ALIYUN_REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public'
        def ALIYUN_JCENTER_URL = 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('http://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com/') || url.startsWith('http://jcenter.bintray.com/')) {
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
buildscript{
        repositories {
            def ALIYUN_REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public'
            def ALIYUN_JCENTER_URL = 'http://maven.aliyun.com/nexus/content/repositories/jcenter'
            all { ArtifactRepository repo ->
                if(repo instanceof MavenArtifactRepository){
                    def url = repo.url.toString()
                    if (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('http://repo1.maven.org/maven2')) {
                        project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                        remove repo
                    }
                    if (url.startsWith('https://jcenter.bintray.com/') || url.startsWith('http://jcenter.bintray.com/')) {
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
}
```
- 最后使用gradle进行编译
![编译]({{ '/images/springboot-gradle-build.jpg' | prepend: site.baseurl }})
最后一直等到编译成功，期间可以查看编译日志，若长时间在下载同一个jar包，则可以考虑将项目根目录下的setttings.gradle、buid.gradle内的仓库改为maven仓库，若还不成功则是之前配置 的全局仓库没生效，需要重新找对应的位置进行配置后重新编译。
