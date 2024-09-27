+++
title = 'gRPC: what is it? An introduction...'
date = 2024-09-25T19:14:38-03:00
draft = true
tags = ["go", "rpc", "grpc", "distributed systems"]
+++

The first time I heard about RPC, I was in a distributed systems class while completing my bachelor's in computer science. I like it but didn't fully understand why I should use it instead of REST. Time passed, and I started working at a company where the legacy services use SOAP. I remember thinking: "hm, that's cool! It is like RPC but using XML instead! Years later, I heard for the first time about gRPC, but I've never fully understood it until now!

As my blog works as a personal documentation, I decided to document what I learned about RPC and gRPC.

# Let's go then: what is RPC?

[RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) is an acronym for `Remote Procedure Call`. In other words, you send procedures/commands to a remote server. Straight forward, that is RPC! This image describes how it works: 

![RPC](/img/posts/en/rpc.png)

RPC works with UDP and TCP. It is up to you to decide what suits you better. If you don't need an answer or lose some packages, go for UDP. If that is important for you, go for TCP. For those who like to read the RFCs, you can find it [here](https://datatracker.ietf.org/doc/html/rfc1831)!

## OK, but how does RPC differ from REST, for example?

Both are a way to design your APIs, but a REST architecture has very well-defined principles that have to be followed to achieve a RESTfull architecture. RPC also has some principles that should be defined between the client and the server. For the RPC client, it is like calling a local procedure. Also, RPC doesn't care about the connection being UDP or TCP, but for REST, if you want to be RESTfull, you cannot use UDP.

