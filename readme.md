### 参照:https://www.jianshu.com/p/98a141701cc7

# 第一阶段 ：配置github

1、创建mvn-repo分支    
首先在你的github上创建一个maven-repo-demo仓库，这个最后将作为实际上jar包发布的仓库

2、设置真实姓名    
在github的个人设置中，设置好自己的姓名 。这个环节很重要，若不设置姓名，会出现一些一些意想不到的错误，如：
```
    [ERROR] Failed to execute goal com.github.github:site-maven-plugin:0.12:site (default) on project rfcore: Error creating commit: Invalid request.
    [ERROR] For 'properties/name', nil is not a string.
    [ERROR] For 'properties/name', nil is not a string. (422)
    [ERROR] -> [Help 1]
```

# 第二阶段：配置部署jar到本地1、 配置本地mvn服务    

1、设置本地maven的配置文件settings.xml,找到其中的servers 标签，加入如下 配置：
```
        <server>
            <id>github</id>
            <username>*****</username>
            <password>*****</password>
        </server>
```

2、修改pom文件发布本地仓库    
在需要发布的项目中的pom文件里的 标签下加入以下插件：

然后运行 mvn clean deploy 命令，即可在对应项目中的target/repository目录下找到本地的jar
```
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.1</version>
                    <configuration>
                        <altDeploymentRepository>internal.repo::default::file://${project.build.directory}/repository</altDeploymentRepository>
                    </configuration>
                </plugin>
            </plugins>
        </build>
```
# 第三阶段：发布jar到远程github上
1、修改pom文件，在properties 中添加下列属性
```
        <properties>
            <github.global.server>github</github.global.server>
        </properties>
```

2、添加修改插件
```
        <build>
            <plugins>
                <!--打包插件-->
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.1</version>
                    <configuration>
                        <altDeploymentRepository>internal.repo::default::file://${project.build.directory}/repository</altDeploymentRepository>
                    </configuration>
                </plugin>
    
                <!--源码-->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>2.2.1</version>
                    <executions>
                        <execution>
                            <id>attach-sources</id>
                            <goals>
                                <goal>jar-no-fork</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
    
                <!--github上传插件,用于修改后的发布,执行 mvn clean deploy 自动打包上传到github-->
                <plugin>
                    <groupId>com.github.github</groupId>
                    <artifactId>site-maven-plugin</artifactId>
                    <version >0.12</version>
                    <configuration>
                        <message >Maven artifacts for ${project.version}</message>
                        <noJekyll>true</noJekyll>
                        <!--本地jar地址-->
                        <outputDirectory>${project.build.directory}/repository</outputDirectory>
                        <!--分支-->
                        <branch>refs/heads/master</branch>
                        <merge>true</merge>
    
                        <includes>
                            <include>**/*</include>
                        </includes>
                        <!--对应github上创建的仓库名称 name-->
                        <repositoryName>maven-repo-demo</repositoryName>
                        <!--github 仓库所有者 不是账号-->
                        <repositoryOwner>l305170891</repositoryOwner>
                    </configuration>
    
                    <executions>
                        <execution>
                            <goals>
                                <goal>site</goal>
                            </goals>
                            <phase>deploy</phase>
                        </execution>
                    </executions>
                </plugin>
    
    
            </plugins>
        </build>
```

再次执行 mvn clean deploy命令即可发布到github上了 。
若出现如下错误请完成第一阶段第二步配置：
```
    [ERROR] Failed to execute goal com.github.github:site-maven-plugin:0.12:site (default) on project rfcore: Error creating commit: Invalid request.
    [ERROR] For 'properties/name', nil is not a string.
    [ERROR] For 'properties/name', nil is not a string. (422)
    [ERROR] -> [Help 1]
```

# 第四阶段：在项目中使用发布到github上的jar包pom文件中添加github仓库
然后添加依赖即可
```
        <dependency>
    			<groupId>com.luojian.github.repo.demo</groupId>
    			<artifactId>maven-repo-demo</artifactId>
    			<version>1.0.1</version>
        </dependency>
```

