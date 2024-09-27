+++
title = 'gRPC: onde vive? o que come?'
date = 2024-09-27T14:14:38-03:00
draft = false
tags = ["go", "rpc", "grpc", "distributed systems"]
+++

A primeira vez que ouvi falar sobre RPC foi em uma aula de sistema distribuídos, ainda quando estava cursando a graduação em Ciência da Computação. Achei legal, mas na época lembro de não compreender exatamente o porque eu usaria RPC ao invés de usar o padrão REST, por exemplo. Passa o tempo, e vou trabalhar em uma empresa em que parte do sistema legado era utilizando SOAP. Lembro de pensar: "hmm, interessante! Parece com RPC, mas traféga XML". Anos depois, ouço pela primeira vez falar sobre gRPC, mas nunca entendi complementamente o que era, o que comia e pra que servia.

Como meu blog serve muito de documentação pessoal, achei legal documentar aqui o que aprendi sobre, começando sobre o que é RPC e depois indo para o gRPC.

# Vamos lá, o que é RPC?

[RPC](https://pt.wikipedia.org/wiki/Chamada_de_procedimento_remoto) é uma sigla para `Remote Procedure Call` (em Português `Chamada de Procedimento Remoto`). Ou seja, você envia procedimentos/comandos para um servidor remoto. Sendo simples e direto, isso é RPC. Ele funciona da seguinte forma:

![RPC](/img/posts/br/rpc.png)

O RPC funciona tanto sobre UDP, quanto TCP. Cabe a você ver o que faz sentido para seu caso de uso! Se você não se importa com uma eventual resposta ou até mesmo em perder pacotes, UDP. Caso contrário, use TCP. Para aqueles que gostam de ler as RFCs, pode encontrar o link [aqui!](https://datatracker.ietf.org/doc/html/rfc1831)

## OK, mas como o RPC se difere de uma chamada REST, por exemplo?

Ambos são maneiras de arquiteturar APIs, porém, a arquitetura REST possuí principíos muito bem definidos e que devem ser seguidos para se ter uma arquitetura RESTfull. O RPC até possui principios, mas eles são definidos entre cliente e servidor. Para o cliente RPC, é como se ele tivesse chamando um procedimento local.

Outro ponto importante é que para o RPC, não importa muito se a conexão é TCP ou UDP. Já para APIs REST, se você quiser seguir o RESTfull, não vai conseguir utilizar UDP.

Para quem quiser saber mais sobre, recomendo este excelente guia da AWS sobre [RPC x REST](https://aws.amazon.com/pt/compare/the-difference-between-rpc-and-rest/).

## E como implementar um servidor RPC com Go?

Temos duas entidades principais, o cliente e o servidor.

### Começando pelo servidor...

O servidor é um servidor WEB, comumente usado em qualquer microsserviço. Vamos definir então o tipo de conexão que vamos utilizar, para nosso caso, TCP foi o escolhido:

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


Com o nosso servidor instânciado, vamos precisar de um `handler`, ou seja, nosso procedimento a ser executado. É importante dizer que precisamos definir sempre o que vai vir de argumentos e o que vamos responder na nossa conexão HTTP. Para simplificar nossa prova de conceito, vamos receber uma estrutura de argumentos e responder essa mesma estrutura:

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

Tendo o nosso processador criado, agora é só fazer ele aceitar as conexões:

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

### Definindo o cliente...

Como o cliente e o servidor precisam seguir a mesma estrutura definida, vamos redefinir aqui a estrutura de argumentos a ser enviada pelo nosso cliente:

```golang
type Args struct {
  Message string
}
```

Para facilitar, vamos fazer um cliente interativo: ele vai ficar lendo entradas no STDIN e ao receber uma nova entrada, ele envia para o nosso servidor. Para fins didáticos, vamos escrever a resposta recebida.

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

Pode-se notar que precisamos fornecer o endereço de onde o servidor está rodando e qual o `Handler` (procedimento) que queremos executar.

Um adendo importante é que estamos trafegando dados binários e por padrão o Go vai utilizar o [encoding/gob](https://pkg.go.dev/encoding/gob). Se quiser utilizar um outro padrão, como por exemplo `JSON`, vai ser preciso dizer ao seu servidor que aceite aquele codec novo.

Para quem quiser ver o código completo, é só acessar a [PoC.](https://github.com/mfbmina/poc_rpc)

# E o que é gRPC?

gRPC é um framework para se escrever aplicações usando RPC! Esse framework hoje é mantido pela [CNCF](https://www.cncf.io/) e segundo a [documentação oficial](https://grpc.io/) foi criado pela Google:
> gRPC was initially created by Google, which has used a single general-purpose RPC infrastructure called Stubby to connect the large number of microservices running within and across its data centers for over a decade. In March 2015, Google decided to build the next version of Stubby and make it open source. The result was gRPC, which is now used in many organizations outside of Google to power use cases from microservices to the "last mile" of computing (mobile, web, and Internet of Things).

Além de funcionar em diversos sistemas operacionais e em diversas arquiteturas, o gRPC ainda possui as seguintes vantagens:
- Bibliotecas idiomáticas em 11 linguagens;
- Framework simples para definição do seu serviço e extremamente performático.
- Fluxo bi-direcional de dados utilizando `http/2` para transporte;
- Funcionalidades extensíveis como autenticação, tracing, balanceador de carga e verificador de saúde.

## E como utilizar o gRPC com Go?

Para nossa sorte, Go é uma das 11 linguagens que tem bibliotecas oficiais para o gRPC! É importante falar que esse framework usa o [Protocol Buffer](https://protobuf.dev/) para serializar a mensagem. O primeiro passo então é instalar o protobuf de forma local e os plugins para Go:

```sh
brew install protobuf
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

E adicionar os plugins ao seu PATH:

```sh
export PATH="$PATH:$(go env GOPATH)/bin"
```

### A mágica do protobuf...

Vamos então criar nossos arquivos `.proto`! Nesse arquivo vamos definir nosso serviço, quais os handlers que ele possui e para cada handler, qual a requisição e qual resposta esperadas.

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

Com o arquivo `.proto`, vamos fazer a mágica do gRPC + protobuf acontecer. Os plugins instalados acima, conseguem gerar tudo o que for necessário para um servidor ou cliente gRPC com o seguinte comando:

```sh
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative proto/ping_pong.proto
```

Esse comando vai gerar dois arquivos: `ping_pong.pb.go` e `ping_pong_grpc.pb.go`. Recomendo dar uma olhada nesses arquivos para entender melhor a estrutura do servidor e do cliente. Com isso, podemos então construir o servidor:

### Construindo o servidor...

Para conseguir comparar com o RPC comum, vamos utilizar a mesma lógica: recebemos `PING` e respondemos `PONG`. Aqui definimos um servidor e um handler para a requisição e usamos as definições vindas do protobuf para a requisição e resposta. Depois, é só iniciar o servidor:

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

### E o cliente...

Para consumir o nosso servidor, precisamos de um cliente. o cliente é bem simples também. A biblioteca do gRPC já implementa basicamente tudo que precisamos, então inicializamos um client e só chamamos o método RPC que queremos usar, no caso o `Ping`. Tudo vem importado do código gerado via plugins do protobuf.

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

Quem tiver interesse para ver o código completo, pode acessar a [PoC gRPC](https://github.com/mfbmina/poc_grpc).

# Considerações finais

O gRPC não é nada mais que uma abstração em cima do RPC convencional utilizando o protobuf como serializador e o protocolo `http/2`. Existem algumas considerações de performance ao se utilizar o `http/2` e em alguns cenários, como em requisições com o corpo simples, o `http/1` se mostra mais performático que o `http/2`. Recomendo a leitura deste [benchmark](https://github.com/duh-rpc/duh-go-benchmarks) e desta issue aberta no [golang/go](https://github.com/golang/go/issues/47840) sobre o `http/2`. Contudo, em requisições de corpo complexo, como grande parte das que resolvemos dia a dia, gRPC se torna uma solução extremamente atraente devido ao serializador do protobuf, que é extremamente mais rápido que JSON. O Elton Minetto fez um [blog post](https://eltonminetto.dev/post/2024-08-05-json-vs-flatbuffers-vs-protobuf/) explicando melhor essas alternativas e realizando um benchmark. Um consideração também é o protobuf consegue resolver o problema de inconsistência de contratos entre servidor e cliente, contudo é necessário uma maneira fácil de distribuir os arquivos `.proto`.

Por fim, minha recomendação é use gRPC se sua equipe tiver a necessidade e a maturidade necessária para tal. Hoje, grande parte das aplicações web não necessitam da performance que gRPC visa propor e nem todos já trabalharam com essa tecnologia, o que pode causar uma menor velocidade e qualidade nas entregas. Como nessa postagem eu citei muitos links, decidi listar todas as referências abaixo:

- [RPC](https://pt.wikipedia.org/wiki/Chamada_de_procedimento_remoto)
- [RPC RFC]((https://datatracker.ietf.org/doc/html/rfc1831))
- [RPC x REST](https://aws.amazon.com/pt/compare/the-difference-between-rpc-and-rest/)
- [PoC RPC](https://github.com/mfbmina/poc_rpc)
- [net/rpc](https://pkg.go.dev/net/rpc)
- [encoding/gob](https://pkg.go.dev/encoding/gob)
- [CNCF - Cloud Native Computing Foundation](https://www.cncf.io/)
- [gRPC](https://grpc.io/)
- [Protocol Buffer](https://protobuf.dev/)
- [PoC gRPC](https://github.com/mfbmina/poc_grpc)
- [http/1 x http/2 x gRPC](https://github.com/duh-rpc/duh-go-benchmarks)
- [http/2 issue](https://github.com/golang/go/issues/47840)
- [JSON x Protobuffers X Flatbuffers](https://eltonminetto.dev/post/2024-08-05-json-vs-flatbuffers-vs-protobuf/)

Espero que vocês tenham gostado do tema e obrigado!
