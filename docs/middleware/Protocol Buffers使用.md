## 简介

[Protocol Buffers](https://protobuf.dev/) are language-neutral, platform-neutral extensible mechanisms for serializing
structured data.

## 安装客户端

| 环境变量       | 值                                   |
|------------|-------------------------------------|
| PROTO_HOME | F:\Tools\Protobuf\protoc-24.3-win64 |
| Path       | ...;%PROTO_HOME%\bin                |

```sh
$ protoc --version
libprotoc 24.3
```

## 生成源代码

### 1. 编辑 ***.proto*** 文本文件

***./protocol/YIMRequestProto.proto***

```protobuf
syntax = "proto3";

package com.yejh.yim.common.protocol;

option java_outer_classname = "YIMRequestProto"; // 生成的 wrapper 类名，优先级 > .proto 文件名
option java_package = "com.yejh.yim.common.protocol"; // 生成的包名，优先级 > package 关键字

// 实体类名
message YIMRequestBO {
  int64 requestId = 1;
  string requestMsg = 2;
  int32 type = 3;
}
```

### 2. 执行命令（以 Java 语言为例）

```sh
protoc --java_out=./yim-common/src/main/java/ ./protocol/YIMRequestProto.proto
或者
protoc --java_out=./yim-common/src/main/java/ ./protocol/*
```

## 测试代码

```java
package com.yejh.yim.common.protocol;

public static void main(String[] args) throws InvalidProtocolBufferException {
    YIMRequestProto.YIMRequestBO requestBO = YIMRequestProto.YIMRequestBO.newBuilder()
            .setRequestId(1L)
            .setRequestMsg("中文English123")
            .setType(3)
            .build();

    byte[] encode = encode(requestBO);

    YIMRequestProto.YIMRequestBO decode = decode(encode);

    log.info("{}", requestBO);
    log.info("{}", Objects.equals(requestBO, decode));
}

public static byte[] encode(YIMRequestProto.YIMRequestBO requestBO) {
    return requestBO.toByteArray();
}

public static YIMRequestProto.YIMRequestBO decode(byte[] bytes) throws InvalidProtocolBufferException {
    return YIMRequestProto.YIMRequestBO.parseFrom(bytes);
}
```