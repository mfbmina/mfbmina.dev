+++
title = 'Testando chamadas para APIs da melhor forma'
date = 2025-02-05T16:35:31-03:00
draft = true
tags = ["go", "api", "testing"]
+++

Atualmente, grande parte do trabalho de um desenvolvedor WEB consiste em chamar APIs, seja para realizar uma integração com um sistema de uma equipe parceira ou para integrar com algum fornecedor.

Outra grande frente do dia-a-dia é a escrita de testes. Testes garantem (ou deveriam garantir :D) que o código escrito por nós funciona da maneira esperada e que surpresas não vão acontecer quando a funcionalidade estiver em ambiente produtivo.

Desta forma, é natural pensar que saber escrever testes para chamadas a alguma API é essencial para uma pessoa engenheira de software competente. Nessa postagem, quero mostrar algumas técnicas que vão facilitar a escrita de testes de suas funcionalidades!

Então, o primeiro passo é construir o serviço que irá ser testado. Esse serviço vai ser extremamente simples: vamos chamar uma API da PokéDex (estou no hype do Pokémon TCG Pocket) e listar os Pokémons existentes.

```golang
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type RespBody struct {
	Results []Pokemon `json:"results"`
}

type Pokemon struct {
	Name string `json:"name"`
}

const URL = "https://pokeapi.co"

func main() {
	pkmns, err := FetchPokemon(URL)
	if err != nil {
		fmt.Println(err)
		return
	}

	for _, pkmn := range pkmns {
		fmt.Println(pkmn.Name)
	}
}

func FetchPokemon(u string) ([]Pokemon, error) {
	r, err := http.Get(fmt.Sprintf("%s/api/v2/pokemon", u))
	if err != nil {
		return nil, err
	}

	defer r.Body.Close()
	resp := RespBody{}
	err = json.NewDecoder(r.Body).Decode(&resp)
	if err != nil {
		return nil, err
	}

	return resp.Results, nil
}
```

## httptest
O [httptest](https://pkg.go.dev/net/http/httptest) é um pacote da própria linguagem.  Ele permite a criação de `mock servers` para serem usados nos nossos testes. A principal vantagem é que não adicionamos nenhuma dependência extra ao projeto. Contudo, ele não intercepta as requisições automaticamente.

```golang
package main

import (
	"encoding/json"
	"net/http"
	"net/http/httptest"
	"testing"

	"github.com/stretchr/testify/assert"
)

func Test_HTTPtest(t *testing.T) {
	j, err := json.Marshal(RespBody{Results: []Pokemon{{Name: "Charizard"}}})
	assert.Nil(t, err)

	server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.URL.Path != "/api/v2/pokemon" {
			t.Errorf("Expected to request '/api/v2/pokemon', got: %s", r.URL.Path)
		}
		w.WriteHeader(http.StatusOK)
		w.Write([]byte(j))
	}))
	defer server.Close()

	p, err := FetchPokemon(server.URL)
	assert.Nil(t, err)
	assert.Equal(t, p[0].Name, "Charizard")
}
```

## mocha
Já o [mocha](https://github.com/vitorsalgado/mocha) é uma biblioteca inspirada no [nock](https://github.com/nock/nock) e no [WireMock](https://wiremock.org/). Uma característica interessante é que é possível verificar se de fato o `mock` foi chamado ou não. Assim como o `httptest`, ele também não intercepta as requisições automaticamente.

```golang
package main

import (
	"encoding/json"
	"fmt"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/vitorsalgado/mocha/v3"
	"github.com/vitorsalgado/mocha/v3/expect"
	"github.com/vitorsalgado/mocha/v3/reply"
)

func Test_Mocha(t *testing.T) {
	j, err := json.Marshal(RespBody{Results: []Pokemon{{Name: "Charizard"}}})
	assert.Nil(t, err)

	m := mocha.New(t)
	m.Start()

	scoped := m.AddMocks(mocha.Get(expect.URLPath("/api/v2/pokemon")).
		Reply(reply.OK().BodyString(string(j))))

	p, err := FetchPokemon(m.URL())
	fmt.Println(m.URL())
	assert.Nil(t, err)
	assert.True(t, scoped.Called())
	assert.Equal(t, p[0].Name, "Charizard")
}
```

## gock
Outra opção legal é o [gock](https://github.com/h2non/gock), uma biblioteca também inspirada no `nock`, que possui uma API simples e fácil. Ela funciona interceptando qualquer requisição HTTP feita por qualquer `http.Client` e colocando numa lista para verificar se existe algum `mock`. Se não existir, um erro é retornado, porém, caso o modo `real networking` esteja ativo, a requisição é realizada normalmente. 
 
```golang
package main

import (
	"encoding/json"
	"testing"

	"github.com/h2non/gock"
	"github.com/stretchr/testify/assert"
)

func Test_Gock(t *testing.T) {
	defer gock.Off()

	j, err := json.Marshal(RespBody{Results: []Pokemon{{Name: "Charizard"}}})
	assert.Nil(t, err)

	gock.New("https://pokeapi.co").
		Get("/api/v2/pokemon").
		Reply(200).
		JSON(j)

	p, err := FetchPokemon(URL)
	assert.Nil(t, err)
	assert.Equal(t, p[0].Name, "Charizard")
}
```

## apitest
Por fim, o [apitest](https://github.com/steinfletcher/apitest) é uma biblioteca inspirada pelo `gock` e possui uma infinidade de `matchers` e funcionalidades. Ele inclusive permite ao usuário gerar um diagrama de sequência das chamadas HTTP. Uma coisa bacana é que ele também possui um excelente [website](https://apitest.dev/) com exemplos de uso.

```golang
package main

import (
	"encoding/json"
	"net/http"
	"testing"

	"github.com/steinfletcher/apitest"
	"github.com/stretchr/testify/assert"
)

func Test_APItest(t *testing.T) {
	j, err := json.Marshal(RespBody{Results: []Pokemon{{Name: "Charizard"}}})
	assert.Nil(t, err)

	defer apitest.NewMock().
		Get("https://pokeapi.co/api/v2/pokemon").
		RespondWith().
		Body(string(j)).
		Status(http.StatusOK).
		EndStandalone()()

	p, err := FetchPokemon(URL)
	assert.Nil(t, err)
	assert.Equal(t, p[0].Name, "Charizard")
}
```

O [Minetto](https://eltonminetto.dev) tem um ótimo [post](https://eltonminetto.dev/post/2020-04-10-golang-apitest/) sobre essa biblioteca! Vale a pena conferir!

## Conclusão
Na minha opinião, não existe uma técnica melhor que a outra, mas sim o que você prefere. Se não quer ter nenhuma dependência extra no seu projeto e não liga de ter que escrever os `matchers` manualmente, escolha o `httptest` e seja feliz.

Se ter mais dependências não for um problema, veja outros critérios! Uma API mais rica? Uma dependência mais ou menos completa? Veja o que faz sentido no cenário seu e da sua equipe. Pessoalmente, curto bastante o `apitest` e incentivo o seu uso aqui na equipe, pois na minha opinião é o mais completo de todos.

Se quiser ver o exemplo completo, é só acessar este [link!](https://github.com/mfbmina/golang_api_testing)
