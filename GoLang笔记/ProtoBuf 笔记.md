# ProtoBuf相关笔记

[TOC]

## 一、PRotoBuf简介

Google Protocol Buffer(简称 Protobuf)是一种轻便高效的结构化数据存储格式，平台无关、语言无关、可扩展，
可用于通讯协议和数据存储等领域。

## 二、ProtoBuf的优势

> 1：序列化后体积相比Json和XML很小，适合网络传输
> 2：支持跨平台多语言
> 3：消息格式升级和兼容性还不错
> 4：序列化反序列化速度很快，快于Json的处理速速

## 三、ProtoBuf的优缺点

### 1、ProtoBuf的优点

> Protobuf 有如 XML，不过它更小、更快、也更简单。可以定义自己的数据结构，然后使用代码生成器生成的代码来读写这个数据结构。甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。
>
> 它有一个非常棒的特性，即“向后”兼容性好，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对数据结构进行升级。
>
> Protobuf 语义更清晰，无需类似 XML 解析器的东西（因为 Protobuf 编译器会将 .proto 文件编译生成对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作）。使用 Protobuf 无需学习复杂的文档对象模型，Protobuf 的编程模式比较友好，简单易学，同时它拥有良好的文档和示例。

### 2、ProtoBuf的缺点

> Protobuf 与 XML 相比也有不足之处。它功能简单，无法用来表示复杂的概念。
>
> Protobuf 不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 Protobuf 不行，它以二进制的方式存储，除非你有 .proto 定义，否则你没法直接读出 Protobuf 的任何内容。

## 四、ProtoBuf的安装

```shell
# 下载 protoBuf：
$ git clone https://github.com/protocolbuffers/protobuf.git

# 或者直接将压缩包拖入后解压
$ unzip protobuf.zip

#安装依赖库
$ sudo apt-get install autoconf automake libtool curl make g++ unzip libffidev
-y

# 进入到clone或者解压后的protobuf目录安装
$ cd protobuf/
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
# 刷新共享库 很重要的一步啊
$ sudo ldconfig 

# 安装的时候会比较卡
# 成功后需要使用命令测试
$ protoc –h

```

## 五、ProtoBuf数据类型



| .protoType | Notes                                                        | C++<br/>Type | Python<br/>Type | Go<br/>Type |
| :--------: | :----------------------------------------------------------- | :----------: | :-------------: | :---------: |
|   double   |                                                              |    double    |      float      |   float64   |
|   float    |                                                              |    float     |      float      |   float32   |
|   int32    | 使用变长编码，对于负值的效率很低，如果你的域有可能有负值，请使用sint64替代 |    int32     |       int       |    int32    |
|   uint32   | 使用变长编码                                                 |    uint32    |    int/long     |   uint32    |
|   uint64   | 使用变长编码                                                 |    uint64    |    ing/long     |   uint64    |
|   sint32   | 使用变长编码，这些编码在负值时比int32高效的多                |    int32     |       int       |    int32    |
|   sint64   | 使用变长编码，有符号的整型值。编码时比通常的int64高效。      |    int64     |    int/long     |    int64    |
|  fixed32   | 总是4个字节，如果数值总是比总是比228大的话，这个类型会比uint32高效。 |    uint32    |       int       |   uint32    |
|  fixed64   | 总是8个字节，如果数值总是比总是比256大的话，这个类型会比uint64高效。 |    uint64    |    int/long     |   uint64    |
|  sfixed32  | 总是4个字节                                                  |    int32     |       int       |    int32    |
|  sfixed64  | 总是8个字节                                                  |    int64     |    int/long     |    int64    |
|    bool    | bool                                                         |     bool     |      bool       |    bool     |
|   string   | 一个字符串必须是UTF-8编码或者7-bit ASCII编码的文本。         |    string    |   str/unicode   |   string    |
|   bytes    | 可能包含任意顺序的字节数据                                   |    string    |       str       |   []byte    |

注意：<font color=red>如果是传输文件，全部用bytes二进制类型。</font>

<font color=red>**Repeated 关键字表示数组的，那么在go语言中用切片进行代表 正如上述文件格式，在消息定义中，每个字段都有唯一的一个标识符。**</font>

## 六、获取GoLang第三方包

### 1、获取proto包

