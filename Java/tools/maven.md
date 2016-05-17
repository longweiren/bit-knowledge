##maven 使用手册##

[maven](https://maven.apache.org) 不仅仅是一个构建工具，或依赖包管理软件，更是一个优秀的项目管理工具，集成了构建、文档生成、报告、依赖管理、版本控制集成、发布集成、持续集成管理、bug管理系统集成、多项目管理等。

* 安装

> 从 [maven](https://maven.apache.org) 官网下载压缩包解压即可。
> 如解压后的目录可能为：
> /apache-maven-3.3.9
> ---/bin
> ---/boot
> ---/conf
> ---/lib
> 
> 把 ../apache-maven-3.3.9/bin 加入系统变量 `PATH`
> 设置系统变量 `M2-HOME`  M2-HOME= ../apache-maven-3.3.9
> 
> 在命令行输入 `mvn -version` 验证安装结果。如果能正常输入 maven 的信息和 JDK 的信息，说明 maven 安装成功。如果是 Java 系统变量设置问题，安装 JDK 并设置 `PATH` 和 `JAVA-HOME` 后重新验证。
> 

* 创建 Maven 项目

> 创建普通 Java 项目(使用 Archetype 插件 )
> mvn archetype:generate -DgroupId=packageName -DartifactId=projectName  
> 创建 web 项目(使用 Archetype 插件 )
> mvn archetype:generate -DgroupId=packageName -DartifactId=webappName -DarchetypeArtifactId=maven-archetype-webapp

* 理解  pom.xml

> POM(Project Object Model) 是一个描述项目相关配置信息的xml文件

```
<project>
    ...
    <!-- Basics Infomation-->
    <!-- groupId 标识一个组织或项目，如: org.apache.commons-->
    <groupId></groupId>
    <!-- artifactId 项目名称，如：commons-collections4-->
    <artifactId></artifactId>
    <version></version>
    <packaging>jar|war|ear|rar|par|ejb|pom|maven-plugin</packaging>
    <dependencies></dependencies>
    <parent></parent>
    <dependencyManagement></dependencyManagement>
    <modules></modules>
    <properties></properties>

    ...
    <!-- Build Settings-->
    <build></build>
    <reporting></report>

    ...
    <!-- More Project Infomation-->
    <name></name>
    <description></description>
    <url></url>
    <licenses></licenses>
    <organization></organization>
    <developers></developers>
    <contributors></contributors>

    ...
    <!-- Environment Settings-->
    <issueManagement></issueManagement>
    <ciManagement></ciManagement>
    <scm></scm>
    <repositories></repositories>
    <pluginRepositories></pluginRepositories>
    <distributionManagement></distributionManagement>
    <profiles></profiles>

</project>
```

依赖管理
```
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <!-- version 取值方式： xx, [x1, x2], (x1, x2), (x1, x2], [, x2]...-->
        <version>4.0</version>
        <type>jar</type>
        <scope>compile|provided|runtime|test|system</scope>
        <optional>true</optional>
    </dependency>
</dependencies>
```

能被子项目集成的配置
```
dependencies
developers and contributors
plugin lists
reports lists
plugin executions with matching ids
plugin configuration
```

父子项目

```
<!-- pom.xml of parent -->
<project>
    <modelVersion>1.0.0</modelVersion>

    <groupId>com.kopcoder</groupId>
    <artifactId>mvndemo</artifactId>
    <version>1.0</version>
    <!-- 父项目只能是pom -->
    <packaging>pom</packaging>

    <modules>
        <module>distrib_query</module>
    </modules>
</project>

<!-- pom.xml of child -->
<project>
    <modelVersion>1.0.0</modelVersion>

    <parent>
        <groupId>com.kopcoder</groupId>
        <artifactId>mvndemo</artifactId>
        <version>1.0</version>

        <!-- 如果已 install 到仓库，则不需配置-->
        <relativePath>../mvndemo</relativePath>
    </parent>

    <artifactId>distrib_query</artifactId>
</project>
```

集成 Bug管理系统
```
<!--Bugzilla jira-->
<issueManagement>
    <system>Bugzilla</system>
    <url>http://127.0.0.1/bugzilla/
</issueManagement>
```

集成 SCM(software configuration management)
```
<scm>
    <!--有 读 权限(read access)的连接-->
    <connection>scm:git:git://github.com/path_to_repository</connection>
    <!--有 写 权限(write access)的连接-->
    <developerConnection>scm:git:https://github.com/path_to_repository</developerConnection>
    <tag>HEAD</tag>
    <url>http://127.0.0.1/websvn/my-project</url>
</scm>
```

集成 CI(持续集成管理)
```
<!-- Jenkins Hudson-->
<ciManagement>
    <system>jenkins</system>
    <url>https://ci.jenkins-ci.org/job/jenkins_main_trunk</url>
</ciManagement>
```

仓库管理
```
<repositories>
    <repository>
        <id>codehausSnapshots</id>
        <name>Codehaus Snapshots</name>
        <url>http://snapshots.maven.codehaus.org/maven2</url>
        <layout>default</layout>
        <releases>
            <enabled>false</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
    </repository>
</repositories>
```

发布管理
```
<distributionManagement>
    <repository>
        <uniqueVersion>false</uniqueVersion>
        <!--可以在settings.xml里配置corp1的访问信息-->
        <id>corp1</id>
        <name>Corporate Repository</name>
        <url>scp://repo/maven2</url>
        <layout>default</layout>
    </repository>
    <snapshotRepository>
        <uniqueVersion>true</uniqueVersion>
        <id>propSnap</id>
        <name>Propellors Repository</name>
        <url>sftp://propellers.net/maven</url>
        <layout>legacy</layout>
    </snapshotRepository>
</distributionManagement>

<settings>
    <servers>
        <server>
            <id>corp1</id>
            <username></username>
            <!--default is ~/.ssh/id_dsa-->
            <privateKey></privateKey>
            <passphrase></passphrase>
        </server>
    </servers>
</settings>
```

* 理解 settings.xml

> pom.xml 是项目的配置文件，settings.xml 是 maven 的配置文件。
> 
> 一个项目只有一个 pom.xml。而在 maven 中，可能有两个 settings.xml
> ${maven.home}/conf/settings.xml (global settings)
> ${user.home}/.m2/settings.xml   (user settings，优先级更高)

```
<settings>
    <!-- 本地仓库地址 -->
    <localRepository>${user.home}/.m2/repository</localRepository>
    <!-- mvn 是否需要与用户输入交互-->
    <interactiveMode></interactiveMode>
    <usePluginRegistry />
    <!-- mvn 构建系统是否能在离线模式下操作。当构建服务器因网络原因或安全原因不能连接远程仓库时，需要使用该配置项-->
    <offline />
    <pluginGroups />
    <servers />
    <mirrors />
    <proxies />
    <profiles />
    <activeProfiles>
        <activeProfile>env-test</activeProfile>
    </activeProfiles>
</settings>
```

maven 使用 profiles 来区分不同的环境（实际项目中可能分为开发环境、测试环境、生产环境），不同环境配置属性等可能不一样

```
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <db-url>jdbc:oracle:thin:@localhost:1521:XX</db-url>
            <db-username>*</db-username>
            <db-password>*</db-password>
        </properties>
        <activation>
            <activeByDefault>false</activeByDefault>
            <jdk>1.5</jdk>
            <property>
                <name>mavenVersion</name>
                <value>2.0.3</value>
            </property>
            <file>
                <exists>${basedir}/file2.properties</exists>
                <missing>${basedir}/file1.properties</missing>
            </file>
        </activation>
    </profile>
    <provile>
        <id>test</id>
    </provile>
    <provile>
        <id>prd</id>
    </provile>
</profiles>

\> mvn -P dev package
```

* 使用 Maven
使用语法 `mvn [options] [<goal(s)>] [<phase(s)>]`

> 
> mvn clean
> 清空 target 目录
> mvn compile
> 编译项目
> mvn test
> 运行单元测试
> mvn test-compile
> 编译单元测试代码，而不运行
> mvn package
> 打包项目成 jar (or rar, ejb, war...)
> mvn install
> 把打包结果安装到本地仓库 ${user.home}/.m2/repository
> mvn site
> 生成项目 site 信息
> 

* 使用 Maven 插件

> archetype(原型)
> -----
> 可直接使用的 goals
> archetype:generate 创建一个 Maven 项目
> archetype:create-from-project 以已存在项目为模板创建新项目原型
> archetype:crawl
> 
> 绑定都默认生命周期的 goals
> archetype:jar  
> 绑定到 package 阶段(phase)，用户构建 jar
> archetype:integration-test
> 绑定到 integration-test 阶段，用于执行集成测试
> archetype:update-local-catalog
> 绑定到 install 阶段，用户更新本地目录
> 
> compiler
> -----
> compiler:compile 绑定到  compile 阶段，编译 main 源代码
> compiler:testCompile 绑定到 test-compile 阶段，编译 test 源代码
> 
> resources
> -----
> resources:resources
> 复制 main 源代码目录的资源文件到 main 输出目录，使之能打包到最终 jar 包。绑定到　process-resource 阶段，自动执行。
> resources:testResource
> 复制 test 源代码目录的资源文件到 test 输出目录。绑定到 process-test-resources　阶段，自动执行。 
> resources:copy-resources
> 赋值指定的资源文件到指定的输出目录
> 
> Surefire
> -----
> 在构建生命周期的 test 阶段用来运行单元测试。
> surefire:test
> 
> Failsafe
> -----
> failsafe:integration-test
> 运行集成测试
> failsafe:verify
> 验证项目的集成测试是否通过
> 
> Maven 的生命周期中有4个阶段运行集成测试：
> pre-integration-test
> integration-test
> post-integration-test
> verify
> Failsafe 插件在构建生命周期的 integration-test 和 verify 阶段用来运行集成测试。当构建过程中出现错误时，会继续执行 post-integration-test 阶段
> 一般直接执行如下命令
> \> mvn verify         //运行检查，验证包是否有效且达到质量标准。
> mvn integration-test  //运行集成测试。
> 
> jar
> -----
> 用来构建 jar包
> jar:jar
> jar:test-jar
> 
>  deploy
>  -----
>  在 deploy 阶段把 artifact 加入到远程仓库
>  deploy:deploy
>  deploy:deploy-file
>  
>  mvn deploy:deploy-file -DgroupId=com -DartifactId=client 
>  -Dversion=0.1.0 -Dpackaging=jar 
>  -Dfile=d:\client-0.1.0.jar 
>  -DrepositoryId=maven-repository-inner 
>  -Durl=ftp://xxxxxxx/opt/maven/repository/  
>  
>  
>  install
>  -----
>  用于在 install 阶段发布第三方Jar到本地库中
>  install:install
>  install:install-file
>  install:help
>  
>  mvn install:install-file -DgroupId=com -DartifactId=client 
>  -Dversion=0.1.0 
>  -Dpackaging=jar -Dfile=d:\client-0.1.0.jar 
>  -DdownloadSources=true 
>  -DdownloadJavadocs=true 

* maven 生命周期

maven 有3套独立的生命周期：
1. clean,   进行构建前的清理工作 
2. default, 构建的核心部分，包括编译、测试、打包部署等
3. site     生成项目报告、项目站点、发布站点

每个生命周期由一系列阶段构成。我们在命令行中输入的命令总会对应一个阶段。而运行某个阶段时，在生命周期中之前的阶段也会执行。
1. clean生命周期的阶段
> pre-clean     执行 clean 前需要完成的工作
> clean         清除上一次构建结果.
> post-clean    执行 clean 后需要立刻完成的工作
> 
> mvn clean 命令相当于 mvn pre-clean clean。会先执行 pre-clean

2. default生命周期的阶段

> validate    验证项目必要的信息可用
> initialize
> generate-sources
> process-sources
> generate-resources
> process-resources   复制并处理资源文件到目标目录，准备打包
> compile             编译源代码
> process-classes
> generate-test-sources
> process-test-sources
> generate-test-resources
> process-test-resources  复制并处理资源文件到测试目标目录，准备打包
> test-compile         编译测试代码
> process-test-classes
> test                 执行单元测试
> prepare-package
> package
> pre-intepration-test
> integration-test
> post-integration-test
> verify   检查验证包符合质量要求
> install  安装包到本地仓库
> deploy   在集成或发布环境下，发布最终包到远程仓库供他人使用

3. site生命周期的阶段

> pre-site    生成项目站点文档前需要完成的工作
> site        生成项目站点文档
> post-site   生成项目站点文档后需要完成的工作
> site-deploy 将生成的站点文档部署到服务器上


* 使用 Nexus 搭建私服

[Nexus](http://nexus.sonatype.org/downloads) 是一个 Maven 仓库管理器，可用于组织内部管理 Maven 仓库（避免每次从 Maven 中心仓库下载包）。

nexus 是一个 war 应用。下载后部署到 jetty 或 tomcat 等 Java 应用服务器即可。默认的管理员密码 admin/admin123。
