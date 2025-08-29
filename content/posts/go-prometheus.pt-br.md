+++
title = 'Métricas com Go e Prometheus'
date = 2024-12-27T19:07:46-03:00
draft = false
+++

No mundo do desenvolvimento, é necessário saber como a aplicação que estamos trabalhando está se comportando e a maneira mais conhecida de realizarmos isso é por meio de métricas.  Elas podem ser de diversos tipos, como, por exemplo, de desempenho, de produto ou de saúde. Atualmente, o [Prometheus](https://www.cncf.io/projects/prometheus/) é amplamente utilizado pelo mercado a fim de coletar essas métricas.


## Como funciona?
Ele é um serviço open-source mantido pela [CNCF](https://www.cncf.io/), a Cloud Native Computing Foundation. Ele funciona da seguinte maneira: um endpoint é exposto na aplicação. Esse endpoint retorna um texto no formato esperado, e o Prometheus acessa esse endpoint de tempos em tempos coletando as informações dali. 

```
# HELP failure_rate The total number of failed events
# TYPE failure_rate counter
failure_rate 3280
```

O formato é bem simples, para cada métrica, você vai ter pelo menos três entradas:
- O #HELP mostra a descrição da métrica. 
- O #TYPE define o tipo da métrica.
- A terceira linha mostra o valor da métrica.

## Como utilizar em Go?
Em aplicações escritas com Go, temos uma biblioteca que facilita ainda mais a exposição dessas métricas. Ela implementa um handler que expõe por padrão as principais métricas relacionadas ao software, como, por exemplo, goroutines, memória, heap, etc. O exemplo abaixo demonstra melhor como se usa:

```golang
package main

import (
  "fmt"
  "net/http"

  "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
  fmt.Println("Monitoring service...")
  http.Handle("/metrics", promhttp.Handler())
  http.ListenAndServe(":8080", nil)
}
```

Pode-se notar que com poucas linhas temos o endpoint de métricas e um servidor web funcionando! Agora, como configurar um Prometheus para coletá-las? O primeiro passo é subir ambas as aplicações, para isso o melhor é utilizar o `docker compose`. 

```yaml
services:
  myapp:
    image: myapp
    build: .
    container_name: myapp
    ports:
      - 8080:8080
    restart: unless-stopped
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yaml'
    ports:
      - 9090:9090
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prom_data:/prometheus
volumes:
  prom_data:
```

No arquivo compose, é definida que a aplicação vai escutar na porta `8080` e o Prometheus na porta `9090`. Ele também espera um arquivo de configuração. Este arquivo define onde e com qual frequência ele deve fazer a varredura.

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
scrape_configs:
- job_name: myapp
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - 'myapp:8080'
```

Também definimos o Dockerfile da nossa aplicação falsa:

```yaml
# copied from the internet
# syntax=docker/dockerfile:1

FROM golang:1.23

# Set destination for COPY
WORKDIR /app

# Download Go modules
COPY go.mod go.sum ./
RUN go mod download

# Copy the source code. Note the slash at the end, as explained in
# https://docs.docker.com/reference/dockerfile/#copy
COPY ./ ./

# Build
RUN CGO_ENABLED=0 GOOS=linux go build -o /myapp

# Optional:
# To bind to a TCP port, runtime parameters must be supplied to the docker command.
# But we can document in the Dockerfile what ports
# the application is going to listen on by default.
# https://docs.docker.com/reference/dockerfile/#expose
EXPOSE 8080

# Run
CMD ["/myapp"]
```

Se quiser verificar se tudo está funcionando, é só acessar `http://localhost:9090` e ver se deu certo a configuração. 

## Métricas customizadas
Contudo, muitas vezes as métricas padrões não são suficientes para representar o comportamento da nossa aplicação e é necessário definir métricas customizadas. Ao utilizar a biblioteca padrão do Prometheus para Go, essa tarefa se torna trivial e simples, como o exemplo abaixo:

```golang
successRate := promauto.NewCounter(prometheus.CounterOpts{
  Name: "success_rate",
  Help: "The total number of succeded events",
})

successRate.Inc()
```

Desta forma, uma nova métrica será retornada ao acessar o endpoint `/metrics`. Porém, nem sempre é possível ter um servidor web para ter as métricas expostas dessa forma. 

## Enviando métricas de forma ativa
A partir dessa premissa, foi desenvolvido o [Pushgateway](https://prometheus.io/docs/instrumenting/pushing/). Ele funciona da seguinte forma: você envia suas métricas por chamadas HTTP e ele armazena e expõe o endpoint `/metrics` para a coleta do Prometheus. Todavia, nem sempre é uma boa ideia utilizar esta estratégia, pois, segundo a própria documentação oficial:
- Quando se monitora múltiplas instâncias por meio de um único Pushgateway, ele se torna um ponto único de falha e um potencial gargalo.
- Você perde o monitoramento automático da saúde da sua aplicação, gerada em cada varredura.
- O Pushgateway nunca esquece nenhum dado que foi enviado para ele e vai sempre os expor para o Prometheus, exceto caso seja manualmente deletado.

Para mais informações, recomendo a leitura dessa [documentação](https://prometheus.io/docs/practices/pushing/). Mas, como existem casos de uso, vamos ver como podemos usar a biblioteca do Prometheus para fazer um Push. O exemplo abaixo demonstra que muda muito pouco do exemplo anterior. Só adicionamos a instrução referente ao Push e para onde ele deve ser realizado.

```golang
successRate := promauto.NewCounter(prometheus.CounterOpts{
  Name: "success_rate_pg",
  Help: "The total number of succeded events",
})

p.successRate.Inc()

err := push.New("http://pushgateway:9091", "pg").
  Collector(p.successRate).
  Grouping("myapp", "success_rate_pg").
  Push()

if err != nil {
  fmt.Println("Could not push completion time to Pushgateway:", err)
}
```

Também é necessário configurar o compose para iniciar o Pushgateway.

```yaml
services:
  # ... myapp and prometheus already exposed
  pushgateway:
    image: prom/pushgateway
    container_name: pushgateway
    restart: unless-stopped
    expose:
      - 9091
    ports:
      - "9091:9091"
```

E por fim, vamos dizer ao Prometheus que ele deve varrer o Pushgateway também.

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
scrape_configs:
- job_name: pushgateway
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - 'pushgateway:9091'
```

## Conclusão
Vale a pena mencionar que, para o exemplo, eu utilizei somente o tipo de métrica Counter, ou em português Contador. Contudo, existem diversos outros tipos que podem ser encontrados [aqui](https://prometheus.io/docs/concepts/metric_types/)! Para mais exemplos utilizando o Prometheus com Go, você pode checar os [exemplos oficiais](https://github.com/prometheus/client_golang/tree/main/examples) da biblioteca ou a [PoC](https://github.com/mfbmina/poc-prometheus-exporter) onde implemento um simulador de monitoramento sintético e gero métricas através do endpoint e do Pushgateway.

Se você gostou do post, me diz: quais métricas você geralmente monitora na sua aplicação? Elas são métricas padrões ou customizadas?
