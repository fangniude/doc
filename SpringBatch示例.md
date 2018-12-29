# Spring Batch示例

前段时间用Spring Batch做了个数据同步，为了提高性能，也简单翻了一下源码，对源码的分析，会放在另外的文章中细说，本文简单写个示例。

示例是从Oracle中读取数据，写到ArangoDB中。为了性能好一些，用了

# 1. maven依赖

同步是一个单独的服务，使用Spring Boot来做的，因为要与其他项目使用同样版本的Spring，所以用的Spring版本有点老。

以下是maven依赖，为了清晰，我删掉了些不相关的依赖。

``` xml

    <properties>
        <spring.boot.version>1.3.5.RELEASE</spring.boot.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>oracle</groupId>
            <artifactId>ojdbc6</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
            <version>${spring.boot.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.codehaus.jettison</groupId>
                    <artifactId>jettison</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
        <dependency>
            <!-- 用1.3.1有问题，使用1.1 -->
            <groupId>org.codehaus.jettison</groupId>
            <artifactId>jettison</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${spring.boot.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
            <scope>compile</scope>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.6.1</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.6.1</version>
        </dependency>
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-models</artifactId>
            <version>1.5.10</version>
        </dependency>
    </dependencies>
```

Oracle数据源