To learn more about both, I recommend this guide from AWS about [RPC x REST](https://aws.amazon.com/en/compare/the-difference-between-rpc-and-rest/).

## And how to implement an RPC server with Go?

We have two main entities, the client and the server.

### Starting from the server...

The server is like any web server, used by microservices. Let's define then which connection protocol we will use. For the sake of simplicity, TCP was chosen.

```golang
func main() {
  addr, err := net.ResolveTCPAddr("tcp", "0.0.0.0:52648")
  if err != nil {
    log.Fatal(err)
  }

  conn, err := net.ListenTCP("tcp", addr)
  if err != nil {
    log.Fatal(err)
  }
  defer conn.Close()

  // ...
}
```

With our server in hand, we need to build the handler, or in RPC terms, the procedure to be executed. It is important to mention that is necessary to define the arguments and the response for the HTTP connection. To simplify this example, the same structure will be used for the args and the response.

```golang
type Args struct {
  Message string
}

type Handler int

func (h *Handler) Ping(args *Args, reply *Args) error {
  fmt.Println("Received message: ", args.Message)

  switch args.Message {
  case "ping", "Ping", "PING":
    reply.Message = "pong"
  default:
    reply.Message = "I don't understand"
  }

  fmt.Println("Sending message: ", reply.Message)
  return nil
}
```

Now the procedure exists and can handle the connections:

```golang
func main() {
  // ...

  h := new(Handler)
  log.Printf("Server listening at %v", conn.Addr())
  s := rpc.NewServer()
  s.Register(h)
  s.Accept(conn)
}
```

### Defining the client...

As the client and the server have to follow the same structure for request and response, we need to redefine the same args structure here.

```golang
type Args struct {
  Message string
}
```

Easing, we will build an interactive client: it will be reading the STDIN and if it receives any new entry, it send it to the server. To be more didactic, let's print all responses.

```golang
func main() {
  client, err := rpc.Dial("tcp", "localhost:52648")
  if err != nil {
    log.Fatal(err)
  }

  for {
    log.Println("Please, inform the message:")

    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()

    args := Args{Message: scanner.Text()}
    log.Println("Sent message:", args.Message)
    reply := &Args{}
    err = client.Call("Handler.Ping", args, reply)
    if err != nil {
      log.Fatal(err)
    }

    log.Println("Received message:", reply.Message)
    log.Println("-------------------------")
  }
}
```

It is possible to notice that we need to inform the server's address and which handler we wanna execute. As we are using binary data, the package [encoding/gob](https://pkg.go.dev/encoding/gob) is used to transport the data. If you go for another codec, like `JSON`, you must tell your server to use it. 

To see the full example, please visit the [PoC](https://github.com/mfbmina/poc_rpc).

# And what is gRPC?

gRPC is a framework for writing services using RPC. This framework is part of [CNCF](https://www.cncf.io/) and the [official documentation]((https://grpc.io/)) says it was created by Google:
> gRPC was initially created by Google, which has used a single general-purpose RPC infrastructure called Stubby to connect the large number of microservices running within and across its data centers for over a decade. In March 2015, Google decided to build the next version of Stubby and make it open source. The result was gRPC, which is now used in many organizations outside of Google to power use cases from microservices to the "last mile" of computing (mobile, web, and Internet of Things).

It can work with multiple operational systems and architectures. It also has the following core features:
- Idiomatic client libraries in 11 languages
- Highly efficient on wire and with a simple service definition framework
- Bi-directional streaming with `http/2` based transport
- Pluggable auth, tracing, load balancing and health checking

## And how we can use gRPC with Go?

For our luck, Go is one of the 11 languages with official libraries. It is important to say that the framework uses [Protocol Buffer](https://protobuf.dev/) to serialize the message. The first step then is to install locally the protobuf and its Go plugins:

```sh
brew install protobuf
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

You must add the plugins to your PATH:

```sh
export PATH="$PATH:$(go env GOPATH)/bin"
```

### protobuf's magic...

Let's build the `.proto` files! This file will contain our service and the handlers. For each handler, we also define what is the expected request and response.

```proto
syntax = "proto3";

option go_package = "github.com/mfbmina/poc_grpc/proto";

package ping_pong;

service PingPong {
  rpc Ping (PingRequest) returns (PingResponse) {}
}

message PingRequest {
  string message = 1;
}

message PingResponse {
  string message = 1;
}
```

With the `.proto` done, let's make the `gRPC + protobuf` magic happens. The plugins above can generate all needed code for the protobuf client/server with the following command:

```sh
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative proto/ping_pong.proto
```

Two files will be created: `ping_pong.pb.go` and `ping_pong_grpc.pb.go`. I highly recommend you look into those files, so you can understand better how the client and the server are structured. Now, we can build the server:

### Building the server...

To compare with the regular RPC, let's use the same logic: if we get a `PING`, we respond with `PONG`. Let's define the server and the request handler using the definitions for the request and response body from protobuf. Later, we just start the server:

```go
type server struct {
  pb.UnimplementedPingPongServer
}

func (s *server) Ping(_ context.Context, in *pb.PingRequest) (*pb.PingResponse, error) {
  r := &pb.PingResponse{}
  m := in.GetMessage()
  log.Println("Received message:", m)

  switch m {
  case "ping", "Ping", "PING":
    r.Message = "pong"
  default:
    r.Message = "I don't understand"
  }

  log.Println("Sending message:", r.Message)

  return r, nil
}

func main() {
  l, err := net.Listen("tcp", ":50051")
  if err != nil {
    log.Fatal(err)
  }

  s := grpc.NewServer()
  pb.RegisterPingPongServer(s, &server{})
  log.Printf("Server listening at %v", l.Addr())

  err = s.Serve(l)
  if err != nil {
    log.Fatal(err)
  }
}
```

### For the client...

To use our server, we need a client. It is also very simple because the gRPC lib has everything that we need. We initialize the client and call the desired RPC method, in this case, the `Ping`. It all came from the generated code from protobuf.

```go
func main() {
	conn, err := grpc.NewClient("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
	c := pb.NewPingPongClient(conn)

	for {
		log.Println("Enter text: ")
		scanner := bufio.NewScanner(os.Stdin)
		scanner.Scan()
		msg := scanner.Text()
		log.Printf("Sending message: %s", msg)

		ctx, cancel := context.WithTimeout(context.Background(), time.Second)
		defer cancel()
		r, err := c.Ping(ctx, &pb.PingRequest{Message: msg})
		if err != nil {
			log.Fatal(err)
		}

		log.Printf("Received message: %s", r.GetMessage())
		log.Println("-------------------------")
	}
}
```

To see the full example, please visit the [PoC gRPC](https://github.com/mfbmina/poc_grpc).

# Last two cents

gRPC is not more than an abstraction over the conventional RPC, using protobuf as the serializer and making requests over `http/2`. There are some performance considerations when using `http/2`, and in some cases, using `http/1` can be faster! I recommend you to read this [benchmark](https://github.com/duh-rpc/duh-go-benchmarks) and this open issue on [golang/go](https://github.com/golang/go/issues/47840) about the `http/2`. However, when dealing with requests with a large/complex body, gRPC turns out to be a great solution due to having the protobuf as the serializer, which is much faster than serializing `JSON` as an example. Elton Minetto wrote a great [blog post](https://eltonminetto.dev/en/post/2024-08-05-json-vs-flatbuffers-vs-protobuf/) explaining better those alternatives and benchmarking them. Another great benefit of using protobuf is solving the contract inconsistency between the client and the server since they both use the same `.proto` files. 

My final recommendation is to use gRPC if you and your team need it and are mature enough for it. Usually, web applications don't need all the performance that gRPC aims to give. It is usual to find people who have never worked with it, which can cause a slow development process and a loss in software quality. In this post, I've cited a lot of references, and they are all listed below:
- [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call)
- [RPC RFC]((https://datatracker.ietf.org/doc/html/rfc1831))
- [RPC x REST](https://aws.amazon.com/en/compare/the-difference-between-rpc-and-rest/)
- [PoC RPC](https://github.com/mfbmina/poc_rpc)
- [net/rpc](https://pkg.go.dev/net/rpc)
- [encoding/gob](https://pkg.go.dev/encoding/gob)
- [CNCF - Cloud Native Computing Foundation](https://www.cncf.io/)
- [gRPC](https://grpc.io/)
- [Protocol Buffer](https://protobuf.dev/)
- [PoC gRPC](https://github.com/mfbmina/poc_grpc)
- [http/1 x http/2 x gRPC](https://github.com/duh-rpc/duh-go-benchmarks)
- [http/2 issue](https://github.com/golang/go/issues/47840)
- [JSON x Protobuffers X Flatbuffers](https://eltonminetto.dev/en/post/2024-08-05-json-vs-flatbuffers-vs-protobuf/)

Thanks for reading, and bye!
