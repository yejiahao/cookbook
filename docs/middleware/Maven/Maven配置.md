### 安装 Maven

```sh
[root@localhost ~]# mkdir -pv /usr/maven/
mkdir: created directory `/usr/maven/'
[root@localhost ~]# 
[root@localhost ~]# tar -zxvf ~/upload/apache-maven-3.8.6-bin.tar.gz -C /usr/maven/
[root@localhost ~]# 
[root@localhost ~]# ln -snf apache-maven-3.8.6/ /usr/maven/home
```

***/etc/profile (source /etc/profile)***

```properties
# set maven profile
MAVEN_HOME=/usr/maven/home
PATH=$PATH:$MAVEN_HOME/bin
export PATH MAVEN_HOME
```

### 查看版本

```sh
[root@vmx ~]# mvn -v
Apache Maven 3.8.6 (84538c9988a25aec085021c365c560670ad80f63)
Maven home: /usr/maven/home
Java version: 1.8.0_351, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_351/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.18.0-348.7.1.el8_5.x86_64", arch: "amd64", family: "unix"
```

### 配置阿里云镜像中央仓库

***$MAVEN_HOME/conf/settings.xml***

```xml

<localRepository>/usr/maven/Maven_Repository</localRepository>

<mirrors>
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/repository/central</url>
</mirror>
</mirrors>
```

### 安装 jar 包到本地仓库

- [参考文档](http://maven.apache.org/components/plugins/maven-install-plugin/install-file-mojo.html)

```sh
[root@localhost ~]# mvn install:install-file -Dfile="ojdbc14-10.2.0.1.0.jar" -DgroupId="com.oracle" -DartifactId="ojdbc14" -Dversion="10.2.0.1.0" -Dpackaging="jar" -DpomFile="ojdbc14-10.2.0.1.0.pom"
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------< org.apache.maven:standalone-pom >-------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] --------------------------------[ pom ]---------------------------------
[INFO] 
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
[INFO] Installing /root/ojdbc14-10.2.0.1.0.jar to /usr/maven/home/Maven_Repository/com/oracle/ojdbc14/10.2.0.1.0/ojdbc14-10.2.0.1.0.jar
[INFO] Installing /tmp/mvninstall6167534367552105801.pom to /usr/maven/home/Maven_Repository/com/oracle/ojdbc14/10.2.0.1.0/ojdbc14-10.2.0.1.0.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.822 s
[INFO] Finished at: 2019-06-02T21:17:55+08:00
[INFO] ------------------------------------------------------------------------
```

### 自动建立目录骨架

```sh
[root@localhost ~]# mvn archetype:generate
```

### 打包跳过测试阶段

- **不执行测试用例，也不编译测试用例类**

```sh
[root@localhost yejh-api]# mvn clean package -Dmaven.test.skip=true
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ yejh-api ---
[INFO] Not copying test resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ yejh-api ---
[INFO] Not compiling test sources
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ yejh-api ---
[INFO] Tests are skipped.
```

- **不执行测试用例，但编译测试用例类生成相应的 class 文件至 target/test-classes 下**

```sh
[root@localhost yejh-api]# mvn clean package -DskipTests
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ yejh-api ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /root/yejh-api/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ yejh-api ---
[INFO] No sources to compile
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ yejh-api ---
[INFO] Tests are skipped.
```