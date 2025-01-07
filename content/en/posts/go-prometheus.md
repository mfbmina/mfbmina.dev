+++
title = 'Metrics with Go and Prometheus'
date = 2024-12-27T19:07:46-03:00
draft = false
tags = ["go", "monitoring", "metrics", "prometheus"]
+++

At the developing world, it is necessary to know how the application that you're working on is behaving, and the most known way of doing that is by metrics. They can be of several types, such as performance, product, or health. Nowadays, [Prometheus](https://www.cncf.io/projects/prometheus/) is the market way for collecting metrics.

It is an open-source service maintained by [CNCF](https://www.cncf.io/), the Cloud Native Computing Foundation. It works like the following: an endpoint is exposed, and it responds with a desired body format. Then Prometheus calls this endpoint time to time, collecting all the information from there. 

```
# HELP failure_rate The total number of failed events
# TYPE failure_rate counter
failure_rate 3280
```

This format is pretty simple. For each metric, there are three entries:
- #HELP shows its description.
- #TYPE defines its type.
- The third value has its value.

In Go apps, there is a lib that facilitates even more the metrics exposition. It implements a handler and this handler exposes by default the main metrics related to the software, like memory, heap or even information about goroutines. The example below describes better how to use it:

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

It is noticeable that with a few lines we have a running web server with an endpoint for metrics! Now, how to configure Prometheus to collect them? The first step is to run both apps, and for that we can use `docker compose`.

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

At compose file, it is defined that the app will listen at `8080` port and Prometheus will listen`9090`. It also expects for a configuration file that defines where and how frequently it should scrap.

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

A Dockerfile for the fake app must be created too:

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


If you want to ensure that everything is fine, you can access `http://localhost:9090` and check if the configuration worked. However, most times the default metrics are not enough to understand how our app is behaving and then must define custom metrics. When using the Prometheus standard lib for Go, this task turns out to be simple, as shown below:

```golang
successRate := promauto.NewCounter(prometheus.CounterOpts{
  Name: "success_rate",
  Help: "The total number of succeded events",
})

successRate.Inc()
```

That way, a new metric will be returned when accessing the endpoint `/metrics`. But, it is not always possible to have a web server to expose metrics. From that premise, the  [Pushgateway](https://prometheus.io/docs/instrumenting/pushing/) was developed. It works like the following: you send your metrics to it using HTTP calls, and it stores all data. Eventually a Prometheus instance will scrap its information. Anyway, it is not always a good idea to use this strategy. Quoting the official documentation:
- When monitoring multiple instances through a single Pushgateway, the Pushgateway becomes both a single point of failure and a potential bottleneck.
- You lose Prometheus's automatic instance health monitoring via the "up" metric (generated on every scrape).
- The Pushgateway never forgets series pushed to it and will expose them to Prometheus forever unless those series are manually deleted via the Pushgateway's API.

For more information, I recommend read this [doc](https://prometheus.io/docs/practices/pushing/). Still, how there are use cases, let's see how to use the Prometheus lib for pushing metrics. The example below demonstrates that it almost do not change from the previous example. It is just necessary to add the instruction for where the push should be made.

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

It is also important to configure the Docker compose to start the Pushgateway.

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

At last, let's tell Prometheus that it should collect metrics from  Pushgateway too.

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

It's worth mentioning that, for the example, I only used the type counter. Therefore, there are other types that can be found [here](https://prometheus.io/docs/concepts/metric_types/)! For more examples on how to use Prometheus with Go, you can check the [official examples](https://github.com/prometheus/client_golang/tree/main/examples) or the [PoC](https://github.com/mfbmina/poc-prometheus-exporter) where I build a synthetic monitor simulator and build metrics for the default endpoint and for Pushgateway.

If you liked the post, tell me: which metrics you usually monitor at your application? Are they  custom or default metrics?
