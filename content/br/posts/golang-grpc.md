+++
title = 'Golang Grpc'
date = 2024-09-25T19:14:38-03:00
draft = true
+++

A primeira vez que ouvi falar sobre RPC foi em uma aula de sistema distruídos, ainda quando estava cursando a graduação em Ciência da Computação. Achei legal, mas na época lembro de não compreender exatamente o porque eu usaria RPC ao invés de usar o padrão REST, por exemplo. Passa o tempo, e vou trabalhar em uma empresa em que parte do sistema legado era utilizando SOAP. Lembro de pensar: "hm, interessante. Parece com RPC, mas traféga XML". Mais tempo depois, ouço pela primeira vez falar sobre GRPC, mas nunca entendi complementamente o que era, o que comia e pra que servia. 

Como meu blog serve muito de documentação pessoal, achei legal documentar aqui o que aprendi sobre, começando sobre o que é RPC e depois indo para o gRPC.

# Vamos lá, o que é RPC?

[RPC](https://pt.wikipedia.org/wiki/Chamada_de_procedimento_remoto) é uma sigla para `Remote Procedure Call` (em Português `Chamada de Procedimento Remoto`). Ou seja, você envia procedimentos/comandos para um servidor remoto. Sendo simples e direto, isso é RPC. Ele funciona da seguinte forma:

![RPC](/img/posts/rpc.png)

O RPC funciona tanto sobre UDP, quanto TCP. Cabe a você ver o que faz sentido para seu caso de uso! Se você não se importa com uma eventual resposta ou até mesmo em perder pacotes, UDP. Caso contrário, use TCP. Para aqueles que gostam de ler as RFCs, pode encontrar o link [aqui]((https://datatracker.ietf.org/doc/html/rfc1831))

## OK, mas como o RPC se difere de uma chamada REST, por exemplo? 

Ambos são maneiras de arquiteturar APIs. Porém, a arquitetura REST possuí principíos muito bem definidos e que devem ser seguidos para se ter uma arquitetura RESTfull. O RPC até possui principios, mas eles são definidos entre cliente e servidor. Para o cliente RPC, é como se ele tivesse chamando um procedimento local. Outro ponto importante é que para o RPC, não importa muito se a conexão é TCP ou UDP. Já para APIs REST, se você quiser seguir o RESTfull, não vai conseguir utilizar UDP.


Para quem quiser saber mais sobre, recomendo este excelente guia da AWS sobre [RPC x REST](https://aws.amazon.com/pt/compare/the-difference-between-rpc-and-rest/)

## E como implementar um servidor RPC com Go?

Temos duas entidades principais, o cliente e o servidor. 

### Vamos começar pelo servidor

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
	fmt.Println("Listening on... ", conn.Addr())
	s := rpc.NewServer()
	s.Register(h)
	s.Accept(conn)
}
```

### Definindo o cliente

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
		fmt.Println("Enter text: ")

		scanner := bufio.NewScanner(os.Stdin)
		scanner.Scan()

		args := Args{Message: scanner.Text()}
		fmt.Println("Sent message: ", args.Message)
		reply := &Args{}
		err = client.Call("Handler.Ping", args, reply)
		if err != nil {
			log.Fatal(err)
		}

		fmt.Println("Received message: ", reply.Message)
		fmt.Println("-------------------------")
	}
}
```

É importante notar que precisamos fornecer o endereço de onde o servidor está rodando e qual o `Handler` (procedimento) que queremos executar. 

Um adendo importante é que como estamos trafegando dados binários e por padrão o Go vai utilizar o [encoding/gob](https://pkg.go.dev/encoding/gob). Se quiser utilizar um outro padrão, como por exemplo `JSON`, vai ser preciso dizer ao seu servidor que aceite aquele codec novo.

Para quem quiser ver o código completo, é só acessar a [PoC.](https://github.com/mfbmina/poc_rpc)

# E o que é gRPC?

gRPC é um framework para se escrever aplicações usando RPC! Esse framework hoje é mantido pela [CNCF](https://www.cncf.io/) e 

# Links utéis

Como nessa postagem eu citei muitos links, seguem aqui todos:

- [RPC](https://pt.wikipedia.org/wiki/Chamada_de_procedimento_remoto)
- [RPC RFC]((https://datatracker.ietf.org/doc/html/rfc1831))
- [RPC x REST](https://aws.amazon.com/pt/compare/the-difference-between-rpc-and-rest/)
- [PoC RPC](https://github.com/mfbmina/poc_rpc)
- [net/rpc](https://pkg.go.dev/net/rpc)
- [encoding/gob](https://pkg.go.dev/encoding/gob)
- [CNCF - Cloud Native Computing Foundation](https://www.cncf.io/)
-
