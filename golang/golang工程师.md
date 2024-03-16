# 1、HTTP 协议 RFC7231（掌握）

RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content 

预期：

## 1.1.了解 HTTP 协议格式，常见请求头、响应头、响应码

## 1.2.掌握 RFC 文档检索技巧，在未来工作中能根据文档持续学习 HTTP，并解决相关问题

# 2、Gin 开发开发（掌握）

官方文档：文档 | Gin Web Framework

项目源码：GitHub - gin-gonic/gin: Gin is a HTTP web framework written in Go (Golang). It 

features a Martini-like API with much better performance -- up to 40 times faster. If you need 
smashing performance, get yourself some Gin.
项目示例：GitHub - eddycjy/go-gin-example: An example of gin
预期：

## 2.1、能使用 Gin 框架进行业务开发

## 2.2、gin 框架的中间件机制及使用场景

在Gin的整个实现中，中间件可谓是Gin的精髓。一个个中间件组成一条中间件链，对HTTP Request请求进行拦截处理，实现了代码的解耦和分离，并且中间件之间相互不用感知到，每个中间件只需要处理自己需要处理的事情即可。

### Gin默认中间件

在Gin中，我们可以通过Gin提供的默认函数，来构建一个自带默认中间件的*Engine。

```golang
r := gin.Default()
```

Default函数会默认绑定两个已经准备好的中间件，它们就是Logger 和 Recovery，帮助我们打印日志输出和painc处理。

```golang

func Default() *Engine {
    debugPrintWARNINGDefault()
    engine := New()
    engine.Use(Logger(), Recovery())
    return engine
}
```

Gin的中间件是通过Use方法设置的，它接收一个可变参数，所以我们同时可以设置多个中间件。

```golang
func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes
```

### 自定义中间件

```golang
func costTime() gin.HandlerFunc {
    return func(c *gin.Context) {
        //请求前获取当前时间
        nowTime := time.Now()
 
        //请求处理
        c.Next()
 
        //处理后获取消耗时间
        costTime := time.Since(nowTime)
        url := c.Request.URL.String()
        fmt.Printf("the request URL %s cost %v\n", url, costTime)
    }
}
```

要注意的是c.Next方法，这个是执行后续中间件请求处理的意思（含没有执行的中间件和我们定义的GET方法处理），这样我们才能获取执行的耗时。也就是在c.Next方法前后分别记录时间，就可以得出耗时。

有了自定义的中间件，我们就可以这么使用。

```golang
func main() {
    r := gin.New()
 
    r.Use(costTime())
 
    r.GET("/", func(c *gin.Context) {
        c.JSON(200, "首页")
    })
    r.Run(":8080")
}
```

### 局部中间件

局部中间件意味着部分接口才会生效，只在局部使用，这时候访问

```
 r.GET("/", MiddleWare(), func(c *gin.Context) {
        // 取值
        req, _ := c.Get("request")
        fmt.Println("request:", req)
        // 页面接收
        c.JSON(200, gin.H{"request": req})
    })
```

<br/>

## 2.3、panic 的概念和处理，应对应用程序中的错误

panic是go的内置函数，它可以终止程序的正常执行流程并发出panic。比如当函数F调用panic，F的执行将被终止，并返回到调用者。该过程一直跟随堆栈向上，直到当前goroutine中的所有函数都返回，此时程序崩溃。panic可以通过直接调用panic产生。同时也可能由运行时的错误所产生，例如数组越界访问。

recover是go语言的内置函数，它的唯一作用是可以从panic中重新控制goroutine的执行。recover必须通过defer来运行。在正常的执行流程中，调用recover将会返回nil且没有什么其他的影响。但是如果当前的goroutine产生了panic，recover将会捕获到panic抛出的信息，同时恢复其正常的执行流程。

### panic从哪儿来

- 空指针 invalid memory address or nil pointer dereference
- 数组越界 index out of range；slice bounds out of range
- 除数为零 integer divide by zero
- 自定义panic
  
  <br/>

###  panic到哪儿去

defer语句将函数调用保存到一个列表上。保存的调用列表在当前函数返回前执行。Defer通常用于简化执行各种清理操作的函数。

通俗地说，就是defer保证函数调用不管在什么情况下（即使当前函数发生panic），在当前函数返回前必然执行。另外defer的函数调用符合**先进后出**的规则，即先defer的函数后执行。

```golang
defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
```

panic处理过程中会检测是否有defer的函数调用
如果有，按照先进后出的顺序依次执行
如果defer中有recover调用，则将调用栈修改到deferreturn，使得程序正常执行
否则当defer的函数调用执行完后，打印panic信息，进程退出

# 3、GRPC（掌握）

学习地址：Guides | gRPC
预期：

## 3.1、通信模式理解以及实践经验

### GRPC 四种通信模式‍

#### 普通模式（unary RPC）

假如我们需要构建一个订单管理系统，这个系统为用户提供了订单查询的接口，每次用户输入订单号，便会返回对应的订单信息。每个请求独立，且响应和请求一一对应，这就是简单的 RPC 模式，对于大多数业务场景均可以适用。

![截图](3ba6763918d1d219407ce9c832702ef8.png)

#### 服务端流模式（server-streaming RPC）

假如我们要创建一个订单的信息的缓存池，实现订单的高效查询，缓存服务启动的时候就需要请求服务端，同步服务端的订单信息。这种情况下，我们只需要请求一次，服务端便会持续的为我们提供同步订单信息，这就是服务端流 RPC 的场景。

和简单的 RPC 模式不同，服务端流 RPC 模式，客户端发起一个 RPC 请求，服务端会返回一系列的响应结果，发送完所有的响应后，服务端在流结尾标记服务状态详情作为结束的元数据。

![截图](9b402ee18035897f9ca6ce864df0f2f3.png)

#### 客户端流模式（client-streaming RPC）

假设以下场景，有一个服务每天需要定时和订单管理系统更新订单的最终状态，服务端只需要在最后告诉该服务，最终的处理结果（哪些更新成功，哪些失败）。不需要频繁和服务端建立和断开连接。从而可以降低服务端的并发节省连接资源。

与服务端流模式类似，客户端流会发送连续的更新请求给服务端，服务端在收到请求后不会立马给到客户端响应结果，直到请求结束了，才会返回一个单独的响应。

![截图](a377d37bfb803a465f526554e58ebcb1.png)

#### 双向流模式（bidirectional-streaming RPC）

同样以上面的订单更新的服务为例子，每天定时向管理系统推送一批订单进行更新，假如这个批订单有上千万的量，而且订单更新服务之后还需要和其他系统进行数据同步，这个时候使用客户端流模式显然不是那么合适，我们可以采用双向流模式，不断地推送订单请求给管理系统，服务端在收到请求，处理完立马返回处理结果给客户端，直到所有的请求处理结束。这种场景就是双向流模式。

![截图](0c24ef007681d22e1cd98ea665f6fc58.png)

<br/>

## 3.2、RPC 的通信协议、序列化

### 协议

#### 协议的作用

我们知道 RPC 需要将对象序列化成二进制数据，写入本地 Socket 中，然后被网卡发送到网络设备中进行网络传输。但是在传输过程中，RPC 并不会把请求参数的所有二进制数据整体一下子发送到对端机器上，中间可能会拆分成好几个数据包，也可能会合并其他请求的数据包（同一个 TCP 连接上的数据），至于怎么拆分合并，这其中的细节会涉及到系统参数配置和 TCP 窗口大小。对于服务提供方来说，他会从 TCP 通道里面收到很多的二进制数据，那这时候怎么识别出哪些二进制是第一个请求的呢？

