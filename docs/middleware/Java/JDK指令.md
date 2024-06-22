### JDK 设置

- **安装**

```sh
[root@localhost ~]# mkdir -pv /usr/java/
mkdir: created directory `/usr/java/'
[root@localhost ~]# 
[root@localhost ~]# tar -zxvf ~/upload/jdk-8u211-linux-x64.tar.gz -C /usr/java/
[root@localhost ~]# 
[root@localhost ~]# ln -snf jdk1.8.0_211/ /usr/java/home
```

***/etc/profile (source /etc/profile)***

```properties
# set jdk profile
JAVA_HOME=/usr/java/home
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export PATH JAVA_HOME JRE_HOME CLASS_PATH
```

- **查看版本**

```sh
[root@localhost ~]# java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

***

### Web 服务

- **导出wsdl文档**

```
wsimport -d E:\x -s E:\x\src -p cn.yejh -encoding UTF-8 -keep -verbose http://<ip>[:<port>]/context/services/a?wsdl
```

***

### 证书

- **列出证书**

```sh
[root@localhost ~]# keytool -list [-alias yejh] -keystore $JAVA_HOME/jre/lib/security/cacerts [-storepass changeit]
Keystore type: jks
Keystore provider: SUN

Your keystore contains 95 entries

yejh, Jun 2, 2019, trustedCertEntry, 
Certificate fingerprint (SHA1): CA:06:F5:6B:25:8B:7A:0D:4F:2B:05:47:09:39:47:86:51:15:19:84
```

- **导入证书**

```sh
[root@localhost ~]# keytool -importcert -alias yejh -file ./DigiCert.cer -keystore $JAVA_HOME/jre/lib/security/cacerts [-storepass changeit]
Owner: CN=github.com, O="GitHub, Inc.", L=San Francisco, ST=California, C=US, SERIALNUMBER=5157550, OID.1.3.6.1.4.1.311.60.2.1.2=Delaware, OID.1.3.6.1.4.1.311.60.2.1.3=US, OID.2.5.4.15=Private Organization
Issuer: CN=DigiCert SHA2 Extended Validation Server CA, OU=www.digicert.com, O=DigiCert Inc, C=US
Serial number: a0630427f5bbced6957396593b6451f
Valid from: Tue May 08 08:00:00 CST 2018 until: Wed Jun 03 20:00:00 CST 2020
Certificate fingerprints:
         MD5:  9C:81:F7:D7:AD:12:6A:83:21:7B:35:AC:5D:C7:AA:02
         SHA1: CA:06:F5:6B:25:8B:7A:0D:4F:2B:05:47:09:39:47:86:51:15:19:84
         SHA256: 31:11:50:0C:4A:66:01:2C:DA:E3:33:EC:3F:CA:1C:9D:DE:45:C9:54:44:0E:7E:E4:13:71:6B:FF:36:63:C0:74
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

...

Trust this certificate? [no]:  yes
Certificate was added to keystore
```

- **删除证书**

```sh
[root@localhost ~]# keytool -delete -alias yejh -keystore $JAVA_HOME/jre/lib/security/cacerts [-storepass changeit]
```

- **制作 Java KeyStore 文件**

1. 生成 ***crt*** 证书和 ***key*** 私钥

```sh
[root@vmx ~]# openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /usr/nginx/nginx.key -out /usr/nginx/nginx.crt
Generating a RSA private key
............................................+++++
.......+++++
writing new private key to '/usr/nginx/nginx.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:GD
Locality Name (eg, city) [Default City]:SZ
Organization Name (eg, company) [Default Company Ltd]:yejh
Organizational Unit Name (eg, section) []:ssl.yejh
Common Name (eg, your name or your server's hostname) []:yejh.com
Email Address []:yejh.1248@qq.com
```

2. 合并成 ***p12*** 文件，填入 ```server.ssl.key-password``` 值

```sh
[root@vmx ~]# openssl pkcs12 -export -in /usr/nginx/nginx.crt -inkey /usr/nginx/nginx.key -out /usr/nginx/nginx.p12
Enter Export Password:
Verifying - Enter Export Password:
```

3. 转换成 Java 生态识别的 ***jks***
   文件，```server.ssl.key-password=ssl_kp```，```server.ssl.key-store-password=ssl_ksp```

```sh
[root@vmx ~]# keytool -importkeystore -v -srckeystore /usr/nginx/nginx.p12 -srcstoretype pkcs12 -srcstorepass ssl_kp -destkeystore /usr/java/java.jks -deststoretype jks -deststorepass ssl_ksp
Importing keystore /usr/nginx/nginx.p12 to /usr/java/java.jks...
Entry for alias 1 successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled
[Storing /usr/java/java.jks]

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /usr/java/java.jks -destkeystore /usr/java/java.jks -deststoretype pkcs12".
```

***

### 内存分析

- **jps**

```sh
[root@vm1 ~]# jps -ml
2528 org.apache.rocketmq.namesrv.NamesrvStartup
2546 org.apache.rocketmq.broker.BrokerStartup -c /usr/rocketmq/home/conf/2m-2s-async/broker-a.properties -n localhost:9876
9908 sun.tools.jps.Jps -ml
2586 org.apache.rocketmq.broker.BrokerStartup -c /usr/rocketmq/home/conf/2m-2s-async/broker-b-s.properties -n localhost:9876
2671 rocketmq-console-ng-2.0.0.jar --server.port=6789 --rocketmq.config.namesrvAddr=localhost:9876
```

- **jinfo**

```sh
[root@vm1 ~]# jinfo <pid>
Attaching to process ID 2671, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.271-b09
```

- **jmap**

```sh
[root@vm1 ~]# jmap -dump:format=b,file=./dump.hprof <pid>
Dumping heap to /root/dump.hprof ...
Heap dump file created
[root@vm1 ~]# 
[root@localhost ~]# jmap -histo:live <pid>

 num     #instances         #bytes  class name
----------------------------------------------
   1:         41722        6996816  [C
   2:         40850         980400  java.lang.String
```

- **jstat**

```sh
[root@localhost ~]# jstat -gcutil <pid> 1s 2
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00   0.00   1.81  35.67  93.90  90.95     29    0.124    12    0.589    0.713
  0.00   0.00   1.81  35.67  93.90  90.95     29    0.124    12    0.589    0.713
```

- **jstack**

```sh
[root@localhost ~]# jstack [-m] -l <pid>
2019-06-02 01:11:05
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.211-b12 mixed mode):

"Attach Listener" #31 daemon prio=9 os_prio=0 tid=0x00007f28fc001800 nid=0x3f74 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"DestroyJavaVM" #30 prio=5 os_prio=0 tid=0x00007f2928009000 nid=0x3f30 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

```

***

### 参考[官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html)