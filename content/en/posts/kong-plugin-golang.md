+++
title = 'Writing Kong plugins with Go'
date = 2023-10-03T12:55:19-03:00
tags = ["go", "apigateway", "kong"]
+++

The Kong, quoting its own [documentation](https://github.com/Kong/kong), is an open-source API gateway, cloud-native, platform-agnostic, scalable API Gateway distinguished for its high performance and extensibility via plugins. Kong has a lot of [official plugins](https://docs.konghq.com/konnect/reference/plugins/) that allow us to customize what we need, and when there aren't any available, you can build your own.

The default language for building plugins its Lua, but other languages are supported, including Go. I don't have much experience writing code with Lua, so using Go allows me a higher development speed and quality on my plugins.

## Development

Building a Go plugin is easy, as you need to follow the signature proposed by Kong's [plugin development kit](https://docs.konghq.com/gateway/latest/plugin-development/) or PDK, but it is easy to understand. To demonstrate that, we will build a plugin that will read a request header and set a new response header.

The first thing to do is define both constants: `version` and `priority`. As the name says, we will configure the current version of the plugin and its execution priority within the other plugins. Its worth mentioning that the higher the priority, the earlier it will be executed. If you wanna understand more about it, check this [post](https://docs.konghq.com/konnect/reference/plugins/).

```golang
var Version = "0.0.1"
var Priority = 1
```

After, we will define which params the plugin accepts by building a `struct` called `Config`. Here, we will receive the field message as a string.

```golang
type Config struct {
	Message string `json:"message"`
}
```

All kong plugins work based on which phase access they need to run. The possible phase access are:
- Certificate
- Rewrite
- Access
- Response
- Preread
- Log

In this example, we will be using `Access` because we need to update the response before forwarding the request to the responsible microservice. You don't need to worry about it because the signatures for all phases are the same. To know more about all phase access, check this [documentation](https://docs.konghq.com/gateway/latest/plugin-development/custom-logic/).

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

This function uses the PDK to read the available headers by using `kong.Request.GetHeader`, and later we add a response header using `kong.Response.SetHeader`. For more information about all available functions, you can check the [Go PDK](https://pkg.go.dev/github.com/Kong/go-pdk).

At last, let's check the main function:

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

As we can notice, the main function initializes the Kong server, sending the `Config`, `Version`, and `Priority` defined before. The full code is:

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

## Tests

Testing our Go plugins is also easy once the PDK gives us tools that make it straightforward. Let's test our plugin:

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

To deploy our plugin, we build an executable and must add it to the Kong image.

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

Kong needs some information to be able to find and load the customized plugin, so we need to add some environment variables:

```yaml
KONG_PLUGINS: "bundled,hello"
KONG_PLUGINSERVER_NAMES: hello
KONG_PLUGINSERVER_HELLO_START_CMD: /kong/hello
KONG_PLUGINSERVER_HELLO_QUERY_CMD: /kong/hello -dump
```

## Considerations about performance

The Kong company already did a study about the performance, and you can find it [here](https://docs.konghq.com/gateway/latest/plugin-development/pluginserver/performance/).

## Conclusion

Go is an excellent tool for writing plugins for Kong. We can have all the performance and tools it provides to us. In my case, I have also been able to speed up my deliveries and have a higher code coverage. It is also possible to isolate the plugin's logic with the PDK use, making it easy to write plugins that work for different technologies, decoupling Kong.

If you wanna check all the code shown in this post, you can access the [repo](https://github.com/mfbmina/poc-goplugin-kong).

You also can find me on **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)**, or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**

## References

- [Kong](https://github.com/Kong/kong)
- [Kong plugins](https://docs.konghq.com/gateway/latest/kong-plugins/queue/reference/)
- [Kong PDK](https://docs.konghq.com/gateway/latest/plugin-development/)
- [Kong Go plugins](https://docs.konghq.com/gateway/latest/plugin-development/pluginserver/go/)
- [Kong Go PDK](https://pkg.go.dev/github.com/Kong/go-pdk)
- [Kong Phase Access](https://docs.konghq.com/gateway/latest/plugin-development/custom-logic/)
- [Exemplos de plugins em Go](https://github.com/Kong/go-plugins)
- [Repositório com os códigos do post](https://github.com/mfbmina/poc-goplugin-kong)