所以我们需要对 RPC 传输数据的时候进行“断句”，在应用发送请求的数据包里面加入“句号”，这样接收方应用数据流里面分割出正确的数据，“句号”就相当于是消息的边界，用于标识请求数据的结束位置。于是需要在发送请求的时候定一个边界，在请求收到的时候按照这个设定的边界进行数据分割，避免语义不一致的事情发生，而这个边界语义的表达，就是所谓的协议。

#### 协议的设计

相对于 HTTP 的用处，RPC 给更多的是负责应用间的通信，所以性能要求更高。但是 HTTP 协议的数据包大小相对于请求数据本身要大很多，有需要加入很多无用的内容，比如换行符号、回车符等等；还有一个重要的原因就是 HTTP 协议属于无状态协议，客户端无法对请求和响应进行关联，每次请求都需要重新建立连接，响应完之后再关闭连接。所以对于要求高性能的 RPC 来说，HTTP 协议很难满足需求，RPC 会选择设计更紧凑的私有协议。

上面我们说了需要对传输的数据进行“断句”来确定消息边界，由于 RPC 每次发请求的大小都是不固定的，所以我们的协议必须能让接收方正确的读出不定长的内容。可以固定一个长度（比如4字节）用来保存请求数据大小，这样接收到数据的时候，可以先读取固定长度位置里面的值，值的大小就代表协议的长度，接着根据值的大小来读取协议体的数据，于是协议这个设计成这样：

![截图](abf406f4af3459f9fd2c9d33e54e68be.png)

上面的这个协议只实现了正确的断句效果，在 RPC 里面还行不通。因为对于服务方来说，它不知道这个协议体里面的二进制数据是通过哪种序列化方式生成的，也就不能还原出正确的语义，即不能把二进制数据转换为对象了，那服务方收到这个数据后也就不能完成调用了。于是我们需要把序列化方式单独拿出来，类似于协议长度一样用固定的长度存放，这些固定长度存放的参数可统称为“协议头”，于是协议被拆分成“协议头”和“协议体”。

在协议头里面，除了放入协议长度、序列化方式，还会放入一些协议提示、消息 ID、消息类型这样的参数，而协议体只放请求接口方法、请求的业务参数值和一些扩展属性。于是一个 RPC 协议就出来了：

![截图](a773c330d356a157bf71131080fdca1b.png)

#### 可扩展的协议

如果我们升级为新请求，往协议头里面多加 2bit 的参数并放在协议头最后。当用新的协议发出请求，而没有升级的应用受到请求后还是会按照原来的方式读取协议头，就会导致新加入的 2bit 协议头会被当做最开始的 2bit 的协议体，而丢弃最后 2bit 的协议体，导致整个协议题的数据还是错误的。

那如果我们把参数加在不定长的协议体里面行不行，而且协议体里面也会放入一些扩展属性。但是协议体里面的内容都是经过序列化了的，要获取到参数值，需要把整个协议体里面的数据反序列化一遍，导致代价很高。那为了保证能平滑地升级改造前后的协议，就需要设计一种可支持扩展的协议，让协议头可以支持扩展，而且在这之前也需要一个固定的地方读取长度，于是协议就可以分为：固定部分、协议头长度、协议体内容，前两部分可以统称为”协议头“。具体如下：

![截图](e8b88dbf00b367b186a10257a10d7593.png)

### 序列化

网络传输的数据必须是二进制数据，但调用方请求的出入参数都是对象。对象是不能直接在网络中传输，我们需要提前把它转成可传输的二进制，并要求转换算法是可逆的，这个过程我们一般叫做“序列化”。服务提供方就可以正确的从二进制数据中分割出不同的请求，同时根据请求类型和序列化类型，把二进制消息逆向还原成请求对象，称之为”反序列化“。

![截图](9c1b739f4c7fa3e6beca1406bd90856b.png)

序列化就是将对象转换成二进制数据的过程，而反序列化就是将对象转换为二进制数据的过程。

#### JDK原生序列化

在使用 Java 进行开发的时候，可以通过实现 Serializable 接口来实现序列化，具体实现是由 ObjectOutputStream 和 ObjectInputStream 完成的，其过程如下：

![截图](e3352d15cd0b68a67fe8be79df111293.png)

头部数据同来声明序列化协议、序列化版本，用于高低版本向后兼容
对象数据主要包括类名、签名、属性名、属性类型及属性值，还要开头结尾等数据，除了属性值属于真正的对象值，其他都是为了反序列化用的元数据
存在对象引用、继承的情况下，就是递归遍历“写对象”逻辑

#### JSON

JSON 是典型的 key-value 方式，没有数据类型，是一种文本型序列化框架。

JSON 序列化的两大问题：

- JSON进行序列化的额外空间开销比较大，对于数据量大的服务这意味着需要巨大的内存和磁盘开销。
- JSON 没有类型，但像 Java 这种强类型语言，需要通过反射同一解决，性能不太好。

所以如果 RPC 框架选用 JSON序列化，服务提供者与服务调用者之间传输的数据量要相对较小，否则将严重影响性能。

#### Hessian

Hessian 是动态类型、二进制、紧凑的，并且可跨语言移植的一种序列化框架。Hessian 协议要比 JDK、JSON更加紧凑，性能上要币 JDK、JSON 序列化高很多，而且序列化的字节数也要更小。有非常好的兼容性和稳定性，所以 Hessian 更加适合作为 RPC 框架远程通信的序列化协议。

但是 Hessian 本身也有问题，比如：

- Linked系列，LinkedHashMap、LinkedHashSet等，但是可以通过扩展 CollectionDeserializer 类修复
- Locale 类，可以通过扩展 ContextSerializerFactory 类修复
- Byte/Short 反序列化的时候编程 Integer

#### Protobuf

Protobuf 是 Google 内部的混合语言数据标准，是一种轻便、高效的结构化数据存储格式，可以用于结构化数据序列化，支持 Java、Python、C ++、Go 等语言。 Protobuf 使用时需要定义 IDL，使用不同语言的 IDL 编译器，生成序列化工具类

优点：

- 序列化后体积相比 JSON 之类的小很多
- IDL 能清晰地描述语义，保证应用程序之间的类型不会丢失，无需类似 XML 解析器
- 序列化反序列化速度很快，不需要通过反射获取类型
- 消息格式升级和兼容性不错，可以做到向后兼容

但是使用 Protobuf 对于具有反射和动态能力的语言来说使用起来很费劲，可以考虑使用 Protostuff。

Protostuff不需要依赖 IDL 文件，可以直接对 Java 领域对象进行反/序列化操作，在效率上根Protobuf差不多，生成的二进制格式和 Protobuf 是完全相同的，可以说是一个 Java 版本的 Protobuf 序列化框架。

缺点：

- 不支持 null
- Protostuff 不支持单纯的 Map、List 集合对象，需要包在对象里面

### RPC 框架在使用时需要注意哪些问题

1. 对象构造的过于复杂：属性很多，存在多层的嵌套。对象越复杂，序列化框架在序列化与反序列化时，就越浪费性能，消耗 CPU，出问题概率就会越高。
2. 对象过于庞大：比如一个大 List 或者大 Map，这种情况同样会严重浪费性能、CPU，并且序列化如此的一个大对象是很费时间的，肯定会直接影响到请求的耗时。
3. 使用序列化框架不支持的类作为入参类：应该尽量选择原生的，最为常用的集合类。
4. 对象有复杂的继承关系：大多数序列化框架在序列化对象时都会将对象的属性进行序列化，当有继承关系时，会不停地寻找父类，遍历属性。对象关系越复杂，就越浪费性能，容易出现序列化上面的问题。

