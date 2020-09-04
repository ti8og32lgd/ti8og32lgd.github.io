title: 记录一个gRPC学习中使用protobuf命令生成代码遇到的坑
author: Someone
tags: []
categories: []
date: 2020-09-04 10:58:00
---
最近在学习gRPC框架，里面的protobuf模块需要用单独的protoc编译器和针对go语言的protoc-gen-go插件联合对.proto文件进行编译，这一框架中文教程较少而且比较混乱，于是从官方教程https://www.grpc.io/docs/languages/go/basics/ 上开始学习。但是在这个基础章节Basics Tutorial一步一步学习中出现了这一条编译.proto文件生成代码的命令无法执行的问题，具体的命令是：

> protoc -I routeguide/ routeguide/route_guide.proto --go_out=plugins=grpc:routeguide
–go_out: protoc-gen-go: plugins are not supported; use ‘protoc –go-grpc_out=…’ to generate gRPC

作为一个新手，完全不知道怎么回事，我首先按照提示将--go_out=plugins=grpc:routeguide替换成了--go-grpc_out=，生成了代码，但是经过compare发现生成的代码和教程中原来的代码有很大的不一样，具体的例子不能使用新生成的代码，这样生成的代码和原来生成的代码不兼容。

然后我又参考了下protobuf官方文档https://developers.google.com/protocol-buffers 里面给出的命令还是使用参数 `--go_out`，完全没有`--go-grpc_out`的内容，这可是官方文档啊。

最后经过搜索引擎查询，一篇博文 里，作者是在升级项目时遇到了这个问题，最后发现是包的问题，protoc-gen-go有两个包的来源：

1）github.com/golang/protobuf
2）google.golang.org/protobuf

可以使用--go_out的包是 1）github.com/golang/protobuf，而使用--go-grpc_out的包是2）google.golang.org/protobuf。

google.golang.org/protobuf是更新的包，使用了新的工具来生成gRPC代码，可是生成的同样的proto3版本的代码竟然不兼容！而且在官方文档中，要求从google.golang.org/protobuf安装包，但是命令还是之前的(github包的)命令，这使得不只是新手，就算是常用者在使用的过程中也很可能遇到这个问题。