[TOC]

# ProtoBuf

- 编码
- 序列化
- 反序列化
- 反射

## 使用

1. 创建`.proto`文件
2. 定义数据结构

```proto
message ${YourDataStructure} {
  // 字段规则：required -> 字段只能也必须出现 1 次
  // 字段规则：optional -> 字段可出现 0 次或1次
  // 字段规则：repeated -> 字段可出现任意多次（包括 0）
  // 类型：int32、int64、sint32、sint64、string、32-bit ....
  // 字段编号：0 ~ 536870911（除去 19000 到 19999 之间的数字）
  字段规则 类型 名称 = 字段编号;
}
```

3. 编译

```
// $SRC_DIR: .proto 所在的源目录
// --cpp_out: 生成 c++ 代码
// $DST_DIR: 生成代码的目标目录
// xxx.proto: 要针对哪个 proto 文件生成接口代码

protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/xxx.proto
```

e.g.

```proto
// test.proto

syntax = "proto3";

message Test {
  optional string query = 1;
  int32 number = 2;
}

// 命令行编译
protoc.exe -I=./ --java_out=./ ./test.proto
```

## 链接

- https://github.com/protocolbuffers/protobuf
- https://developers.google.com/protocol-buffers/docs/tutorials
- https://developers.google.com/protocol-buffers/