在使用 RPC 框架的过程中，我们构造入参、返回值对象，注意以下几点：

1. 对象要尽量简单，没有太多的依赖关系，属性不要太多，尽量高内聚
2. 入参对象与返回值对象体积不要太打，更不要传太大的集合
3. 尽量使用简单的、常用的、开发语言原生的对象，尤其是集合类
4. 对象不要有复杂的继承关系，最好不要有父子类的情况

### Protobuf 语言

```golang
message <NameOfTheMessage> {
  <data-type> name_of_field_1 = tag_1;
  <data-type> name_of_field_2 = tag_2;
  ...
  <data-type> name_of_field_n = tag_n;
}
```

1. 消息类型采用驼峰格式
2. 消息域采用小写字母蛇形格式
3. 基本数据类型有：
   ```
       string, bool, bytes,
   
       float, double
   
       int32, int64, uint32, uint4, sint32, sint64, etc.
   ```
4. 数据类型也可以是自定义的消息类型和枚举类型
5. tag_n ：字段编号，用于二进制消息中识别各个字段，该编号在每个消息中唯一，且不可修改。注意在将message编码成二进制消息体时字段编号1-15将会占用1个字节，16-2047将占用两个字节。所以在一些频繁使用用的message中，你应该总是先使用前面1-15字段编号。
6. 注意在将message编码成二进制消息体时字段编号1-15将会占用1个字节，16-2047将占用两个字节。所以在一些频繁使用用的message中，你应该总是先使用前面1-15字段编号。

```golang

syntax = "proto3";                     // 1

package sample;                        // 2

option go_package = "./;pb";           // 3

import "google/api/annotations.proto"; // 4

message CreateMysqlRequest {           // 5
  string sql = 1;                      // 6
}

message CreateMysqlResponse {          // 7
  string body = 1;                     // 8
}

service MysqlService {                 // 9
  rpc SelectRecord(CreateMysqlRequest) returns (CreateMysqlResponse) { // 10
    option (google.api.http) = {       // 11
      post : "/v1/mysql/select"
      body : "*"
    };
  };
}
```

1. syntax = "proto3";，文件开头指定版本号；
2. package sample;，定义本服务的包名，避免不同服务相同消息类型产生冲突；
3. **option go_package = ".;pb";，生成 go 代码对应的路径和目录；**
4. import "google/api/annotations.proto";，需要引用外部 proto 文件时使用；
5. CreateMysqlRequest，请求消息类型定义；
6. sql，定义消息中域的名称和类型，支持的类型可参考下面的链接，后面的数字同一个类型下面必须唯一；
7. CreateMysqlResponse，响应消息类型；
8. body，定义消息中域的名称和类型，支持的类型可参考下面的链接，后面的数字同一个类型下面必须唯一；
9. MysqlService，定义服务的接口名称；
10. SelectRecord ，远程调用方法名；
11. option (google.api.http)，gRPC网关，用于支持 http 协议。

然后使用protoc把它编译就可以了编译的语法有点复杂

然后写服务器端和客户端的函数，导入生成的包的struct和函数声明就行

#### 编译

到.proto文件目录下

```sh
 protoc	--go_out=./ *.proto --go-grpc_out=./ *.proto
```

就生成在./下

## 3.3、拦截器的使用经验与理解

### 拦截器

通常客户端请求到达服务端的时候不会立即进行业务处理，而是进行一些预处理操作，比如监控数据采集（统计 QPS），链路追踪，身份信息校验，必传参数校验等等。gRPC 为此提供了一个拦截器（Interceptor）功能来实现这一系列的操作。按照通信方式可以分为一元拦截器（Unary Interceptor）和流拦截器（Streaming Interceptor），按照应用角色可以分为客户端拦截器（Client-Side Interceptor）和服务端拦截器（Server-Side Interceptor），具体类型如下

```
grpc.UnaryClientInterceptor
grpc.UnaryServerInterceptor
grpc.StreamClientInterceptor
grpc.StreamServerInterceptor
```

### 一元拦截器（Unary Interceptor）

```golang

type UnaryServerInterceptor func(
  ctx context.Context,     // 请求上下文，可以做一些超时处理
  req interface{},        // gRPC 请求参数
  info *UnaryServerInfo,  // gRPC 服务接口信息
  handler UnaryHandler,   // gRPC 实际调用方法
) (resp interface{}, err error)
```

### 流拦截器(Streaming Interceptor)

```golang
type StreamServerInterceptor func(
  srv interface{},         // 请求参数
  ss ServerStream,         // gRPC 服务端流信息
  info *StreamServerInfo,  // gRPC 服务接口信息
  handler StreamHandler    // gRPC 实际调用方法
) error
```

### 客户端拦截器（Client-Side Interceptor）

```
一元拦截器（Unary Interceptor）
type UnaryClientInterceptor func(
  ctx context.Context,    // 请求上下文，可以做一些超时处理
  method string,          // 请求方法
  req, reply interface{}, // 请求和响应
  cc *ClientConn,         // 连接信息
  invoker UnaryInvoker,   // 调用的 gRPC 方法
  opts ...CallOption      // gRPC 调用预处理接口活后处理接口
) error

流拦截器(Streaming Interceptor)
type StreamClientInterceptor func(
  ctx context.Context,    // 请求上下文 
  desc *StreamDesc,       // 调用 gRPC 方法流信息
  cc *ClientConn,         // 连接信息
  method string,          // 调用方法
  streamer Streamer,      // 流对象，通过 desc 初始化
  opts ...CallOption      // gRPC 调用预处理接口活后处理接口
) (ClientStream, error)
```

拦截器（interceptor）处理过程
拦截器的处理过程可以分为以下三个阶段：

预处理阶段（pre-processing）

调用 RPC 方法（invoking RPC method）

后处理阶段（post-processing）

```golang
客户端一元拦截器（Client-Side Unary Interceptor）
func mySqlUnaryClientInterceptor(
  ctx context.Context,
  method string,
  req, reply interface{},
  cc *grpc.ClientConn,
  invoker grpc.UnaryInvoker,
  opts ...grpc.CallOption) error {

  // Pre-processing logic
  // set requestID
  uuID, _ := uuid.NewRandom()
  ctx = context.WithValue(ctx, "requestID", uuID)

  start := time.Now()
  log.Printf("client unary interceptor pre-processing: requstID [%s]\n",
             ctx.Value("requestID"))

  // Invoking the remote method
  err := invoker(ctx, method, req, reply, cc, opts...)

  // Post processing logic
  end := time.Now()
  log.Printf("client unary interceptor post processing: time [%s]\n",
             end.Sub(start))

  return err
}

服务端一元拦截器（Server-Side Unary Interceptor）
// mySqlUnaryServerInterceptor 服务端一元拦截器
func mySqlUnaryServerInterceptor(
  ctx context.Context,
  req interface{},
  info *grpc.UnaryServerInfo,
  handler grpc.UnaryHandler) (interface{}, error) {

  // Pre-processing logic
  // set requestID
  uuID, _ := uuid.NewRandom()
  ctx = context.WithValue(ctx, "requestID", uuID)

  start := time.Now()
  log.Printf("server unary interceptor pre-processing: requstID [%s]\n",
             ctx.Value("requestID"))

  // Invoking the handler to complete the normal execution of a unary RPC.
  m, err := handler(ctx, req)

  // Post processing logic
  end := time.Now()
  log.Printf("server unary interceptor post processing: time [%s]\n", 
             end.Sub(start))
  return m, err
}
```

