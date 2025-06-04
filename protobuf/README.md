# 1 protobuf 概念

*.proto 文件是 Protocol Buffers（简称 Protobuf）的核心定义文件，用于**定义结构化数据的模式**, 以便通过protobuf进行**序列化和反序列化**。它类似于 XML Schema 或 JSON Schema，但更高效、更简洁，特别适合高性能数据交换和 RPC 服务定义。<br>


# 2 基本文件结构

```proto
syntax = "proto3";  // 指定协议版本

package mypackage;  // 包名（防止命名冲突）

import "google/protobuf/timestamp.proto"; // 导入其他proto文件

// 选项配置
option java_package = "com.example.generated";
option optimize_for = SPEED;

// 消息类型定义
message Person {
  // 字段定义
  string name = 1;
  int32 id = 2;
  string email = 3;

  // 枚举类型
  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  // 嵌套消息
  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  // 重复字段
  repeated PhoneNumber phones = 4;

  // 时间戳（使用导入的类型）
  google.protobuf.Timestamp last_updated = 5;
}

// 服务定义（用于gRPC）
service PersonService {
  rpc GetPerson (PersonRequest) returns (Person) {}
  rpc CreatePerson (Person) returns (PersonResponse) {}
}
```

# 3 核心组件

## 3.1 消息类型(Message)

```protobuf
message Person {
  // 字段规则 数据类型 字段名 = 字段编号;
  required string name = 1;      // 必需字段 (proto3移除了required)
  optional int32 age = 2;        // 可选字段
  repeated string emails = 3;    // 重复字段（数组）

  // 字段编号要求：
  // 1-15: 占1字节 (常用字段)
  // 16-2047: 占2字节
  // 不能使用19000-19999 (保留范围)
}
```

## 3.2 数据类型

```markdown
| 类型      | 说明               | 对应语言类型                     |
|-----------|--------------------|----------------------------------|
| double    | 双精度浮点         | C++/Java double                 |
| float     | 单精度浮点         | C++/Java float                  |
| int32     | 32位整数           | C++/Java int                    |
| int64     | 64位整数           | C++/Java long                   |
| uint32    | 无符号32位         | C++ uint32_t                    |
| uint64    | 无符号64位         | C++ uint64_t                    |
| sint32    | 有符号32位（高效编码） |                                  |
| sint64    | 有符号64位（高效编码） |                                  |
| bool      | 布尔值             | bool                             |
| string    | UTF-8字符串        | C++ std::string                 |
| bytes     | 二进制数据         | C++ std::string                 |
| enum      | 枚举               |                                  |
| message   | 嵌套消息           |                                  |
```

## 3.3 枚举类型

```proto
enum PhoneType {
  // 必须从0开始
  MOBILE = 0;  // 默认值
  HOME = 1;
  WORK = 2;
  // 可以定义别名（相同数值）
  OFFICE = 2;  // 与WORK相同
}
```

## 3.4 服务定义(gRPC)

```proto
service PersonService {
  // 简单RPC
  rpc GetPerson (PersonRequest) returns (Person) {}

  // 服务端流式
  rpc ListPersons (Filter) returns (stream Person) {}

  // 客户端流式
  rpc RecordPersons (stream Person) returns (Summary) {}

  // 双向流式
  rpc Chat (stream Message) returns (stream Message) {}
}
```

## 3.5 常用选项
```proto
option optimize_for = SPEED;  // 优化目标：SPEED, CODE_SIZE, LITE_RUNTIME
option java_multiple_files = true;  // 生成多个Java文件
option go_package = "github.com/example/protos";  // Go包路径

// 字段选项
string name = 1 [deprecated = true];  // 标记字段弃用
int32 id = 2 [(custom_option) = "value"];  // 自定义选项
```

## 3.6 高级特性
- Oneof 字段

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    int32 id = 9;
  }
}
// 同一时间只能设置其中一个字段

// 使用场景：
// - 替代多个optional字段
// - 节省内存空间
```

- Map 类型

```proto
map<string, Project> projects = 3;
// 等价于：
message MapFieldEntry {
  string key = 1;
  Project value = 2;
}
repeated MapFieldEntry projects = 3;
```

- 保留字段
```proto
message Foo {
  reserved 2, 15, 9 to 11;  // 保留字段编号
  reserved "email", "phone"; // 保留字段名
}
```

- Any 类型
```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
// 可以包含任意类型的消息
```

- 时间戳和常用类型

```proto
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/empty.proto";

message Task {
  google.protobuf.Timestamp create_time = 1;
  google.protobuf.Duration duration = 2;
}
```

# 4 开发案例
.proto 文件作为 Protobuf 的核心，通过简洁的语法提供了强大的数据结构定义能力。结合 gRPC 可以构建高性能的跨语言服务，是现代分布式系统和微服务架构的基础设施之一。

## 4.1 安装依赖

```sh
pip install grpcio grpcio-tools protobuf
```

## 4.2 person.proto 文件
```proto
syntax = "proto3";

package example;

// 定义服务接口
service HelloService {
  // 单向请求响应模式
  rpc SendMessage (HelloRequest) returns (HelloResponse) {}
}

// 请求消息类型
message HelloRequest {
  string name = 1;
}

// 响应消息类型
message HelloResponse {
  string message = 1;
}
```

## 4.2 编译生成代码

```bash
# 生成python代码
python -m grpc_tools.protoc  -I. --python_out=. --grpc_python_out=. hello.proto

# 生成两个文件
# - `hello_pb2.py` ：数据序列化类
# - `hello_pb2_grpc.py` ：gRPC服务接口类

# 也可程序Java/Go/C++ 等类型代码
```
## 4.3 服务端实现
```python
import grpc
import hello_pb2
import hello_pb2_grpc
from concurrent import futures

class HelloServiceServicer(hello_pb2_grpc.HelloServiceServicer):
    def SendMessage(self, request, context):
        response = hello_pb2.HelloResponse()
        response.message  = f"Hello, {request.name}!"
        return response

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    hello_pb2_grpc.add_HelloServiceServicer_to_server(
        HelloServiceServicer(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    print("Server started on port 50051")
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```

## 4.4 客户端调用
```python
import grpc
import hello_pb2
import hello_pb2_grpc

def run():
    channel = grpc.insecure_channel('localhost:50051')
    stub = hello_pb2_grpc.HelloServiceStub(channel)

    request = hello_pb2.HelloRequest(name="World")
    response = stub.SendMessage(request)

    print("Received:", response.message)

if __name__ == '__main__':
    run()
```

```python
# Client 端 和 Server 端调用流程
# Client                     Server
#    │                           │
#    ├─────── SendMessage ──────►
#    │       (HelloRequest)      │
#    │                           │
#    ◄─────── Response ──────────┤
#    │       (HelloResponse)     │
```

## 4.5 运行服务端和客户端
```bash
python server.py

# other terminal
python client.py

# 输出：Received: Hello, World!
```