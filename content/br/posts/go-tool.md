+++
title = 'Go Tool: tudo o que ninguem pediu'
date = 2025-04-03T20:02:01-03:00
draft = false
tags = ["go", "1.24", "tools", "news", "opinion"]
+++

Depois de muitos anos trabalhando com Ruby, migrei para trabalhar com Go sem muita experiência com a linguagem. Meu primeiro atrito foi com a gestão de dependências, pois sempre achei a versão de dependências de Go ruim, com os comandos confusos e, o pior, sem distinção entre dependências de desenvolvimento e dependências produtivas, pois ambas são incluídas no binário final. Vamos olhar o exemplo do `go.mod` de uma PoC:

```go.mod
module github.com/mfbmina/poc_circuit_breaker

go 1.24

require github.com/sony/gobreaker/v2 v2.0.0 // indirect
```

Mas o tempo passou, os atritos foram superados e acabei entendendo como essa gestão funcionava e aqui estou trabalhando com Go até hoje. Quando li que a versão 1.24 ia trazer a funcionalidade de "gestão de ferramentas" fiquei super animado, e geralmente não sou o tipo de pessoa que fica acompanhando as novidades da próxima versão de alguma linguagem, mas confesso que após testar a funcionalidade fiquei bem desapontado.

O `go tools`, como o próprio nome diz, é uma funcionalidade para você adicionar as ferramentas que seu projeto usa, como, por exemplo, goreleaser, golangci-lint, etc. Para instalar uma nova ferramenta, é só rodar o comando `$ go get -tool github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.0.2`. Uma vez instalada, você a utiliza pela CLI do Go: `$ go tool github.com/golangci/golangci-lint/v2/cmd/golangci-lint run`. Para listar todas as ferramentas instaladas, rode `$ go tool`.

```go.mod
module github.com/mfbmina/poc_circuit_breaker

go 1.24

require github.com/sony/gobreaker/v2 v2.0.0

// omiting a lot of indirect dependencies...

tool github.com/golangci/golangci-lint/v2/cmd/golangci-lint
```

A primeira vista parece excelente, já que você pode ter as ferramentas no seu arquivo de dependências, mas o ponto principal ainda não foi resolvido. Ele até mesmo piora dependendo do seu ponto de vista, pois além de incluir todas as dependências de produção e desenvolvimento num mesmo arquivo `go.mod`, ele ainda traz as ferramentas utilizadas no desenvolvimento, sem a possibilidade de só instalar alguns deles. Inclusive, ferramentas como o [golangci-lint não recomendam](https://golangci-lint.run/welcome/install/#install-from-sources) esse tipo de instalação, justamente para isolar essa ferramenta das dependências reais. Existem algumas estratégias para se utilizar o `go tools`, mas confesso que não testei por simplesmente ter perdido o interesse na funcionalidade.

O que eu esperava era algo na linha de se poder criar grupos de dependências, assim como aplicações Ruby, JavaScript, PHP, etc. Como exemplo, vamos olhar um Gemfile, um arquivo de dependências Ruby:

```Gemfile
# copied from https://github.com/sidekiq/sidekiq/blob/main/Gemfile
source "https://rubygems.org"

gemspec

gem "rake"
RAILS_VERSION = "~> 8.0"
gem "actionmailer", RAILS_VERSION
gem "actionpack", RAILS_VERSION
gem "activejob", RAILS_VERSION
gem "activerecord", RAILS_VERSION
gem "railties", RAILS_VERSION
gem "redis-client"
# gem "bumbler"
# gem "debug"

gem "sqlite3", "~> 2.2", platforms: :ruby
gem "activerecord-jdbcsqlite3-adapter", platforms: :jruby
gem "after_commit_everywhere", require: false
gem "yard"
gem "csv"
gem "vernier" unless RUBY_VERSION < "3"
gem "webrick"

group :test do
  gem "maxitest"
  gem "simplecov"
  gem "debug"
end

group :development, :test do
  gem "standard", require: false
end

group :load_test do
  gem "toxiproxy"
  gem "ruby-prof"
end
```

Dessa maneira, podemos instalar somente as dependências dos grupos que queremos. Essa funcionalidade foi tudo o que ninguém pediu, justamente por trazer algo para solucionar um problema que não era real, ou pelo menos que não incomodava tanto. Ou talvez eu tivesse expectativas erradas sobre a funcionalidade...

De qualquer forma, ainda sigo na expectativa de ter meus binários em Go sem minhas dependências de teste e desenvolvimento. Quem sabe isso não venha nas próximas versões.