流拦截器预处理阶段和与一元拦截器类似，调用 RPC 方法和后处理两个阶段则不相同，stream 都是 streamer （客户端：clientStream, 服务端：serverStream）调用 SendMsg 和 RecvMsg 获取的，streamer 又是调用 RPC 方法获取的，因此在流拦截器中我们可以对 Streamer 进行封装，在封装的方法 SendMsg 实现流的预处理和 RecvMsg 实现流的拦截

```golang
客户端流拦截器（Client-Side Stream Interceptor）
// wrappedStream  用于包装 grpc.ClientStream 结构体并拦截其对应的方法。
type wrappedStream struct {
  grpc.ClientStream
}

func newWrappedStream(s grpc.ClientStream) grpc.ClientStream {
  return &wrappedStream{s}
}

func (w *wrappedStream) RecvMsg(m interface{}) error {
  log.Printf("Client Stream Interceptor Post Processing RecvMsg (Type: %T) at %v",
    m, time.Now().Format(time.RFC3339))
  return w.ClientStream.RecvMsg(m)
}

func (w *wrappedStream) SendMsg(m interface{}) error {
  log.Printf("Client Stream Interceptor Post Processing SendMsg (Type: %T) at %v",
    m, time.Now().Format(time.RFC3339))
  return w.ClientStream.SendMsg(m)
}

func mysqlClientStreamInterceptor(
  ctx context.Context, desc *grpc.StreamDesc,
  cc *grpc.ClientConn, method string,
  streamer grpc.Streamer, opts ...grpc.CallOption) (grpc.ClientStream, error) {
  // Pre-processing
  log.Println("Client Stream Interceptor Pre-processing : ", method)

  s, err := streamer(ctx, desc, cc, method, opts...)
  if err != nil {
    return nil, err
  }

  return newWrappedStream(s), nil
}

////////////////////////////////////////////////////
服务端流拦截器（Server-Side Stream Interceptor）
type wrappedStream struct {
  grpc.ServerStream
}

func newWrappedStream(s grpc.ServerStream) grpc.ServerStream {
  return &wrappedStream{s}
}

func (w *wrappedStream) RecvMsg(m interface{}) error {
  log.Printf("Server Stream Interceptor Post Processing RecvMsg (Type: %T) at %s",
    m, time.Now().Format(time.RFC3339))
  return w.ServerStream.RecvMsg(m)
}

func (w *wrappedStream) SendMsg(m interface{}) error {
  log.Printf("Server Stream Interceptor Post Processing SendMsg (Type: %T) at %v",
    m, time.Now().Format(time.RFC3339))
  return w.ServerStream.SendMsg(m)
}

func mysqlServerStreamInterceptor(
  srv interface{}, ss grpc.ServerStream,
  info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
  // Pre-processing
  log.Println("Server Stream Interceptor Pre-processing : ", info.FullMethod)

  // Invoking the StreamHandler to complete the execution of RPC invocation
  err := handler(srv, newWrappedStream(ss))
  if err != nil {
    log.Printf("RPC failed with error %v", err)
  }
  return err
}
```

使用方法

我们可以在调用 RPC 方法之前和之后添加自己的逻辑，比如链路追踪设置，请求耗时数据采集等等。客户端和服务端调用方法如下所示：

```golang

// 客户端添加一元拦截器
conn, err := grpc.Dial(address, grpc.WithInsecure(),
    grpc.WithUnaryInterceptor(mySqlUnaryClientInterceptor),
    grpc.WithStreamInterceptor(clientStreamInterceptor))
// 服务端添加一元拦截器
s := grpc.NewServer(
    grpc.UnaryInterceptor(mySqlUnaryServerInterceptor),
    grpc.StreamInterceptor(orderServerStreamInterceptor))
```

## 3.4、高性能原理

不同于文本传输的 JSON 和 XML，gRPC 使用基于二进制 proto buffers 协议进行客户端和服务端之间的通信，它是基于 HTTP2 协议标准实现的，不同进程间通信更加的快捷。

### grpc性能高

### protobuf比json性能高

他比json快六倍

protobuf的二进制数据流和json数据流如下图

![截图](52456e2506f5d781ea31a3db57199ba5.png)

1. 对比json数据和protobuf数据格式可以知道
体积小-无需分隔符：TLV存储方式不需要分隔符（逗号，双引号等）就能分隔字段，减少了分隔符的使用
2. 体积小-空字段省略：若字段没有被设置字段值，那么该字段序列化时的数据是完全不存在的，即不需要进行编码，而json会传key和空值的value
3. 体积小-tag二进制表示：是用字段的数字值然后转换成二进制进行表示的，比json的key用字符串表示更加省空间
4. 编解码快：tag的里面存储了字段的类型，可以直接知道value的长度，或者当value是字符串的时候，则用length存储了长度，可以直接从length后取n个字节就是value的值，而如果不知道value的长度，我们就必须要做字符串匹配
   
   细化了解protobuf的编码可以去看：varint 和 zigzag编码方式

### grpc性能高：http2.0比http1.1性能高

http2.0和http 1.* 还有 http1.1pipling的对比
示意图

![截图](3169d17d48f7702e5a8dbdc934897dfd.png)

1. http/1.* ：一次请求，一个响应，建立一个连接用完关闭，每一个请求都要建立一个连接
2. http1.1 pipeling: Pipeling解决方式为，若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞
3. http2: 多个请求可同时在一个连接上并行执行。某个请求任务耗时严重，不会影响到其它连接的正常执行

#### grpc 多路复用还有哪些优点

1. 减少了tcp的连接，降低了服务端和客户端对于内存，cpu等的压力
2. 减少了tcp的连接，保证了不频繁触发tcp重新建立，这样就不会频繁有慢启动
3. 减少了tcp的连接，使网络拥塞情况得以改善

#### 为什么http/1.1不能实现多路复用而http2.0可以？

因为http/1.1传输是用的文本，而http2.0用的是二进制分帧传输

#### 头部压缩

- 固定字段压缩：http可以通过http对body进行gzip压缩，这样可以节省带宽，但是报文中header也有很多字段没有进行压缩，比如cookie， user agent accept，这些有必要进行压缩
- 避免重复：大量请求和响应的报文里面又很多字段值是重复的，所以有必要避免重复性
- 编码改进：字段是ascii编码，效率低，改成二进制编码可以提高

以上通过HPACK算法来进行实现，算法主要包含三个部分

1. 静态字典：将常用的header字段整成字典,比如{":method":"GET"} 就可以用单个数字 2来表示
2. 动态字典：没有在静态字典里面的一些头部字段，则用动态字典
3. Huffman 编码： 压缩编码

#### 二进制分帧

1. 在二进制分帧层上，HTTP 2.0 会将所有传输的信息分割为更小的消息和帧,并对它们采用二进制格式的编码 ，其中HTTP1.x的首部信息会被封装到Headers帧，而我们的request body则封装到Data帧里面。
2. 这样分帧以后这些帧就可以乱序发送，然后根据每个帧首部的流标识符号进行组装
3. 对比http/1.1因为是基于文本以换行符分割每一条key：value则会有以下问题：

- 一次只能处理一个请求或者响应，因为这种以分隔符分割消息的数据，在完成之前不能停止解析
- 解析这种数据无法预知需要多少内存，会给服务端有很大压力

### 服务器主动推送资源

由于支持服务器主动推送资源，则可以省去一部分请求。比如你需要两个文件1.html,1.css，如果是http1.0则需要请求两次，服务端返回两次。但是http2.0则可以客户端请求一次，然后服务端直接回吐两次

## 3.5、重试策略理解

### 重试解决什么问题

从发生时长上而言，网络错误可以分为两类：

