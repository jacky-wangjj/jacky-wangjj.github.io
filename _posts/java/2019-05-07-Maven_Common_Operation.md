---
layout: post
title: Maven可执行程序打包及包冲突解决
date: 2019-05-07
tags: maven
---  
### Spring Boot添加本地jar包
打包./lib/llw-base-rpc.jar到jar包中。
```xml
<dependencies>
    <dependency>
        <groupId>llw-base-rpc</groupId>
        <artifactId>llw-base-rpc</artifactId>
        <version>1.0</version>
        <scope>system</scope>
        <systemPath>${project.basedir}/lib/llw-base-rpc.jar</systemPath>
    </dependency>
</dependencies>
```   
把includeSystemScope属性设置成true。
```xml
<build>    
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <compilerArguments>
                    <extdirs>${project.basedir}/lib</extdirs>
                </compilerArguments>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <includeSystemScope>true</includeSystemScope>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <targetPath>BOOT-INF/lib/</targetPath>
            <includes>
                <include>**/*.jar</include>
            </includes>
        </resource>

        <resource>
            <directory>src/main/resources</directory>
            <targetPath>BOOT-INF/</targetPath>
            <includes>
                <include>**/*.properties</include>
            </includes>
        </resource>
    </resources>
</build>
```
[参考资料](https://www.cnblogs.com/stm32stm32/p/9973325.html)    

### Spring Boot修改默认Logback替换为log4j
1) 去除对默认日志的依赖   
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <!-- 去除对默认日志的依赖 -->
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

