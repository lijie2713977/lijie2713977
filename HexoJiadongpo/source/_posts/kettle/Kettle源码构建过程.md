---
title: Kettle源码构建过程
date: 2016-05-22 14:43:49
tags: [开源项目,kettle]
categories: [开源项目,kettle]
---
#Kettle 源码构建过程#

&emsp;&emsp;Kettle 的源码托管在 Github 和 SVN上，但是托管在SVN上的源码自 5.0 之后就一直没有更新了，而托管在Github上的源码一直保持着更新状态，所以我猜 svn 上的源码并不会去进行维护了，我们以后只关注 git 上的源码就行。

&emsp;&emsp;Kettle 的二进制文件下载地址：
  http://sourceforge.net/projects/pentaho/files/Data%20Integration/
&emsp;&emsp;Kettle 源码地址:
  git:https://github.com/pentaho/pentaho-kettle
  svn:svn://source.pentaho.org/svnkettleroot/archive/Kettle/branches
&emsp;&emsp;其中二进制文件的下载版本分支与 git 上的源码分支是保持一致的，所以git上面的源码是跟着 kettle 的版本随时发布更新的，我们以后fork这个项目就可以一直获取最新发布的源码了。

**下面是我的构建过程(版本5.4)：**  
下载 ivy (http://ant.apache.org/ivy/download.html)。5.0版本之上的 kettle 项目结构与之前的版本项目结构完全不同，构建工具也由ant 变为了 ant + ivy。所以需要下载 ivy 来完成构建过程。将下载的 ivy-2.4.0.jar包放置到 ant_home/lib 下即可。  
从 git 上面 clone  kettle 5.4 的源码于某个目录。命令行进入到源码主目录，如下所示：  
&emsp;&emsp;执行命令：ant clean-all resolve create-dot-classpath。  命令会执行很久很久(资源在国外，如果自己有 vpn 加速会快点)，而且会时不时的报错，找不到 jar 包。当遇到找不到jar 包时，我是自行到kettle 私服(http://nexus.pentaho.org/content/groups/omni/)中去下对应版本jar包，然后放置到ivy 的本地仓库中，并删掉 ivy 本地缓存中的源文件和未下载成功的垃圾文件，然后重复此命令进行构建。本地缓存文件位于C:\Users\${username}\.ivy2\cache 。一直执行此命令，直到出现 successful 的提示。

执行命令：ant dist ，该命令会根据源码生成一份对应的kettle 版本。同样的，在执行此命令的过程中，也会经常报各种错误，也需要自己去私服中找对应的jar包放置到对应的本地缓存目录。构建成功后，在根目录下会生成 一个 dist 文件夹，打开里面的spoon.bat 就可以使用 kettle 了，我下的5.4 版本的界面与之前的大有不同，感觉比以前的好看些，如下所示：


到现在为止，就已经可以用源码构建了。下一步就是搭建好源码，kettle 的源码中分别提供了.project 文件和 .ipr 文件，所以可以用 eclipse和 idea 进行搭建。我是用的 idea 搭建的，eclipse 搭建应该会更简单点。

在搭建之前，我删掉了kettle 中所有的.gitignore、.gitignoreattribute、.template、.project文件，然后在 .ipr 文件中注释掉版本管理的信息和未提供的插件项目信息，这样在导入源码的时候不会提示错误，如下所示：



源码导入idea 中之后，会出现很多错误，需要手动导入一些jar包，由于里面有很多测试文件，所以我们需要手动依赖 /core/test_lib 文件夹里面的jar 包，范围指定为 test；



从 libext 目录中复制 win64 位的swt.jar 包到 lib 目录，然后删除 swt_x86_64.jar 包；

复制 ojdbc.jar 包到lib目录(为了解决登录资源库)；

复制 dist/ui/*.xul 文件到 ui/目录；





新建simple-jndi 空目录；



到了这里，源码编译、调试就没问题了，程序启动的入口为org.pentaho.di.ui.spoon.Spoon.java，运行main入口函数即可启动 kettle，如下所示：



再来欣赏下折腾摸索了这么久才弄出来的东西，确实比以前的好看些了，整体风格感觉统一了，而且向上兼容以前版本的，我用以前版本的转换在5.4 下测试了完全没问题，同时我也把以前写过的插件按照之前的机制进行处理，插件也是能加载出来并正常使用的，所以新的版本下的插件机制并没有发生变化:



总结：上面这些步骤中好几个步骤是跑代码调试才知道的问题，所以调试还是一如既往的重要哈。