1. 长时间不可用，如光纤被挖断，机房被炸等
2. 短时间不可用，比如网络出现抖动，正在通信的对端机器正好重新上线等

而重试是应对短时故障利器，简单却异常有效。

###  短时间故障的产生原因

在任何环节下应用都会有可能产生短时故障。即使是在没有网络参与的应用里，软件bug或硬件故障或一次意外断电都会造成短时故障。短时故障是常态，想做到高可用不是靠避免这些故障的发生，而是去思考短时故障发生之后的应对策略。

就互联网公司的服务而言，通过冗余，各种切换等已经极大提高了整体应用的可用性，但其内部的短时故障却是连绵不断，原因有这么几个：

1. 应用所使用的资源是共享的，比如docker、虚拟机、物理机混布等，如果多个虚拟单位(docker镜像、虚拟机、进程等)之间的资源隔离没有做好，就可能产生一个虚拟单位侵占过多资源导致其它共享的虚拟单元出现错误。这些错误可能是短时的，也有可能是长时间的。
2. 现在服务器都是用比较便宜的硬件，即使是最重要的数据库，互联网公司的通常做法也是通过冗余去保证高可用。贵和便宜的硬件之间有个很重要的指标差异就是故障率，便宜的机器更容易发生硬件故障，虽然故障率很低，但如果把这个故障率乘以互联网公司数以万计、十万计的机器，每天都会有机器故障自然是家常便饭。
3. 除掉本身的问题外，现今的互联网架构所需要的硬件组件也更多了，比如路由和负载均衡等等，更多的组件，意味着通信链路上更多的节点，意味着增加了更多的不可靠。
4. 应用之间的网络通信问题，在架构设计时，对网络的基本假设就是不可靠，我们需要通过额外的机制弥补这种不可靠，有人问了，我的应用就是一个纯内网应用，网络都是内网，也不可靠么？嗯是的，不可靠。

### 处理短时故障的挑战

短时故障处理以下两点挑战

1. 感知。应用需要能够区分不同类型的错误，不同类型的错误对应的错误处理方式是不同的，没有哪种应对手段可以处理所有的错误。比如网络抖动我们简单重试即可，如果网络不可用，对于一个可靠的存储系统，可能就需要经历选主，副本切换等复杂操作才能保证数据的正确性。
2. 处理。如何选择一个合适的处理策略对于快速恢复故障、缩短响应时间以及减少对对端的冲击是非常重要的。

### 重试分为几步

1. 感知错误。通常我们使用错误码识别不同类型的错误。比如在REST风格的应用里面，HTTP的status code可以用来识别不同类型的错误。
2. 决策是否应该重试。不是所有错误都应该被重试，比如HTTP的4xx的错误，通常4xx表示的是客户端的错误，这时候客户端不应该进行重试操作。什么错误可以重试需要具体情况具体分析，对于网络类的错误，我们也不是一股脑都进行重试，比如zookeeper这种强一致的存储系统，发生了network partition之后，需要经过一系列复杂操作，简单的重试根本不管用。
3. 选择重试策略。选择一个合适的重试次数和重试间隔非常的重要。如果次数不够，可能并不能有效的覆盖这个短时间故障的时间段，如果重试次数过多，或者重试间隔太小，又可能造成大量的资源(CPU、内存、线程、网络)浪费。合适的次数和间隔取决于重试的上下文。举例：如果是用户操作失败导致的重试，比如在网页上点了一个按钮失败的重试，间隔就应该尽量短，确保用户等待时间较短；如果请求失败成本很高，比如整个流程很长，一旦中间环节出错需要重头开始，典型的如转账交易，这种情况就需要适当增加重试次数和最长等待时间以尽可能保证短时间的故障能被处理而无需重头来过。.
4. 失败处理与自动恢复。短时故障如果短时间没有恢复就变成了长时间的故障，这个时候我们就不应该再进行重试了，但是等故障修复之后我们也需要有一种机制能自动恢复。

### 常见的重试时间间隔策略

1. **指数避退**。重试间隔时间按照指数增长，如等 3s 9s 27s后重试。指数避退能有效防止对对端造成不必要的冲击，因为随着时间的增加，一个故障从短时故障变成长时间的故障的可能性是逐步增加的，对于一个长时间的故障，重试基本无效。
2. **重试间隔线性增加**。重试间隔的间隔按照线性增长，而非指数级增长，如等 3s 7s 13s后重试。间隔增长能避免长时间等待，缩短故障响应时间。
3. **固定间隔。**重试间隔是一个固定值，如每3s后进行重试。
4. **立即重试。**有时候短时故障是因为网络抖动造成的，可能是因为网络包冲突或者硬件有问题等，这时候我们立即重试通常能解决这类问题。但是立即重试不应该超过一次，如果立即重试一次失败之后，应该转换为指数避退或者其它策略进行，因为大量的立即重试会给对端造成流量上的尖峰，对网络也是一个冲击。
5. **随机间隔。**当服务有多台实例时，我们应该加入随机的变量，比如A服务请求B服务，B服务发生短时间不可用，A服务的实例应该避免在同一时刻进行重试，这时候我们对间隔加入随机因子会很好的在时间上平摊开所有的重试请求。

### gRPC是如何进行重试的

#### 如何感知错误

gRPC有自己一套类似HTTP status code的错误码，每个错误码都是个字符串，如 INTERNAL、ABORTED、UNAVAILABLE。

#### 如何决策

对于哪些错误可以重试是可配置的。通常而言，只有那些明确标识对端没有接收处理请求的错误才需要被重试，比如对端返回一个 UNAVAILABLE 错误，这代表对端的服务目前处于不可用状态。但也可以配置一个更加激进的重试策略，但关键是需要保证这些被重试的gRPC请求是幂等的，这个需要服务使用者和提供者共同协商出一个可以被重试的错误集合。

#### 重试策略

gRPC的重试策略分为两类 

重试策略，失败后进行重试。
对冲策略，一次请求会给对端发出多个相同请求，只要有一个成功就认为成功。

**重试之时间策略**
gPRC用了上面我们提到的 指数避退+随机间隔 组合起来的方式进行重试

```
/* 伪码 */

ConnectWithBackoff()
  current_backoff = INITIAL_BACKOFF
  current_deadline = now() + INITIAL_BACKOFF
  while (TryConnect(Max(current_deadline, now() + MIN_CONNECT_TIMEOUT))
         != SUCCESS)
    SleepUntil(current_deadline)
    current_backoff = Min(current_backoff * MULTIPLIER, MAX_BACKOFF)
    current_deadline = now() + current_backoff +
      UniformRandom(-JITTER * current_backoff, JITTER * current_backoff)
```

INITIAL_BACKOFF：第一次重试等待的间隔
MULTIPLIER：每次间隔的指数因子
JITTER：控制随机的因子
MAX_BACKOFF：等待的最大时长，随着重试次数的增加，我们不希望第N次重试等待的时间变成30分钟这样不切实际的值
MIN_CONNECT_TIMEOUT：一次成功的请求所需要的时间，因为即使是正常的请求也需要有响应时间，比如200ms，我们的重试时间间隔显然要大于这个响应时间才不会出现请求明明已经成功，但却进行重试的操作。
通过指数的增加每次重试间隔，gRPC在考虑对端服务和快速故障处理中间找到了一个平衡点。

**重试之次数策略**
上面的算法里面没有关于次数的限制，gRPC中的最大重试次数是可配置的，硬限制的最大值为5次，设置这个硬限制的目的我想主要还是出于对对端服务的保护，避免一些人为的错误。

**对冲之时间策略**
对冲策略里面，请求是按照如下逻辑发出的：