2) 添加对log4j的依赖   
  ```xml
  <!-- 添加 log4j 依赖 -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-log4j</artifactId>
      <version>1.3.8.RELEASE</version>
  </dependency>
  <!-- 或添加如下依赖 -->
  <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-slf4j-impl</artifactId>
      <version>2.6.2</version>
  </dependency>
  ```
  到此，Spring Boot项目的日志框架就以及替换成log4j了，如需要定义日志输出，只需添加log4j.properties文件并添加相应配置即可。   

  然后需要去除项目中其他依赖中引入的logback-classic包，Maven Projects -> Dependencies可以看到pom.xml中每一个<dependency>引入的隐含包，然后在对应<dependency>中添加<exclusion>去除logback-classic和logback-core包。    

  ```xml
  <exclusions>
      <exclusion>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-classic</artifactId>
      </exclusion>
      <exclusion>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-core</artifactId>
      </exclusion>
  </exclusions>
  ```
  如图所示：       
  ![](https://jacky-wangjj.github.io/images/blog/maven/springboot-exclusion-logback.png#pic_center)

### Maven在MANIFEST.MF文件中的Class-Path添加指定路径。
添加manifestEntries
```xml
<build>
    <resources>
        <!--指定src/main/resources资源要过滤-->
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <!-- 可执行jar插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>  
                <archive>  
                    <manifest>  
                        <mainClass>com.dongwei.test.Main</mainClass>  
                        <addClasspath>true</addClasspath>  
                        <classpathPrefix>lib/</classpathPrefix>  
                    </manifest>  
                    <manifestEntries>  
                        <Class-Path>/usr/lib/phoenix/</Class-Path>  
                    </manifestEntries>  
                </archive>  
            </configuration>
        </plugin>
        <!-- maven资源文件复制插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <version>2.7</version>
            <executions>
                <execution>
                    <id>copy-resources</id>
                    <!-- here the phase you need -->
                    <phase>package</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/etc</outputDirectory>
                        <resources>
                            <resource>
                                <directory>src/main/resources</directory>
                                <includes>
                                    <exclude>**/*.xml</exclude>
                                    <exclude>**/*.conf</exclude>
                                    <exclude>**/*.properties</exclude>
                                </includes>
                                <filtering>true</filtering>
                            </resource>
                        </resources>
                        <encoding>UTF-8</encoding>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <!-- 依赖包插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/etc/lib</outputDirectory>
                        <!-- 是否不包含间接依赖 -->
                        <excludeTransitive>false</excludeTransitive>
                        <!-- 忽略版本 -->
                        <stripVersion>false</stripVersion>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
添加后的效果
```
Manifest-Version: 1.0  
Archiver-Version: Plexus Archiver  
Created-By: Apache Maven  
Built-By: wei.dong  
Build-Jdk: 1.6.0_24  
Main-Class: com.dongwei.test.Main  
Class-Path: /usr/lib/phoenix/ lib/spring-core-3.0.5.RELEASE.jar lib/spring-asm-3.0.5.R  
 ELEASE.jar lib/commons-logging-1.1.1.jar lib/spring-context-3.0.5.REL  
 EASE.jar lib/spring-aop-3.0.5.RELEASE.jar lib/aopalliance-1.0.jar lib  
 /spring-expression-3.0.5.RELEASE.jar lib/spring-context-support-3.0.5  
 .RELEASE.jar lib/spring-beans-3.0.5.RELEASE.jar lib/spring-jdbc-3.0.5  
 .RELEASE.jar lib/spring-tx-3.0.5.RELEASE.jar lib/log4j-1.2.14.jar lib  
 /slf4j-nop-1.4.3.jar lib/slf4j-api-1.4.3.jar lib/commons-lang-2.5.jar  
  lib/commons-dbcp-1.2.2.jar lib/commons-pool-1.3.jar lib/commons-io-1  
 .4.jar lib/commons-digester-2.0.jar lib/commons-beanutils-1.8.0.jar l  
 ib/commons-configuration-1.6.jar lib/commons-collections-3.2.1.jar li  
 b/commons-beanutils-core-1.8.0.jar lib/quartz-1.8.4.jar lib/jta-1.1.j  
 ar lib/mysql-connector-java-5.1.12.jar
```

### 将Maven项目打包成可执行jar文件，配置文件打包在jar包外
[参考资料](https://blog.csdn.net/e_wsq/article/details/64537091)    
- mvn package (一个jar包和一个lib文件夹)
  ```xml
  <build>  
      <plugins>  
          <plugin>  
              <groupId>org.apache.maven.plugins</groupId>  
              <artifactId>maven-jar-plugin</artifactId>  
              <version>2.4</version>  
              <configuration>  
                  <archive>  
                      <manifest>  
                          <addClasspath>true</addClasspath>  
                          <classpathPrefix>lib/</classpathPrefix>  
                          <mainClass>com.tang.CSVUtils</mainClass>  
                      </manifest>  
                  </archive>  
              </configuration>  
          </plugin>  
          <plugin>  
              <groupId>org.apache.maven.plugins</groupId>  
              <artifactId>maven-dependency-plugin</artifactId>  
              <executions>  
                  <execution>  
                      <id>copy</id>  
                      <phase>package</phase>  
                      <goals>  
                          <goal>copy-dependencies</goal>  
                      </goals>  
                      <configuration>  
                          <outputDirectory>${project.build.directory}/lib</outputDirectory>  
                      </configuration>  
                  </execution>  
              </executions>  
          </plugin>  
      </plugins>  
  </build>
  ```
  其中lib就是第三方jar包的目录，只需拷贝两个（lib和testLog4j-0.1.jar）即可放到其他地方用了。    

- mvn package (推荐)(一个zip包，里面有一个jar包，一个lib文件夹和一个conf文件夹)
  conf/package.xml(conf文件夹和pom.xml在同一级目录)
  ```xml
  <assembly>    
      <id>bin</id>    
      <!-- 最终打包成一个用于发布的zip文件 -->    
      <formats>    
          <format>zip</format>    
      </formats>    

      <!-- Adds dependencies to zip package under lib directory -->    
      <dependencySets>    
          <dependencySet>    
              <!--  
                 不使用项目的artifact，第三方jar不要解压，打包进zip文件的lib目录  
             -->    
              <useProjectArtifact>false</useProjectArtifact>    
              <outputDirectory>lib</outputDirectory>    
              <unpack>false</unpack>    
          </dependencySet>    
      </dependencySets>    

      <fileSets>    
          <!-- 把项目相关的说明文件，打包进zip文件的根目录 -->    
          <fileSet>    
              <directory>${project.basedir}</directory>    
              <outputDirectory>/</outputDirectory>    
              <includes>    
                  <include>README*</include>    
                  <include>LICENSE*</include>    
                  <include>NOTICE*</include>    
              </includes>    
          </fileSet>    

          <!-- 把项目的配置文件，打包进zip文件的config目录 -->    
          <fileSet>    
              <directory>${project.basedir}\conf</directory>    
              <outputDirectory>conf</outputDirectory>    
              <includes>    
                  <include>*.xml</include>    
                  <include>*.properties</include>  
                  <include>*.key</include>   
              </includes>    
          </fileSet>    

          <!-- 把项目的脚本文件目录（ src/main/scripts ）中的启动脚本文件，打包进zip文件的跟目录 -->    
          <fileSet>    
              <directory>${project.build.scriptSourceDirectory}</directory>    
              <outputDirectory></outputDirectory>    
              <includes>    
                  <include>startup.*</include>    
              </includes>    
          </fileSet>    

          <!-- 把项目自己编译出来的jar文件，打包进zip文件的根目录 -->    
          <fileSet>    
              <directory>${project.build.directory}</directory>    
              <outputDirectory></outputDirectory>    
              <includes>    
                  <include>*.jar</include>    
              </includes>    
          </fileSet>    
      </fileSets>    
  </assembly>
  ```
  pom.xml
  ```xml
  <build>    
      <plugins>  
          <plugin>  
              <groupId>org.apache.maven.plugins</groupId>  
              <artifactId>maven-compiler-plugin</artifactId>  
              <version>3.1</version>  
              <configuration>  
                  <compilerVersion>1.6</compilerVersion>  
                  <source>1.6</source>  
                  <target>1.6</target>  
              </configuration>  
          </plugin>    
          <!-- The configuration of maven-jar-plugin -->    
          <plugin>    
              <groupId>org.apache.maven.plugins</groupId>    
              <artifactId>maven-jar-plugin</artifactId>    
              <version>2.4</version>    
              <!-- The configuration of the plugin -->    
              <configuration>    
                  <!-- Configuration of the archiver -->    
                  <archive>    
                      <!-- do not include pom.xml and pom.properties in the jar package -->    
                      <addMavenDescriptor>false</addMavenDescriptor>    
                      <!-- Manifest specific configuration -->    
                      <manifest>    
                          <!-- put third party jar package into the classpath of manifest -->    
                          <addClasspath>true</addClasspath>    
                          <!-- the prefix of the jar items in the classpath, it depends on the location(folder) of jar files -->    
                          <classpathPrefix>lib/</classpathPrefix>    
                          <!-- main class of the jar package-->    
                          <mainClass>com.tang.your-Main-class</mainClass>    
                      </manifest>    
                  </archive>    
                  <!-- excludes some files -->    
                  <excludes>    
                      <exclude>${project.basedir}/xml/*</exclude>    
                  </excludes>    
              </configuration>    
          </plugin>    
          <!-- The configuration of maven-assembly-plugin -->    
          <plugin>    
              <groupId>org.apache.maven.plugins</groupId>    
              <artifactId>maven-assembly-plugin</artifactId>    
              <version>2.4</version>    
              <!-- The configuration of the plugin -->    
              <configuration>    
                  <!-- Specifies the configuration file of the assembly plugin -->    
                  <descriptors>    
                      <descriptor>conf/package.xml</descriptor>    
                  </descriptors>    
              </configuration>    
              <executions>    
                  <execution>    
                      <id>make-assembly</id>    
                      <phase>package</phase>    
                      <goals>    
                          <goal>single</goal>    
                      </goals>    
                  </execution>    
              </executions>    
          </plugin>     
      </plugins>    
  </build>   
  ```
  解压zip包后得到conf文件夹，lib文件夹，jar包。    

### Java虚拟机（JVM）寻找Class的顺序
#### 1. Bootstrap classes
属于Java核心的class，如rt.jar，由JVM Bootstrap class loader来载入，一般放置在{java_home}/jre/lib目录下。  

#### 2. Extension classes
基于Java扩展机制，用来扩展Java核心功能模块，一般放置在{java_home}/jre/lib/ext目录下。    

#### 3. User classes
第三方java程序包，通过命令行-classpath或-cp，或者通过CLASSPATH环境变量来引用。
User class路径搜索的顺序、优先级如下：     
- 缺省值，class所在当前目录。  
- CLASSPATH环境变量设置的路径，CLASSPATH的值会覆盖缺省值。  
- 命令行-classpath或-cp的值，如果使用了这两个命令行参数之一，它的值会覆盖环境变量CALSSPATH的值。  
- 通过java -jar来运行一个可执行的jar包，则当前jar包会覆盖上面所有的值，换句话说-jar后面所跟的jar包的优先级别最高，如果指定了-jar选项，所有环境变量和命令行指定的搜索路径都将被忽略。JVM APPClassloader将只会以jar包为搜索范围，若想使用第三方jar包，需将第三方jar包路径写入jar包中MANIFEST.MF文件的Class-Path路径中。  

### 添加第三方jar包的几种方法
Java定义了三种级别的class，分别为BootStrap class, Extend class, User class。  
基于三种java不同级别的class扩展机制，有三种不同的方案：  
- BootStrap class扩展
  Java 命令行提供了如何扩展bootStrap 级别class的简单方法.  
  -Xbootclasspath:基本核心的Java class 搜索路径.不常用,否则要重新写所有Java 核心class  
  -Xbootclasspath/a: 后缀在核心class搜索路径后面.常用.  
  -Xbootclasspath/p:前缀在核心class搜索路径前面.不常用,避免引起不必要的冲突.  
  语法如下:
  java –Xbootclasspath/a:/path/myclass/account.jar: -jar yourself.jar(Unix用:号隔开)  
  java –Xbootclasspath:/d:/myclass/account.jar; -jar yourself.jar(Window用;号隔开)  

- Extend class扩展
  Java exten class 存放在{Java_home}\jre\lib\ext 目录下.当调用Java时,对扩展class路径的搜索是自动的.总会搜索的.这样,解决的方案就很简单了,将所有要使用的第三方的jar包都复制到ext 目录下.    

- User class扩展  
  当使用-jar执行可执行Jar包时,JVM将Jar包所在目录设置为codebase目录,所有的class搜索都在这个目录下开始.所以如果使用了其他第三方的jar包,一个比较可以接受的可配置方案,就是利用jar包的Manifest扩展机制.步骤如下  
  1.将需要的第三方的jar包,复制在同可执行jar所在的目录或某个子目录下.  
  比如:jar 包在 d:\crm\luncher.jar 那么你可以把所有jar包复制到d:\crm目录下或d:\crm\lib 子目录下.  
  2.修改Manifest 文件  
  在Manifest.mf文件里加入如下行   
  Class-Path:classes12.jar lib/class12.jar  
  Class-Path 是可执行jar包运行依赖的关键词.详细内容可以参考http://java.sun.com/docs/books/tutorial/ext/index.html       

  直接使用命令行模式来指定classpath以及要运行的main方法   
  **windows:**   
  java -cp "Test.jar;lib/\*" my.package.MainClass   
  **Under Linux:**  
  java -cp "Test.jar:lib/\*" my.package.MainClass  