```shell
# Go语言的proto API接口
$ go get -v -u github.com/golang/protobuf/proto
```

### 2、安装protoc-gen-go插件

它是一个 go程序，编译它之后将可执行文件复制到\bin目录。

```shell
# 获取安装
$ go get -v -u github.com/golang/protobuf/protoc-gen-go
# 编译
$ cd $GOPATH/src/github.com/golang/protobuf/protoc-gen-go/
$ go build
# 将生成的 protoc-gen-go可执行文件
$ sudo cp protoc-gen-go /bin/
```

## 七、ProtoBuf语法和编译

### 1、ProtoBuf语法

```shell
# 新建protobuf文件
$ vim test.proto
```

文件内容

```protobuf
syntax = "proto3";

message PandaRequest {
    string name = 1;
    int32 shengao = 2;
    // 数组，对应golang里的是切片
    repeated int32 tizhong = 3;
}
```

### 2、编译ProtoBuf文件

```shell
$ protoc --go_out=./ *.proto
```

## 八、ProtoBuf示例

> 编写完ProtoBuf文件后，要编译生成相应的go文件

### 1、示例1

#### 1.1、ProtoBuf文件

```protobuf
syntax = "proto3";

message PandaRequest {
    string name = 1;
    int32 shengao = 2;
    // 数组，对应golang里的是切片
    repeated int32 tizhong = 3;
}
```

#### 1.2、GoLang示例文件

```go
func main()  {

    test1 := &test.PandaRequest{
        Name:"panda",
        Shengao:188,
        Tizhong: []int32{124,553,116},
    }

    data, err := proto.Marshal(test1)
    if err != nil {
        fmt.Println("转码失败 = ",err)
    }

    fmt.Println("data = ",data)


    newTest := &test.PandaRequest{}

    err = proto.Unmarshal(data, newTest)

    if err != nil {
        fmt.Println("解码失败 = ",err)
    }

    fmt.Println("new test un data = ",newTest.String())

}
```

### 2、示例2

#### 2.1、ProtoBuf文件

> 定义一个消息类型当作消息字段

```protobuf
syntax = "proto3";

message PersonInfo {
    repeated Person info = 1;
}

message Person {
    string name = 1;
    int32 shengao = 2;
    repeated int32 tizhong = 3;
}
```

#### 2.2、GoLang示例文件

```go
func main()  {

    test := &test2.PersonInfo{
        Info: []*test2.Person{
            {Name:"name1",Shengao:166,Tizhong:[]int32{124,553,116}},
            {Name:"name2",Shengao:177,Tizhong:[]int32{421,355,611}}},

    }

    fmt.Println("test1 = ",test)

    data, err := proto.Marshal(test)

    if err != nil {
        fmt.Println("转码失败 = ",err)
    }

    fmt.Println("data = ",data)

    newTest := &test2.PersonInfo{}

    err = proto.Unmarshal(data, newTest)

    if err != nil {
        fmt.Println("解码失败 = ",err)

    }

    fmt.Println("data un = ",newTest)

    for _,v := range newTest.Info {
        fmt.Println("v.name = ",v.Name)
        fmt.Println("v = ",v)
    }

}
```

### 3、示例3

#### 3.1、ProtoBuf文件

> 嵌套消息类型，这种使用比较少。<font color=red>使用消息类型作为消息字段，必需定义一个变量，否则外部使用赋值难操作</font>

```protobuf
syntax = "proto3";

message PersonInfo {
    message Person {
        string name = 1;
        int32 shengao = 2;
        repeated int32 tizhong = 3;
    }
    repeated Person info = 1;
}
```

#### 3.2、GoLang示例文件

```go
func main() {

    test1 := &test.PersonInfo{
       Info: []*test.PersonInfo_Person{
           {Name: "person name1", Shengao: 156, Tizhong: []int32{122, 113, 101}},
           {Name: "person name2", Shengao: 156, Tizhong: []int32{122, 113, 101}}},
    }

    fmt.Println(test1)

    data, err := proto.Marshal(test1)

    if err != nil {
       fmt.Println("err = ", err)
    }
    fmt.Println("data = ", data)

    newTest := &test.PersonInfo{}

    err = proto.Unmarshal(data, newTest)

    if err != nil{
     fmt.Println("解码失败 = ",err)
    }

    fmt.Println(newTest)

}
```











