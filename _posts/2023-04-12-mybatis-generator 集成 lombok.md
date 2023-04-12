---
layout:     post
title:      mybatis-generator 集成 lombok
subtitle:   mybatis-generator 集成 lombok
date:       2023-04-12
author:     W
header-img: img/mybatis.jpg
catalog: true
tags:
    - mybatis
    - mybatis-generator
    - lombok
---

## 前言

使用`Lombok`工具可以在生成实体类时省掉getter/setter以及构造方法等等代码，使得代码整体更加简洁优雅。在使用`MyBatis`时，可以利用generator映射数据库反向生成相对应得实体类以及mapper接口和mapper.xml文件

## 正文

### 前期准备

- 工具类（LombokPlugin.java）

```java
package org.mybatis.generator.plugins;

import org.mybatis.generator.api.IntrospectedColumn;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.PluginAdapter;
import org.mybatis.generator.api.dom.java.Field;
import org.mybatis.generator.api.dom.java.FullyQualifiedJavaType;
import org.mybatis.generator.api.dom.java.Interface;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.TopLevelClass;
import org.mybatis.generator.internal.util.StringUtility;

import java.text.SimpleDateFormat;
import java.util.*;

/**
 * LombokPlugin
 *
 * @author Wayen
 * 2023-04-12 8:52
 **/
public class LombokPlugin extends PluginAdapter {

    public LombokPlugin() {
    }

    @Override
    public boolean validate(List<String> list) {
        return true;
    }

    @Override
    public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {

        // domain import
        topLevelClass.addImportedType("lombok.Data");

        // domain description
        topLevelClass.addJavaDocLine("/**");
        String remarks = introspectedTable.getRemarks();
        if (StringUtility.stringHasValue(remarks)) {
            String[] remarkLines = remarks.split(System.getProperty("line.separator"));
            for (String remarkLine : remarkLines) {
                topLevelClass.addJavaDocLine(" * tableRemark: " + remarkLine);
            }
        }
        topLevelClass.addJavaDocLine(" * tableName: " + introspectedTable.getFullyQualifiedTable());
        topLevelClass.addJavaDocLine(" *");
        topLevelClass.addJavaDocLine(" * @author wwei-generator-lombok");
        topLevelClass.addJavaDocLine(" * " + getDateString());
        topLevelClass.addJavaDocLine(" */");

        // domain annotation
        topLevelClass.addAnnotation("@Data");

        return true;
    }

    @Override
    public boolean modelFieldGenerated(Field field, TopLevelClass topLevelClass, IntrospectedColumn introspectedColumn,
                                       IntrospectedTable introspectedTable, ModelClassType modelClassType) {

        // field description
        field.addJavaDocLine("/**");
        String remarks = introspectedColumn.getRemarks();
        if (StringUtility.stringHasValue(remarks)) {
            String[] remarkLines = remarks.split(System.getProperty("line.separator"));
            for (String remarkLine : remarkLines) {
                field.addJavaDocLine(" * " + remarkLine);
            }
        } else {
            field.addJavaDocLine(" * ");
        }
        field.addJavaDocLine(" */");

        return true;
    }

    @Override
    public boolean clientGenerated(Interface interfaze, IntrospectedTable introspectedTable) {

        // mapper import
        interfaze.addImportedType(new FullyQualifiedJavaType("org.apache.ibatis.annotations.Mapper"));

        // mapper description
        interfaze.addJavaDocLine("/**");
        interfaze.addJavaDocLine(" * @author wwei-generator-lombok");
        interfaze.addJavaDocLine(" * " + getDateString());
        interfaze.addJavaDocLine(" */");

        // mapper annotation
        interfaze.addAnnotation("@Mapper");

        return true;
    }

    @Override
    public boolean modelSetterMethodGenerated(Method method, TopLevelClass topLevelClass,
                                              IntrospectedColumn introspectedColumn,
                                              IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        // no getter
        return false;
    }

    @Override
    public boolean modelGetterMethodGenerated(Method method, TopLevelClass topLevelClass,
                                              IntrospectedColumn introspectedColumn,
                                              IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        // no setter
        return false;
    }

    @Override
    public boolean modelExampleClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        // example description
        topLevelClass.addJavaDocLine("/**");
        topLevelClass.addJavaDocLine(" * @author wwei-generator-lombok");
        topLevelClass.addJavaDocLine(" * " + getDateString());
        topLevelClass.addJavaDocLine(" */");
        return true;
    }

    protected String getDateString() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        return sdf.format(new Date());
    }
}
```

- 核心工具包（mybatis-generator-core-1.4.0.jar）

- 编译工具类

  ```java
  javac -cp ../mybatis-generator-core-1.4.0.jar ../LombokPlugin.java
  ```

- 将编译生成的.class文件放入jar包中org.mybatis.generator.plugins目录下

### pom文件

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.4.0</version>
    <dependencies>
    <!--修改后的核心包-->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.4.0</version>
        </dependency>
    </dependencies>
    <!--防止反复生成-->
    <configuration>
        <!--允许移动生成的文件-->
        <verbose>true</verbose>
        <!--是否允许自动覆盖文件-->
        <overwrite>true</overwrite>
        <!--配置文件-->
        <configurationFile>
            ../generatorConfig.xml
        </configurationFile>
    </configuration>
</plugin>
```

### generatorConfig

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="Postgresql" targetRuntime="MyBatis3">
        <!-- 使用自定义的插件 -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <plugin type="org.mybatis.generator.plugins.LombokPlugin"/>
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin"/>
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!-- 数据库-->
        <jdbcConnection
                driverClass="驱动"
                connectionURL="地址"
                userId="账号"
                password="密码">
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- 生成实体类的包名和位置-->
        <javaModelGenerator targetPackage="包名"
                            targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置-->
        <sqlMapGenerator targetPackage="包名" targetProject="src/main">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- 生成DAO的包名和位置-->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="包名"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <table tableName="表名" domainObjectName="对象名"/>
    </context>
</generatorConfiguration>
```

### 生成文件

两种方式

1. IDEA的maven窗口-->Plugins-->mybatis-generator-->mybatis-generator:generator
2. Terminal窗口使用maven命令：mvn mybatis-generator:generator
