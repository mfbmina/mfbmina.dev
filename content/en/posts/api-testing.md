+++
title = 'The best way for testing outbound API calls'
date = 2025-02-05T16:35:31-03:00
draft = false
tags = ["go", "api", "testing"]
+++

Nowadays, a huge part of a developer's work consists in calling APIs, sometimes to integrate with a team within the company, sometimes to build an integration with a supplier.

The other big role in daily work is to write tests. Tests ensure (or should guarantee :D) that all the code written by us works on how it is expected and, therefore, it will not happen any surprises when the feature is running at production environment.

Hence, it is natural to think that writing tests for outbound API calls is essential for a capable software engineer. At this post, I want to share some techniques that will ease the writing of your tests! 

So, the first step is to build the service that will be tested. It will be really simple: we will call a Pokédex API (I'm at the Pokémon TCG Pocket hype) and list all existing Pokémon.

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
The [httptest](https://pkg.go.dev/net/http/httptest)
is a package from Go. It allows the creation of mock servers that can be used in the tests. Its main advantage is that we don't add any extra dependency at the project. However, it don't automatically intercept the requests.

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

## mocha
[mocha](https://github.com/vitorsalgado/mocha) is a lib inspired by [nock](https://github.com/nock/nock) and [WireMock](https://wiremock.org/). It allows checking if the mock was called or not, which is a nice feature. Like `httptest`, it also it don't automatically intercept the requests.

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
Another great option is [gock](https://github.com/h2non/gock), a lib that is also inspired by `nock`, with a simple and easy API. It works by intercepting any HTTP request made by any `http.Client` and adding it to a list, so it can check if a mock request exists. If not, an error is returned, unless the real networking mode is on. In that case, the request is done normally.

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
At last, the [apitest](https://github.com/steinfletcher/apitest) is a lib inspired by `gock` that has an infinity matchers and features. It also allows the user to build a sequence diagram of its HTTP calls. A cool thing is their excellent [website](https://apitest.dev/) with a lot of examples.

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

[Minetto](https://eltonminetto.dev/en) has a great [post](https://eltonminetto.dev/en/post/2020-04-21-golang-apitest/) about this lib! It is worth checking!

## Conclusion
At my opinion, one method is not better than the other. It depends on what's work better for you and your team. If having an extra dependency is a no-go, and you don't mind writing the matchers manually, choose `httptest` and be happy!

If having them is not an issue, check for other criteria. You wish a richer API? A more complete dependency or a smaller one? Choose what makes sense in your scenario. Personally, I like `apitest` the most and I advocate for its use inside the team, because I think it is the most complete one.

If you want to check the whole example, please access this [link!](https://github.com/mfbmina/golang_api_testing)
