---
title: maven打包指定main-class
tags: ['Java']
category: java
---

maven打包时候指定运行主类，pom.xml配置如下：

```XML
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>com.sniper.neo4j.util.Neo4jServerUtil</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>

```
