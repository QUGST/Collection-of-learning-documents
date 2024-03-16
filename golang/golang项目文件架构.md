## 1. go项目分类
 library和executable。
 1. library是以构建库为目的的Go项目,library类又叫package类
 2. executable则是以构建二进制可执行文件为目的的Go项目。又叫command
根据项目的复杂程度又分为七种结构
![[../assets/Pasted image 20240316100004.png]]

# 2.modules项目具体布局
参考来源[Organizing a Go module - The Go Programming Language](https://go.dev/doc/modules/layout)
## 2.1 basic package
```
project-root-directory/
├── go.mod
├── modname.go
└── modname_test.go
或
project-root-directory/
├── go.mod
├── modname.go
├── modname_test.go
├── auth.go
├── auth_test.go
├── hash.go
└── hash_test.go
```
basic package 下只有一个导出package，这个package包含一个或多个源文件，
## 2.2basic command
```
project-root-directory/
├── go.mod
└── main.go
或
project-root-directory/
├── go.mod
├── main.go
├── auth.go
├── auth_test.go
├── hash.go
└── hash_test.go
```
与basic package 类似
## 2.3package with supporting package
稍复杂或规模稍大的一些package类项目，会将很多功能分拆到supporting packages中，并且通常项目作者是不希望导出这些supporting packages的，这样这些supporting packages便可以不作为暴露的API的一部分，后续重构和优化起来十分方便，对package的用户也是无感的。这样Go官方建议将这些supporting packages放入internal目录
>   
> 注：internal目录是**Go 1.4版本**引入的机制，简单来说放在internal中的包是local的，不能导出到module之外，但module下的某些内部代码可以导入internal下的包。如今一般都会将internal放在项目的根目录下，所以项目下的所有代码都可以导入internal下的包。
```text
project-root-directory/
├── go.mod
├── modname.go
├── modname_test.go
└── internal/
    ├── auth/
    │   ├── auth.go
    │   └── auth_test.go
    └── hash/
        ├── hash.go
        └── hash_test.go
```
modname.go或modname_test.go可以通过下面导入语句使用internal下面的包：

```text
import "github.com/someuser/modname/internal/auth"
```
## 2.4 command with supporting packages
与package with supporting packages类似
```text
project-root-directory/
├── go.mod
├── main.go
└── internal/
    ├── auth/
    │   ├── auth.go
    │   └── auth_test.go
    └── hash/
        ├── hash.go
        └── hash_test.go
```
不同的是，main.go使用的包名为main，这样Go编译器才能将其构建为command。
##  2.5multiple packages
作为一个库项目，作者可能要暴露不止一个package，可能是多个packages。这不会给Go项目目录布局带来过多复杂性，只需多建立几个导出package的目录。
```text
project-root-directory/
├── go.mod
├── modname.go
├── modname_test.go
├── auth/
│   ├── auth.go
│   ├── auth_test.go
│   └── token/
│       ├── token.go
│       └── token_test.go
├── hash/
│   ├── hash.go
│   └── hash_test.go
└── internal/
    └── trace/
        ├── trace.go
        └── trace_test.go
```
此时可以访问到多个导出包
```text
github.com/user/modname
github.com/user/modname/auth
github.com/user/modname/hash
github.com/user/modname/auth/token
```
## 2.6 multiple commands
```txt
project-root-directory/
├── go.mod
├── prog1/
│   └── main.go
├── prog2/
│   └── main.go
└── internal/
    └── trace/
        ├── trace.go
        └── trace_test.go
```
将每个command放置在一个单独的目录下(比如prog1、prog2等)，supporting packages和之前的建议一样，统一放到internal下面。
command：

```text
$go build github.com/someuser/modname/prog1
$go build github.com/someuser/modname/prog2
```

command的用户通过下面步骤可以安装这些命令：

```text
$go install github.com/someuser/modname/prog1@latest
$go install github.com/someuser/modname/prog2@latest
```
## 2.7 multiple packages and commands
```text
project-root-directory/
├── go.mod
├── modname.go
├── modname_test.go
├── auth/
│   ├── auth.go
│   ├── auth_test.go
│   └── token/
│       ├── token.go
│       └── token_test.go
├── hash/
│   ├── hash.go
│   └── hash_test.go
├── internal/
│       └── trace/
│           ├── trace.go
│           └── trace_test.go
└── cmd/
    ├── prog1/
    │   └── main.go
    └── prog2/
        └── main.go
```

为了区分导出package和command，这个示例增加了一个专门用来存放command的**cmd目录**，prog1和prog2两个command都放在这个目录下。这也是Go语言的一个惯例。

这样，这个示例项目既导出了下面的包：

```text
github.com/user/modname
github.com/user/modname/auth
github.com/user/modname/hash
```

又包含了两个可安装使用的command，用户按下面步骤安装即可：

```text
$go install github.com/someuser/modname/cmd/prog1@latest
$go install github.com/someuser/modname/cmd/prog2@latest
```

# 3.本地大型项目目录
## 3.1程序文件
### /web
**前端代码**存放目录。

- 存放Web 应用程序特定的组件：主要有静态 Web 资源，服务器端模板和单页应用（Single-Page App，SPA）等。
### /cmd
存放**当前项目的可执行文件**。

- `cmd` 目录下的每一个**子目录名称都应该匹配可执行文件**。例如，把组件 `main` 函数所在的文件夹统一放在 `/cmd` 目录下。
- 不要在 `/cmd` 目录中放置太多的代码，我们应该将公有代码放置到 `/pkg` 中，将私有代码放置到 `/internal` 中并在 `/cmd` 中引入这些包，**保证 main 函数中的代码尽可能简单和少**。

### /internal

存放**私有应用和库代码**。

- 如果一些代码，你不希望被其他项目/库导入，可以将这部分代码放至`/internal`目录下。一般存储一些比较专属于当前项目的代码包。这是在代码编译阶段就会被限制的，该目录下的代码不可被外部访问到。一般有以下子目录：
    - /router 路由
    - /application 存放命令与查询
        - /command
        - query
    - /middleware 中间件
    - /model 模型定义
    - /repository 仓储层，封装数据库操作
    - /response 响应
    - /errmsg 错误处理
- 在`/internal`目录下应存放每个组件的源码目录，当项目变大、组件增多时，扔可以将新增的组件代码存放到`/internal`目录下
- internal目录并不局限在根目录，在各级子目录中也可以有internal子目录，也会同样起到作用。
### /pkg

存放**可以被外部应用**使用的代码

- `/pkg`目录下时可以被其他项目引用的包，所以我们将代码放入该目录下时候一定要慎重。
- 在非根目录下也可以加入pkg目录，很多项目会在internal目录下加入pkg表示内部共享包库。
- 建议：一开始将所有的共享代码存放在`/internal/pkg`目录下，当确认可以对外开发时，再转至到根目录的`/pkg`目录下
### /vendor

存放**项目依赖**

- 可以通过命令行`go mod wendor`创建
- 如果创建的是一个`Go`库，不要提交wendor依赖包
### /third_party

存放放一些**第三方的资源工具文件**。
### /test

存放**整个应用的测试、测试数据及一些集成测试**等，

- 相较于单元测试在每个go文件对应的目录下，test目录偏向于整体
- 在某些子项目内也会有局部项目的测试会放在子项目的test中。
- 需要Go忽略该目录中的内容，可以使用`/test/data`或`/test/testdata`目录下
- Go会忽略`.`或`_`开头的目录或文件
### /config或/configs

**配置文件或者配置文件模板**所在的文件夹。

- 配置中不能携带敏感信息，可用占位符代替
### /init

存放**初始化系统和进程管理配置文件**
### /deployments 或 /deploy

**存放 `Iaas`、`PaaS` 系统和容器编排部署配置和模板。**
### /go.mod
**保存引包信息**
### /scripts 
**保存脚本文件**
## 3.2 文档
### /README.md

项目的 `README` 文件一般包含了**项目的介绍、功能、快速安装和使用指引、详细的文档链接以及开发指引等**。
### /docs

**各类文档所在目录。**

- 存放设计文档、开发文档和用户文档等

###  /LICENSE

**版权文件**

- 可以是私有的，也可以是开源的。
- 常用的开源协议有：`Apache 2.0`、`MIT`、`BSD`、`GPL`、`Mozilla`、`LGPL`。

### /api

**当前项目对外提供的各种不同类型的 API 接口定义文件**

- 其中可能包含类似 openapi、swagger 的目录，这些目录包含了当前项目对外提供和依赖的所有 API 文件。

### /CONTRIBUTING.md

**用来说明如何贡献代码，如何开源协同等**

- 规范协同流程
- 降低第三方开发者贡献代码的难度。
## 2.3项目管理
### /Makefile

**对项目进行管理**

- 执行静态代码检查、单元测试、编译等功能。

### /build

**存放安装包和持续集成相关的文件**。

### /website

如果不使用 Github pages，则在这里放置项目的网站数据。

### /assets

项目使用的其他资源 (如图片等)。

### /tools

存放这个**项目的支持工具**。

- 这些工具可导入来自 `/pkg` 和 `/internal` 目录的代码。

### /githooks

`Git` 钩子。
## 总目录
一个总体的框架如下
