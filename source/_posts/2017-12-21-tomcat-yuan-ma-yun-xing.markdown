---
layout: post
title: "tomcat-源码运行"
date: 2017-12-21 10:57:30 +0800
comments: true
categories: 
---
#一.下载tomcat源码
##1.tomcat源码 
[tomcat-8.5.24源码地址](http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.24/src/apache-tomcat-8.5.24-src.zip)

##2.tomcat binary
[tomcat-8.5.24 bin地址](http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.24/bin/apache-tomcat-8.5.24.zip)，在运行时需要使用一些jar包（比如：servlet.jar），下载bin文件直接使用解压出来的lib比较方便。

#二.必要的配置
##1.创建目录结构
![目录结构](/images/tomcat-source-mune-struct.png)

##2.将tomcat bin文件解压出来的conf、lib等文件拷贝到catalina-home目录下

##3.创建pom.xml文件，用于引入必要的jar包，pom.xml文件内容如下
```xml
<?xml version="1.0" encoding="UTF-8"?>    
<project xmlns="http://maven.apache.org/POM/4.0.0"    
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    
    
    
    <modelVersion>4.0.0</modelVersion>    
    <groupId>org.apache.tomcat</groupId>    
    <artifactId>Tomcat8.0</artifactId>    
    <name>Tomcat8.0</name>    
    <version>8.0</version>    
    
    <build>    
        <finalName>Tomcat8.0</finalName>    
        <sourceDirectory>java</sourceDirectory>    
        <testSourceDirectory>test</testSourceDirectory>    
        <resources>    
            <resource>    
                <directory>java</directory>    
            </resource>    
        </resources>    
        <testResources>    
            <testResource>    
                <directory>test</directory>    
            </testResource>    
        </testResources>    
        <plugins>    
            <plugin>    
                <groupId>org.apache.maven.plugins</groupId>    
                <artifactId>maven-compiler-plugin</artifactId>    
                <version>2.0.2</version>
    
                <configuration>    
                    <encoding>UTF-8</encoding>    
                    <source>1.8</source>    
                    <target>1.8</target>    
                </configuration>    
            </plugin>    
        </plugins>    
    </build>    
    
    <dependencies>  
        <dependency>  
            <groupId>org.easymock</groupId>  
            <artifactId>easymock</artifactId>  
            <version>3.5</version>  
            <scope>test</scope>  
        </dependency>  
  
        <dependency>    
            <groupId>junit</groupId>    
            <artifactId>junit</artifactId>    
            <version>4.12</version>  
            <scope>test</scope>    
        </dependency>    
        <dependency>    
            <groupId>ant</groupId>    
            <artifactId>ant</artifactId>    
            <version>1.7.0</version>    
        </dependency>    
        <dependency>    
            <groupId>wsdl4j</groupId>    
            <artifactId>wsdl4j</artifactId>    
            <version>1.6.2</version>    
        </dependency>    
        <dependency>    
            <groupId>javax.xml</groupId>    
            <artifactId>jaxrpc</artifactId>    
            <version>1.1</version>    
        </dependency>    
        <dependency>    
            <groupId>org.eclipse.jdt.core.compiler</groupId>    
            <artifactId>ecj</artifactId>    
            <version>4.6.1</version>  
        </dependency>

    </dependencies>    
    
</project>
```

#三.导入文件到idea
![导入到idea后的效果](/images/idea-tomcat-source.png)

##1.编译，发现test/util/TestCookieFilter.java文件报错，直接删除或者注释掉

##2.配置Run/Debug Configurations
![Run/Debug Configurations](/images/idea-tomcat-conf.png)
###a.Main class选择：org.apache.catalina.startup.Bootstrap
###b.VM optioons:
```
-Dcatalina.home=/Users/yongduan/IdeaProjects/tomcat/catalina-home
-Dcatalina.base=/Users/yongduan/IdeaProjects/tomcat/catalina-home
-Djava.endorsed.dirs=/Users/yongduan/IdeaProjects/tomcat/catalina-home/endorsed
-Djava.io.tmpdir=/Users/yongduan/IdeaProjects/tomcat/catalina-home/temp
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.util.logging.config.file=/Users/yongduan/IdeaProjects/tomcat/catalina-home/conf/logging.properties
```

#四.执行验证
##1.启动tomcat
##2.访问locathost可以看到tomcat首页
