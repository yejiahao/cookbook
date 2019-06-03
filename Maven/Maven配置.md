### 安装 Maven
```
[root@localhost ~]# mkdir -pv /usr/maven/
mkdir: created directory `/usr/maven/'
[root@localhost ~]# 
[root@localhost ~]# tar -zxvf ~/upload/apache-maven-3.6.1-bin.tar.gz -C /usr/maven/
[root@localhost ~]# 
[root@localhost ~]# ln -snf apache-maven-3.6.1/ /usr/maven/home
```
***/etc/profile (source /etc/profile)***
```properties
# set maven profile
MAVEN_HOME=/usr/maven/home
PATH=$PATH:$MAVEN_HOME/bin
export PATH MAVEN_HOME
```

### 查看版本
```
[root@localhost ~]# mvn -v
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /usr/maven/home
Java version: 1.8.0_211, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_211/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-754.el6.x86_64", arch: "amd64", family: "unix"
```

### 配置阿里云镜像中央仓库
***$MAVEN_HOME/conf/settings.xml***
```xml
<localRepository>/usr/maven/home/Maven_Repository</localRepository>

<mirrors>
  <mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
  </mirror>
</mirrors>
```

### 安装 jar 包到本地仓库
```
[root@localhost ~]# mvn install:install-file -Dfile="ojdbc14-10.2.0.1.0.jar" -DgroupId="com.oracle" -DartifactId="ojdbc14" -Dversion="10.2.0.1.0" -Dpackaging="jar"
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
```
[root@localhost ~]# mvn archetype:generate
```

### 打包跳过测试阶段
- **不执行测试用例，也不编译测试用例类**
```
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
```
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