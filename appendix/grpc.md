## ProtoBuf 與 gRPC
[ProtoBuf](https://github.com/google/protobuf) 是一套接口描述語言（[Interface Definition Language，IDL](https://en.wikipedia.org/wiki/Interface_description_language)），類似 Apache 的 [Thrift](https://thrift.apache.org/)。

相關處理工具主要是 [protoc](https://github.com/google/protobuf)，基於 C++ 語言實現。

用戶寫好 `.proto` 描述文件，之後便可以使用 protoc 自動編譯生成眾多計算機語言（C++、Java、Python、C#、Golang 等）的接口代碼。這些代碼可以支持 gRPC，也可以不支持。

[gRPC](https://github.com/grpc/grpc) 是 Google 開源的 RPC 框架和庫，已支持主流計算機語言。底層通信採用 HTTP2 協議，比較適合互聯網場景。gRPC 在設計上考慮了跟 ProtoBuf 的配合使用。

兩者分別解決不同問題，可以配合使用，也可以分開單獨使用。

典型的配合使用場景是，寫好 `.proto` 描述文件定義 RPC 的接口，然後用 protoc（帶 gRPC 插件）基於 `.proto` 模板自動生成客戶端和服務端的接口代碼。

### ProtoBuf

需要工具主要包括：

* 編譯工具：[protoc](https://github.com/google/protobuf)，以及一些官方沒有帶的語言插件；
* 運行環境：各種語言的 protobuf 庫，不同語言有不同的安裝來源；

語法類似 C++ 語言，可以參考 ProtoBuf 語言規範：[https://developers.google.com/protocol-buffers/docs/proto](https://developers.google.com/protocol-buffers/docs/proto)。

比較核心的，`message` 是代表數據結構（裡面可以包括不同類型的成員變量，包括字符串、數位、數組、字典……），`service` 代表 RPC 接口。變量後面的數位是代表進行二進制編碼時候的提示信息，1~15 表示熱變量，會用較少的字節來編碼。另外，支持導入。

默認所有變量都是可選的（optional），repeated 則表示數組。主要 service rpc 接口接受單個 message 參數，返回單個 message。如下所示。

```protobuf
syntax = "proto3";
package hello;

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
  repeated int32 number=4;
}

service HelloService {
  rpc SayHello(HelloRequest) returns (HelloResponse){}
}
```

編譯最關鍵的參數是輸出語言格式參數，例如，python 為 `--python_out=OUT_DIR`。

一些還沒有官方支持的語言，可以通過安裝 protoc 對應的 plugin 來支持。例如，對於 Go 語言，可以安裝

```sh
$ go get -u github.com/golang/protobuf/{protoc-gen-go,proto} // 前者是 plugin；後者是 go 的依賴庫
```

之後，正常使用 `protoc --go_out=./ hello.proto` 來生成 hello.pb.go，會自動調用 `protoc-gen-go` 插件。

ProtoBuf 提供了 `Marshal/Unmarshal` 方法來將數據結構進行序列化操作。所生成的二進制文件在存儲效率上比 XML 高 3~10 倍，並且處理性能高 1~2 個數量級。

### gRPC

相關工具主要包括：

* 運行時庫：各種不同語言有不同的[安裝方法](https://github.com/grpc/grpc/blob/master/INSTALL.md)，主流語言的包管理器都已支持。
* protoc，以及 gRPC 插件和其它插件：採用 ProtoBuf 作為 IDL 時，對 .proto 文件進行編譯處理。

類似其它 RPC 框架，gRPC 的庫在服務端提供一個 gRPC Server，客戶端的庫是 gRPC Stub。典型的場景是客戶端發送請求，同步或異步調用服務端的接口。客戶端和服務端之間的通信協議是基於 HTTP2 的 [gRPC](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) 協議，支持雙工的流式保序消息，性能比較好，同時也很輕。

採用 ProtoBuf 作為 IDL，則需要定義 service 類型。生成客戶端和服務端代碼。用戶自行實現服務端代碼中的調用接口，並且利用客戶端代碼來發起請求到服務端。一個完整的例子可以參考 [https://github.com/grpc/grpc-go/blob/master/examples/helloworld](https://github.com/grpc/grpc-go/blob/master/examples/helloworld/)。

以上面 proto 文件為例，需要執行時添加 gRPC 的 plugin：

```sh
$ protoc --go_out=plugins=grpc:. hello.proto
```

gRPC 更多原理可以參考[官方文檔：http://www.grpc.io/docs](http://www.grpc.io/docs/)。


#### 生成服務端代碼

服務端相關代碼如下，主要定義了 HelloServiceServer 接口，用戶可以自行編寫實現代碼。

```go
type HelloServiceServer interface {
        SayHello(context.Context, *HelloRequest) (*HelloResponse, error)
}

func RegisterHelloServiceServer(s *grpc.Server, srv HelloServiceServer) {
        s.RegisterService(&_HelloService_serviceDesc, srv)
}
```

用戶需要自行實現服務端接口，代碼如下。

比較重要的，創建並啟動一個 gRPC 服務的過程：

* 創建監聽套接字：`lis, err := net.Listen("tcp", port)`；
* 創建服務端：`grpc.NewServer()`；
* 註冊服務：`pb.RegisterHelloServiceServer()`；
* 啟動服務端：`s.Serve(lis)`。


```go
type server struct{}

// 這裡實現服務端接口中的方法。
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "Hello " + in.Name}, nil
}

// 創建並啟動一個 gRPC 服務的過程：創建監聽套接字、創建服務端、註冊服務、啟動服務端。
func main() {
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterHelloServiceServer(s, &server{})
	s.Serve(lis)
}
```

編譯並啟動服務端。


#### 生成客戶端代碼

生成的 go 文件中客戶端相關代碼如下，主要和實現了 HelloServiceClient 接口。用戶可以通過 gRPC 來直接調用這個接口。

```go
type HelloServiceClient interface {
        SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloResponse, error)
}

type helloServiceClient struct {
        cc *grpc.ClientConn
}

func NewHelloServiceClient(cc *grpc.ClientConn) HelloServiceClient {
        return &helloServiceClient{cc}
}

func (c *helloServiceClient) SayHello(ctx context.Context, in *HelloRequest, opts ...grpc.CallOption) (*HelloResponse, error) {
        out := new(HelloResponse)
        err := grpc.Invoke(ctx, "/hello.HelloService/SayHello", in, out, c.cc, opts...)
        if err != nil {
                return nil, err
        }
        return out, nil
}
```

用戶直接調用接口方法：創建連接、創建客戶端、調用接口。

```go
func main() {
	// Set up a connection to the server.
	conn, err := grpc.Dial(address, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewHelloServiceClient(conn)

	// Contact the server and print out its response.
	name := defaultName
	if len(os.Args) > 1 {
		name = os.Args[1]
	}
	r, err := c.SayHello(context.Background(), &pb.HelloRequest{Name: name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.Message)
}
```

編譯並啟動客戶端，查看到服務端返回的消息。