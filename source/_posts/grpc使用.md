---
title: grpc使用
date: 2019-03-31 16:12:23
tags:
---

**关于Rpc**

- Rpc的全程是 Remote Procedure Call，它是一种进程间的通信方式。允许像调用本地服务一样调用远程服务，它的具体实现方式可以不同，例如Spring的HTTP Invoker，Facebook的Thrift二进制私有协议通信和Google的高性能RPC框架gRPC。

- RPC框架特点:
  - 简单：RPC概念的语义十分简单和清晰，这样建立分布式计算就更容易。
  - 高效：过程调用看起来十分简单高效。
  - 通用：在单机计算中过程往往是不同算法和API，跨进程调用最重要的是通用的通信机制。

**gRPC**

- gRPC是一个高性能、通用的开源RPC框架，其中Google主要面向移动应用开发并基于HTTP/2协议标准而设计，基于ProtoBuf（Protocol Buffers）序列化协议开发，支撑众多开发语言。gRPC基于HTTP/2标准设计，带来诸如双向流、流控、头部压缩、单TCP连接上的多复用请求等特性。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。
- gRPC目前提供C、Java和Go语言版本，这三个版本的源码全部托管在Github上，分别是grpc、grpc-java、grpc-go。其中C版本支持C、C++、Node.js、Ruby、OC、PHP、C#等。

- gRPC框架的主要特性
  - 支持Protobuf:gRPC使用ProtoBuf的IDL(.proto)来定义数据结构和服务。Protobuf是一个灵活、高效、结构化的数据序列化框架，相比于XML等传统的序列化工具，它更小、更快、更简单。Protobuf支持数据结构化一次可以到处使用，甚至跨语言使用，通过代码生成工具可以自动生成不同语言版本的源代码，甚至可以在使用不同版本的数据结构和结构进程进行数据传递，实现数据结构的前向兼容。目前gRPC仅支持Protobuf，且不支持在浏览器中使用。由于gRPC的设计能够支持多种数据格式，所以读者能够很容易实现对其他数据格式(XML,JSON等)的支持。
  - 支持多种语言：gRPC支持多种语言，并能够根据语言类型自动生成客户端和服务端代码，grpc-java已经支持安卓开发。
  - 基于HTTP/2设计：由于gRPC基于HTTP/2标准设计，所以相对于其他的RPC框架，gRPC带来了更多强大功能，如双向流、头部压缩、多复用请求等。这些功能给移动设备带来重大益处，如节省带宽、降低TCP链接次数、节省CPU使用和延长电池寿命等。同时，gRPC还能够提高了云端服务和Web应用的性能，gRPC即能在客户端应用，也能够在服务端应用，从而以透明的方式实验客户端和服务端的通信和简化通信系统的构建。

**Go使用grpc开发Demo**

- 安装grpc-go，这里在国内会timeout，可以通过"科学上网"的方式解决

  ```
  go get google.golang.org/grpc
  ```

- 安装Protocol Buffers v3

  去<https://github.com/google/protobuf/releases>下载最新的稳定的版本，然后解压缩，把里面的文件放到`$PATH`中

- 安装插件

  ```
  go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
  ```

- 把`$GOPATH/bin`添加到`$PATH`中:

  ```
  export PATH=$PATH:$GOPATH/bin
  ```

- 示例代码结构

  ```
  |kv_demo
  ├── kvrpc
  │   ├── kvrpc.pb.go
  │   ├── kvrpc.proto
  ├── test_client.go
  └── test_server.go
  ```

- 示例代码里面有三个重要文件，kvrpc.proto是协议文件，test_client.go是客户端代码、test_server.go是服务端代码。

- kvrpc.proto，协议中定义了四个结构体KVRequest，VRequest，Status，ValueReply，KVRequest是Post请求参数返回Status反应请求是否成功。VRequest是Get请求参数，ValueReply返回取到的值。

  ```
  syntax = "proto3";
  
  package kvrpc;
  
  service Api {
      rpc PostKV (KVRequest) returns (Status) {}
      rpc GetV (VRequest) returns (ValueReply) {}
  }
  
  message KVRequest {
      string key = 1;
      string value = 2;
  }
  
  message VRequest {
      string key = 1;
  }
  
  message Status {
      string isok = 1;
  }
  
  message ValueReply {
      string value = 1;
  }
  ```

- 我们来看服务端代码

  ```
  package main
  
  import (
      "golang.org/x/net/context"
      "google.golang.org/grpc"
      "google.golang.org/grpc/reflection"
      pb "github.com/turingkv/kvrpc"
      "net"
      "log"
  )
  
  type RServer struct{}
  
  func (s *RServer) PostKV(ctx context.Context, in *pb.KVRequest) (*pb.Status, error) {
  
      return &pb.Status{Isok: "yes"}, nil
  }
  
  func (s *RServer) GetV(ctx context.Context, in *pb.VRequest) (*pb.ValueReply, error) {
  
      return &pb.ValueReply{Value: "testvalue"}, nil
  }
  
  func main(){
  	  //tcp监听端口
        lis, _ := net.Listen("tcp", ":8000")
        log.Printf("Sever listen on: %d", 8000)
        //新建服务器
        s := grpc.NewServer()
        //注册服务
        pb.RegisterApiServer(s, &RServer{})
        reflection.Register(s)
        s.Serve(lis)
  }
  ```

- 再来看客户端代码

  ```
  package main
  
  import (
      "golang.org/x/net/context"
      "google.golang.org/grpc"
      pb "github.com/turingkv/kvrpc"
      "log"
  )
  
  func main(){
      conn, err := grpc.Dial("127.0.0.1:8000", grpc.WithInsecure())
          if err != nil {
          log.Fatalf("did not connect: %v", err)
      }
      defer conn.Close()
      c := pb.NewApiClient(conn)
  
      r, err := c.PostKV(context.Background(), &pb.KVRequest{Key:"key", Value:"hello turingturing"})
      if err != nil {
          log.Fatalf("could not greet: %v", err)
      }
      log.Printf("Set Value Status: %s", r.Isok)
  
      r_, err := c.GetV(context.Background(), &pb.VRequest{Key:"key"})
      if err != nil {
          log.Fatalf("could not greet: %v", err)
      }
      log.Printf("Get Value: %s", r_.Value)
  }
  ```

- 测试结果

  ```
  //服务端
  [root@node1 turingkv]# go run test_server.go 
  2019/04/07 01:53:28 Sever listen on: 8000
  
  //客户端
  [root@node1 turingkv]# go run test_client.go 
  2019/04/07 01:51:28 Set Value Status: yes
  2019/04/07 01:51:28 Get Value: testvalue
  ```