第一次正常的请求正常发出
在等待固定时间间隔后，没有收到正确的响应，第二个对冲请求会被发出
再等待固定时间间隔后，没有收到任何前面两个请求的正确响应，第三个会被发出
一直重复以上流程直到发出的对冲请求数量达到配置的最大次数
一旦收到正确响应，所有对冲请求都会被取消，响应会被返回给应用层

**对冲之次数策略**
次数和上面重试是一样的限制，都是5次。

#### 重试失败

当然不能一直重试，对于重试失败，gRPC有以下的策略以顾全大局，对于每个server，客户端都可配置一个针对该server的限制策略如下：

```
"retryThrottling": {
  "maxTokens": 10,
  "tokenRatio": 0.1
}
```

对于每个server，gRPC的客户端都维护了一个 token_count 变量，变量初始值为配置的 maxTokens 值，每次RPC请求都会影响这个 token_count 变量值：

每次失败的RPC请求都会对 token_count 减1
每次成功的RPC请求都会对 token_count 增加tokenRation值
如果 token_count <= (maxTokens / 2)，那么后续发出的请求即使失败也不会进行重试了，但是正常的请求还是会发出去，直到这个 token_count > (maxTokens / 2) 才又恢复对失败请求的重试。这种策略可以有效的处理长时间故障。

<br/>

# 4、基础命令及 shell 脚本（掌握）

基础的性能观测工具，例如 dmesg、lsof、lostat、netstat、iostat、vmstat、mpstat、sar 等。
基础的 shell 脚本编写，例如升级脚本：停止服务、解压、校验、替换、回滚，重启服务等。
学习地址：
Learn Shell - Free Interactive Shell Tutorial
Home | Linux Journey
监控 Linux 性能的 18 个命令行工具-阿里云开发者社区
Linux 性能检测常用的 9 个基本命令-腾讯云开发者社区-腾讯云

# 5、Mysql、mongodb、redis（掌握）

学习地址：
mysql：https://xorm.io/zh/ 
mongodb：MongoDB | Golang 中文学习文档，mongo package - go.mongodb.org/mongo￾driver/mongo - Go Packages
redis 官方文档：Documentation | Redis
Go Redis 客户端包的官方文档：redis package - github.com/go-redis/redis - Go Packages

# 6、扩展资料

## 6.1.keepalived（Keepalived for Linux），掌握故障切换原理，选主机制，健康检查

## 6.2.LVS（The Linux Virtual Server Project - Linux Server Cluster for Load Balancing），掌握工作原理和架构，调度模型，LVS 集群高可用实现，负载均衡策略，持久化机制

## 6.3.nsq（MQ）（NSQ Docs 1.2.1 - A realtime distributed messaging platform）熟悉 nsq，会进行 nsq 消息的写入和读取，并理解其中的概念

## 6.4.k8s（apiserver、webhook、crd 等）（动态准入控制 | Kubernetes，定制资源 | Kubernetes）

## 6.5.etcd（Documentation versions | etcd，clientv3 package - go.etcd.io/etcd/client/v3 - Go Packages）

## 6.6.Envoy XDS 协议，各 DS 的概念及作用，以及它们之间的引用关系。（xDS REST and gRPC protocol &mdash; envoy 1.29.0-dev-2de016 documentation），

# 4.技术概念

目标：了解以下技术概念，包括其使用的技术、工作原理、应用场景、特点等。

1. ## 负载均衡
   
   负载均衡的分类
负载均衡就是一种计算机网络技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁碟驱动器或其他资源中分配负载，以达到最佳化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。那么，这种计算机技术的实现方式有多种。大致可以分为以下几种，其中最常用的是四层和七层负载均衡：
   
   ![截图](805dfa8e869dd153fd813ab2757907f4.png)
   
   **四层负载均衡**:四层负载均衡工作在OSI模型的传输层，由于在传输层，只有TCP/UDP协议，这两种协议中除了包含源IP、目标IP以外，还包含源端口号及目的端口号。四层负载均衡服务器在接受到客户端请求后，以后通过修改数据包的地址信息（IP+端口号）将流量转发到应用服务器。
**七层负载均衡**:七层负载均衡工作在OSI模型的应用层，应用层协议较多，常用http、radius、dns等。七层负载就可以基于这些协议来负载。这些应用层协议中会包含很多有意义的内容。比如同一个Web服务器的负载均衡，除了根据IP加端口进行负载外，还可根据七层的URL、浏览器类别、语言来决定是否要进行负载均衡。
   
   ### 常见负载均衡算法
   
   **轮询(Round Robin)**
请求到达后，将客户端发送到负载均衡器的请求依次轮流地转发给服务集群的某个节点。
   
   优点：实现简单，每个集群节点平均分担所有的请求。
   
   缺点：当集群中服务器硬件配置不同、性能差别大时，无法区别对待。引出下面的算法。
   
   **随机(Random)**
随机选取集群中的某个节点来处理该请求，由概率论的知识可知，随着请求量的变大，随机算法会逐渐演变为轮询算法，即集群各个节点会处理差不多数量的请求。
   
   优点：简单使用，不需要额外的配置和算法。
   
   缺点：随机数的特点是在数据量大到一定量时才能保证均衡，所以如果请求量有限的话，可能会达不到均衡负载的要求。
   
   **加权**
加权算法主要是根据集群的节点对应机器的性能的差异，给每个节点设置一个权重值，其中性能好的机器节点设置一个较大的权重值，而性能差的机器节点则设置一个较小的权重值。权重大的节点能够被更多的选中。它是和随机、轮训一起使用的。
   
   优点：可以根据机器的具体情况，分配不同的负载，达到能者多劳。
   
   缺点：需要额外管理加权系数。
   
   **最小连接数**
主要是根据集群的每个节点的当前连接数来决定将请求转发给哪个节点，即每次都将请求转发给当前存在最少并发连接的节点。
   
   优点：可以根据集群节点的负载情况来进行请求的动态分发，即机器性能好，处理请求快，积压请求少的节点分配更多的请求。避免某个节点因为处理超过自身所能承受的请求量而导致宕机或者响应过慢。
   
   **hash**
将对请求的IP地址或者URL计算一个哈希值，然后与集群节点的数量进行取模来决定将请求分发给哪个集群节点。它不是真正意义上的负载均衡，在某些意义上也是一个单点服务。
   
   优点：实现简单
   
   缺点：如果某个节点挂了，会使得一部分流量不可用。
   
   ### 负载均衡分类
   1. 硬件负载均衡
常见的硬件有比较昂贵的F5和Array等商用的负载均衡器，它的优点简单，有专业的人负责；缺点就是 贵 如果你是土豪，可以考虑，但是对于规模较小的网络服务来说暂时还没有需要使用。
   2. 软件负载均衡
目前使用最广泛的三种负载均衡软件Nginx/LVS/HAProxy，他们都是开源免费的负载均衡软件，这些都是通过软件级别来实现，所以费用较低。
   3. 成熟的架构
负载均衡业界早已有成熟的架构，比较常用的有LVS+Keepalived、Nginx+Keepalived、HAProxy+Keepalived。
   
   ### 常见负载均衡软件
   
   #### Nginx
   
   Nginx的负载均衡支持：
   
   rr：轮叫，轮流分配到后端服务器；
wrr：权重轮叫，根据后端服务器负载情况来分配；
lc：最小连接，分配已建立连接最少的服务器上；
wlc：权重最小连接，根据后端服务器处理能力来分配。
   
   Nginx优点：
   
   1.简单：安装和配置比较简单、测试也简单
