+++
title = 'Escrevendo plugins para o Kong em Go'
date = 2023-10-03T12:55:19-03:00
+++

O Kong, segundo a própia [documentação](https://github.com/Kong/kong), é um apigateway open source, criado para a nuvem, agnóstico em plataforma, feito para alta performance e extensível via plugins. O Kong possui uma grande gama de [plugins oficiais](https://docs.konghq.com/konnect/reference/plugins/) que nos permitem fazer grande parte das customizações que necessitamos e quando o plugin não está disponível, podemos criar nosso própio plugin.

A linguagem padrão para se criar plugins é Lua, contudo outras linguagens são suportadas e uma delas é Go. Pessoalmente, não tenho experiência em escrever códigos em Lua, portanto, utilizar Go me permite uma maior velocidade de desenvolvimento e maior qualidade no plugin.

## Desenvolvimento

Criar um plugin usando Go é bem simples, é necessário seguir a assinatura proposta pelo [plugin development kit](https://docs.konghq.com/gateway/latest/plugin-development/), ou PDK, do Kong, mas ela é de fácil entendimento. Para demonstrar isso, vamos criar um plugin que vai ler um header da requisição e a configuração do plugin. Com essas duas informações, vamos adicionar um novo header na resposta.

A primeira coisa a ser feita é definir as constantes `version` e `priority`. Como o própio nome diz, isso define a versão do plugin que está sendo desenvolvido e a prioridade em execução a relação a outros plugins. Vale a pena pontuar que quanto maior o valor da prioridade, mais cedo o plugin vai ser executado. Para ver mais sobre a prioridades dos plugins, este [post](https://docs.konghq.com/konnect/reference/plugins/) explica bem.

```golang
var Version = "0.0.1"
var Priority = 1
```

Com isso realizado, vamos definir quais os parametros que o plugin aceita, criando uma `struct` chamada `Config`. No nosso exemplo, aceitamos o campo `message`, que é uma string.

```golang
type Config struct {
	Message string `json:"message"`
}
```

Os plugins do Kong funcionam baseados em quais fases do ciclo de vida da requisição eles devem ser executados. As fases possíveis são:
- Certificate
- Rewrite
- Access
- Response
- Preread
- Log

No nosso exemplo, vamos utilizar o `Access`, pois vamos atualizar a resposta antes de encaminharmos a requisição para o serviço responsável. Contudo, a assinatura de todas as fases são iguais. Para entender mais sobre as fases, recomendo ler essa [documentação](https://docs.konghq.com/gateway/latest/plugin-development/custom-logic/).

```golang
func (conf Config) Access(kong *pdk.PDK) {
	host, err := kong.Request.GetHeader("host")
	if err != nil {
		log.Printf("Error reading 'host' header: %s", err.Error())
	}

	message := conf.Message
	if message == "" {
		message = "hello"
	}
	kong.Response.SetHeader("x-hello-from-go", fmt.Sprintf("Go says %s to %s", message, host))
}
```

Essa função utiliza o PDK para ler os headers disponíveis, através de `kong.Request.GetHeader` e depois adiciona um header na resposta utilizando o `kong.Response.SetHeader`. Para mais informações dos funções dísponíveis, dê uma olhada na documentação da [Go PDK](https://pkg.go.dev/github.com/Kong/go-pdk).

Por fim, vamos a função principal:

```golang
package main

import (
	"fmt"
	"log"

	"github.com/Kong/go-pdk"
	"github.com/Kong/go-pdk/server"
)

func main() {
	server.StartServer(New, Version, Priority)
}

func New() interface{} {
	return &Config{}
}
```

Como podemos notar, a funcão principal só inicializa o servidor do Kong, passando a `Config`, `Version` e `Priority` definidas anteriormente. A implementação completa fica assim:

```golang
package main

import (
	"fmt"
	"log"

	"github.com/Kong/go-pdk"
	"github.com/Kong/go-pdk/server"
)

var Version = "0.0.1"
var Priority = 1

func main() {
	server.StartServer(New, Version, Priority)
}

type Config struct {
	Message string `json:"message"`
}

func New() interface{} {
	return &Config{}
}

func (conf Config) Access(kong *pdk.PDK) {
	host, err := kong.Request.GetHeader("host")
	if err != nil {
		log.Printf("Error reading 'host' header: %s", err.Error())
	}

	message := conf.Message
	if message == "" {
		message = "hello"
	}
	kong.Response.SetHeader("x-hello-from-go", fmt.Sprintf("Go says %s to %s", message, host))
}
```

## Testes

Testar os plugins em Go também é bem simples, uma vez que o própio PDK nos oferece ferramentas que nos permitem fazer isso de forma direta. Então vamos testar nosso plugin:

```golang
package main

import (
	"testing"

	"github.com/Kong/go-pdk/test"
	"github.com/stretchr/testify/assert"
)

func TestPluginWithoutConfig(t *testing.T) {
	env, err := test.New(t, test.Request{
		Method:  "GET",
		Url:     "http://example.com?q=search&x=9",
		Headers: map[string][]string{"host": {"localhost"}},
	})
	assert.NoError(t, err)

	env.DoHttps(&Config{})
	assert.Equal(t, 200, env.ClientRes.Status)
	assert.Equal(t, "Go says hello to localhost", env.ClientRes.Headers.Get("x-hello-from-go"))
}

func TestPluginWithConfig(t *testing.T) {
	env, err := test.New(t, test.Request{
		Method:  "GET",
		Url:     "http://example.com?q=search&x=9",
		Headers: map[string][]string{"host": {"localhost"}},
	})
	assert.NoError(t, err)

	env.DoHttps(&Config{Message: "nice to meet you"})
	assert.Equal(t, 200, env.ClientRes.Status)
	assert.Equal(t, "Go says nice to meet you to localhost", env.ClientRes.Headers.Get("x-hello-from-go"))
}
```

## Deploy

Para o deploy, precisamos gerar um executável e colocar isso na imagem que vamos utilizar do Kong.

```Dockerfile
# Build Golang plugins

FROM golang:1.20 AS plugin-builder

WORKDIR /builder

COPY ./hello ./go_plugins/hello

RUN find ./go_plugins -maxdepth 1 -mindepth 1 -type d -not -path "*/.git*" | \
    while read dir; do \
        cd $dir && go build -o /builds/$dir main.go  ; \
    done

# Build Kong
FROM kong:3.4.0-ubuntu

COPY --from=plugin-builder ./builds/go_plugins/  ./kong/

USER kong
```

Para o Kong conseguir encontrar e carregar o plugin customizado é necessário especificar algumas configurações. Seguindo nosso exemplo, alterei o `docker-compose` para incluir:

```yaml
KONG_PLUGINS: "bundled,hello"
KONG_PLUGINSERVER_NAMES: hello
KONG_PLUGINSERVER_HELLO_START_CMD: /kong/hello
KONG_PLUGINSERVER_HELLO_QUERY_CMD: /kong/hello -dump
```

## Considerações sobre performance

A própia Kong fez um estudo sobre a performance, que pode ser encontrado [aqui](https://docs.konghq.com/gateway/latest/plugin-development/pluginserver/performance/).

## Conclusão

Go é uma ótima ferramenta para se escrever plugins para o Kong. Podemos contar com toda a facilidade e performance da linguagem, além de todas as ferramentas que a linguagem oferece. No meu caso, consegui ganhar velocidade na entrega e uma maior cobertura de testes. Também é possível isolar a lógica dos plugins com a utilização da PDK, facilitando assim escrever plugins que funcionam pra diversas tecnologia, desacoplando do Kong.

Para ver todo o contéudo apresentado nesse post, é só acessar o [repositório](https://github.com/mfbmina/poc-goplugin-kong).

Você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**

## Referências

- [Kong](https://github.com/Kong/kong)
- [Kong plugins](https://docs.konghq.com/gateway/latest/kong-plugins/queue/reference/)
- [Kong PDK](https://docs.konghq.com/gateway/latest/plugin-development/)
- [Kong Go plugins](https://docs.konghq.com/gateway/latest/plugin-development/pluginserver/go/)
- [Kong Go PDK](https://pkg.go.dev/github.com/Kong/go-pdk)
- [Kong Phase Access](https://docs.konghq.com/gateway/latest/plugin-development/custom-logic/)
- [Exemplos de plugins em Go](https://github.com/Kong/go-plugins)
- [Repositório com os códigos do post](https://github.com/mfbmina/poc-goplugin-kong)