2.稳定：单机一般能支撑几万次的并发量
3.轻量：能ping通就就能进行负载功能
4.易用：明确的错误码、超时提醒
5.强大：负载均衡、反向代理、WEB容器等功能
   
   Nginx缺点：
   
   1.仅能支持http、https和Email协议
2.对后端服务器的健康检查， 只支持通过端口来检测，不支持通过url来检测
   
   roundrobin：轮询，轮流分配到后端服务器； 
static-rr：根据后端服务器性能分配； 
leastconn：最小连接者优先处理； 
source：根据请求源IP，与Nginx的IP_Hash类似。
   
   #### LVS
   
   LVS的负载均衡支持：
   
   roundrobin：轮询，轮流分配到后端服务器；
static-rr：根据后端服务器性能分配；
leastconn：最小连接者优先处理；
source：根据请求源IP，与Nginx的IP_Hash类似。
   
   LVS优点：
   
   1.高效：工作在网络4层之上 仅作分发之用，没有流量的产生，这个特点也决定了它在负载均衡软件里的性能最强的 ，对内存和cpu资源消耗比较低
2.易用：配置性比较低，简化操作成本
3.稳定：本身抗负载能力很强，自身有完整的双机热备方案
4.应用广：因为LVS工作在4层，所以它几乎可以对所有应用做负载均衡，包括http、tcp、数据库、在线聊天室等
   
   LVS缺点：
   
   1.不能做动静分离
2.大型网站LVS+Keepalived实施起来就比较复杂，配置成本高
   
   #### HAProxy
   
   HAProxy负载均衡策略非常多，包括：
   
   roundrobin，表示简单的轮询，这个不多说，这个是负载均衡基本都具备的；
static-rr，表示根据权重，建议关注；
leastconn，表示最少连接者先处理，建议关注；
source，表示根据请求源IP，这个跟Nginx的IP_hash机制类似，我们用其作为解决session问题的一种方法，建议关注；
ri，表示根据请求的URI；
rl_param，表示根据请求的URl参数’balance url_param’ requires an URL parameter name；
hdr(name)，表示根据HTTP请求头来锁定每一次HTTP请求；
rdp-cookie(name)，表示根据据cookie(name)来锁定并哈希每一次TCP请求。
   
   优点：
   
   HAProxy的优点能够补充Nginx的一些缺点， 比如支持Session的保持，Cookie的引导；同时支持通过获取指定的url来检测后端服务器的状态。
   
   HAProxy跟LVS类似，本身就只是一款负载均衡软件；单纯从效率上来讲HAProxy会比Nginx有更出色的负载均衡速度，在并发处理上也是优于Nginx的。
   
   支持TCP协议的负载均衡转发，可以对MySQL读进行负载均衡，对后端的MySQL节点进行检测和负载均衡。
   
   <br/>
2. ##  HA
2.     ### 一、什么是高可用集群
2.     高可用集群（High Availability Cluster，简称HA Cluster），是指以减少服务中断时间为目的的服务器集群技术。它通过保护用户的业务程序对外不间断地提供服务，把因为软件，硬件，人为造成的故障对业务的影响降低到最小程度。总而言之就是保证公司业务7*24小时不宕机
2.     ### 二、高可用集群的衡量标准
2.     通常用平均无故障时间(MTTF：mean time to failure)来衡量系统的可靠性，用平均故障维修时间（MTTR：Mean Time Between Failures）来度量系统的可维护性。于是可用性被定义为： HA=MTTF/(MTTF+MTTR)*100%。
2.     ![截图](312fe1b25660c18708a5d31bbe93715c.png)
2.     ### 三、高可用集群实现原理
2.     高可用集群主要是实现自动侦测（Auto-Detect）故障、自动切换/故障转移（FailOver）和 自动恢复(FailBack)。
2.     1：自动侦测、故障检测：通过集群各节点间心跳信息判断节点是否出现故障；
2.     2：当有节点（一个或多个）和另外节点互相接收不到对方心跳信息时，如何决定哪一部分接点是正常运行的，而哪一部分是出现故障需要隔离的呢？
2.     这时候通过法定票数（quorum）决定，即当有节点故障时，节点间投票决定哪个节点是有问题，得票数大于半数为合法，每个节点可以设置其票数，当一个节点能和另一个节点保持心跳信息，该节点就获取了另一个节点的票数，该节点获得就是正常节点，反之为故障节点。
2.     四、高可用集群的分类
2.     双机热备（Active/Passive）
2.     多节点热备（N+1）
2.     多节点共享存储（N-TO-N）
2.     共享存储热备 （Split Site）
2.     ### 五、高可用集群软件
2.     在高可用集群朝多样化、易操作维护等方向迅速发展的今天，市场上的集群软件产品也品种繁多，但对于任何一款高可用集群产品，故障监视都是最核心的功能。监视资源种类的多少和监视层次的深浅，都成为评价一款集群软件高可用性的重要指标。目前市面上成熟的高可用集群软件已有不少，比如国外就有RedHat 公司的RHCS、Novell公司的Novell Cluster Service、Steeleye公司的Lifekeeper for Linux、Keepalived等，在国内其实也有，比如中兴新支点的Newstart HA 就已经做得不错。
3. ##  WAF
3.     Web应用防火墙可以防止Web应用免受各种常见攻击，比如SQL注入，跨站脚本漏洞（XSS）等。WAF也能够监测并过滤掉某些可能让应用遭受DOS（拒绝服务）攻击的流量。WAF会在HTTP流量抵达应用服务器之前检测可疑访问，同时，它们也能防止从Web应用获取某些未经授权的数据。
3.     ### 1，WAF预防的攻击类型
3.     开放Web应用安全项目（OWASP）所例举的攻击类型，都在WAF实施时考虑的范围内，其中几种比较常见的攻击类型如下：
3.     #### 1.1 跨站脚本漏洞（XSS）
3.     攻击者通过往Web页面里插入恶意Script代码，当用户浏览该页面时，嵌入在Web页面里的Script代码会被执行，从而达到恶意攻击用户的目的。
3.     XSS大概分为两类：
3.     反射型攻击。恶意代码并没有保存在目标网站，通过引诱用户点击一个链接到目标网站的恶意链接来实施攻击。
存储型攻击。恶意代码被保存到目标网站的服务器中，这种攻击具有较强的稳定性和持久性，比较常见的场景是在博客，论坛等社交网站上。
XSS攻击能够：
3.     获取用户Cookie，将用户Cookie发送回黑客服务器。
3.     获取用户的非公开数据，比如邮件、客户资料、联系人等。
3.     #### 1.2 SQL注入
3.     通过在目标数据库执行可疑SQL代码，以达到控制Web应用数据库服务器或者获取非法数据的目的。SQL注入攻击可以用来未经授权访问用户的敏感数据，比如客户信息、个人数据、商业机密、知识产权等。
3.     SQL注入攻击是最古老，最流行，最危险的Web应用程序漏洞之一。比如查询?id=1，如果不对输入的id值1做检查，可以被注入?id=1 or 1=1从而得到所有数据。
3.     SQL注入的产生原因通常表现在以下几方面：
3.     不当的类型处理。
3.     不安全的数据库配置。
3.     不合理的查询集处理。
3.     不当的错误处理。
3.     转义字符处理不合适。
多个提交处理不当。
3.     #### 1.3 Cookie篡改
3.     Cookie篡改是攻击者通过修改用户Cookie获得用户未授权信息，进而盗用身份的过程。攻击者可能使用此信息打开新账号或者获取用户已存在账号的访问权限。
3.     很多Web应用都会使用Cookie保存用户的Session信息，当用户使用Cookie访问该应用时，Web应用能够识别用户身份，监控用户行为并提供个性化的服务。而如果Cookie的使用缺乏安全机制的话，也很容易被人篡改和盗用，并被攻击者用来获取用户的隐私信息。
3.     #### 1.4 未经验证的输入
3.     Web应用往往会依据HTTP的输入来触发相应的执行逻辑。而攻击者则很容易对HTTP的任何部分做篡改，比如URL地址、URL请求参数、HTTP头、Cookies等，以达到攻破Web应用安全策略的目的。
3.     #### 1.5 第七层Dos攻击
3.     我们将在下文中详细介绍WAF针对第七层（应用层）的Dos攻击防护。
3.     #### 1.6 网页信息检索（Web scraping）
3.     通过一些工具来获取网页内容，并从中提炼出有用的网站数据信息。
3.     ### WAF部署方式
3.     WAF可以按照下面几种方式进行部署：
3.     #### 2.1 透明代理模式
3.     WAF代理了WEB客户端和服务器之间的会话，并对客户端和server端都透明。从WEB客户端的角度看，WEB客户端仍然是直接访问服务器，感知不到WAF的存在。
3.     这种部署模式的优点是对网络的改动最小，通过WAF的硬件Bypass功能在设备出现故障或者掉电时可以不影响原有网络流量，只是WAF自身功能失效。
3.     缺点是网络的所有流量都经过WAF，对WAF的处理性能要求高。**采用该工作模式无法实现负载均衡功能。**
3.     #### 2.2 反向代理模式
3.     反向代理模式是指将真实服务器的地址映射到反向代理服务器上，此时代理服务器对外就表现为一个真实服务器。当代理服务器收到HTTP的请求报文后，将该请求转发给其对应的真实服务器。后台服务器接收到请求后将响应先发送给WAF设备，由WAF设备再将应答发送给客户端。
3.     **这种部署模式需要对网络进行改动，配置相对复杂，除了要配置WAF设备自身的地址和路由外，还需要在WAF上配置后台真实WEB服务器的地址和虚地址的映射关系。**
3.     优点则是**可以在WAF上实现负载均衡。**
3.     #### 2.3 路由代理模式
3.     它与网桥透明代理的唯一区别就是该代理工作在路由转发模式而非网桥模式，其它工作原理都一样。由于工作在路由（网关）模式因此需要为WAF的转发接口配置IP地址以及路由。
3.     **这种部署模式需要对网络进行简单改动，要设置该设备内网口和外网口的IP地址以及对应的路由。工作在路由代理模式时，可以直接作为WEB服务器的网关，但是存在单点故障问题，同时也要负责转发所有的流量。该种工作模式也不支持服务器负载均衡功能。**
3.     ### 3，WAF安全模式
3.     WAF可以采用白名单和黑名单两种安全模式，也可以两者相结合。
3.     在白名单安全模式下，所有不在名单中的请求类型都会被拒绝；而黑名单正好相反，只会拒绝在黑名单上的请求类型，其它通通放行。
3.     对于新的、还不为开发人员所知晓的攻击类型，白名单可以很好地工作。
3.     黑名单相对来说更容易实现，但问题是维护成本高，因为很多时候我们并不能够枚举所有的攻击类型。
3.     ### 4，WAF和传统防火墙的区别
3.     传统防火墙主要用来保护服务器之间传输的信息，而WAF则主要针对Web应用程序。网络防火墙和WAF工作在OSI7层网络模型的不同层，相互之间互补，往往能搭配使用。
3.     网络防火墙工作在网络层和传输层，它们没有办法理解HTTP数据内容，而这个正式WAF所擅长的。网络防火墙一般只能决定用来响应HTTP请求的服务器端口是开还是关，没办法实施更高级的、和数据内容相关的安全防护。
4. ##  传输协议
4.     <br/>
5. ##  交换机
   
   交换机又称交换式集线器，它们俩很相似，都是基于MAC识别的，但是又有本质上的区别。是一种用于电（光）信号转发的网络设备。它可以为接入交换机的任意两个网络节点提供独享的电信号通路，把传输的信息送到符合要求的相应路由上。发生在数据链路层。
   
   数据传输：集线器工作的时候，如果局域网中的一台电脑要发送消息，则局域网内的所有电脑都可以接收到这个消息，安全性较差，而且每一次只能有一个发送，只有这个发送完毕其他电脑才能再发送，这称为半双工模式。而交换机有“记忆功能”，它能根据相应的MAC地址直接有目的的发送到目标电脑。但是如果向一台新的电脑发送消息，那么传输方式也将是广播，只有找到这台电脑， 并记住它的MAC地址后，以后才能直接发送给它。通过交换机连接的电脑可以同时发送消息互不影响，就像我们平时打电话一样，这称为全双工模式，传输速率比集线器大大提高。
   宽带占用上：通过集线器，所有的电脑都共享一个宽带，如果宽带是100M，有5台电脑，则每台电脑只有20M；如果通过交换机，则所有的电脑都是100M。
   通过比较可以发现，交换机与集线器相比有很大优势，所以现在基本上都用交换机，集线器已经渐渐被淘汰了。
   
   ### 工作原理
   
   交换机工作于OSI参考模型的第二层，即数据链路层。交换机内部的CPU会在每个端口成功连接时，通过将MAC地址和端口对应，形成一张MAC表。在今后的通讯中，发往该MAC地址的数据包将仅送往其对应的端口，而不是所有的端口。因此，交换机可用于划分数据链路层广播，即冲突域；但它不能划分网络层广播，即广播域。
交换机拥有一条很高带宽的背部总线和内部交换矩阵。交换机的所有的端口都挂接在这条背部总线上，控制电路收到数据包以后，处理端口会查找内存中的地址对照表以确定目的MAC（网卡的硬件地址）的NIC（网卡）挂接在哪个端口上，通过内部交换矩阵迅速将数据包传送到目的端口，目的MAC若不存在，广播到所有的端口，接收端口回应后交换机会“学习”新的MAC地址，并把它添加入内部MAC地址表中。使用交换机也可以把网络“分段”，通过对照IP地址表，交换机只允许必要的网络流量通过交换机。通过交换机的过滤和转发，可以有效的减少冲突域。
   
   ##  路由器
6. 路由器：（Router）是连接因特网中各局域网、广域网的设备。在路由器中记录着路由表，它会根据信道的情况自动选择和设定路由，以最佳路径，按前后顺序发送信号。发生在网络层。
6.     路由器是连接不同的网段的，负责将局域网连接到广域网和互联网中，并找到网络中数据传输最合适的路径。大家通过同一个路由器上网共用一个宽带，上网要相互影响。
7. ##  反向代理
7.     在反向代理的情形中, 从浏览器的角度看还是类似于直接访问, 但它的请求在服务端被透明的代理了. 类似于我们在网上从一个声称是厂家直销的"伪厂家"那里购物, 这个伪厂家实际还是把我们的订单转给了真正的厂家, 并从中拿了货给我们, 只是我们无从知道这一切幕后的交易, 如下图所示:
7.     ![截图](339ee06bb93479bbcfd47d48546decf0.png)
7.     代理机制就是借助第三方访问或者被访问。实现代理服务的是代理服务器。在无歧义的情况下，代理一次可以指代理机制、代理功能或代理服务器。正向代理就是访问代理，即访问方主动设置的代理，可以大致理解为代理设置在访问方。反向代理就是被访问代理，即被访问方设置的代理，可以大致理解为代理设置在被访问方。所谓正向和反向，是以访问服务的方向为参考的，以访问方为主体，发起访问的是正向代理，返回服务的是反向代理。
8. ##  Linux 常用命令
