+++
title = 'testcontainers: melhorando testes E2E'
date = 2025-07-16T15:26:36-03:00
draft = false
+++

Em um [post anterior]({{< relref api-testing >}}) mostrei algumas formas de melhorar os testes criando mocks das APIs que chamamos. Porém, isso nem sempre é suficiente e podemos precisar de testes E2E (ou testes de aceitação), como, por exemplo, para testar integrações com banco de dados, serviços de mensageria ou qualquer outra coisa. Para esses casos, venho apresentar a ferramenta Testcontainers.

O [testcontainer](https://testcontainers.com/) é uma biblioteca open-source que permite que você crie containers durante a execução dos testes. A ideia é que as dependências de teste da sua aplicação sejam parte do código, evitando a necessidade de mocks e até mesmo a instalação local de dependências. Outra grande vantagem é que dessa forma é mais fácil conseguirmos isolamento entre os testes e replicabilidade entre os desenvolvedores. A ferramenta suporta diversas linguagens de programação como Go, Ruby, Elixir, Java, etc.

## Instalação
Para começar, é necessário ter o [Docker](https://www.docker.com/) instalado em sua máquina ou algum substituto como o [Rancher](https://www.rancher.com/) ou [Colima](https://github.com/abiosoft/colima). Dependendo da sua instalação, será necessário configurar alguns passos adicionais na sua máquina que podem ser encontrados [aqui](https://golang.testcontainers.org/system_requirements/docker/). Com o ambiente configurado, vamos escrever um teste para demonstrar o seu uso.

Imagine que exista uma tabela chamada `posts`, que possui um `id` e um campo `content` e você quer testar a funcionalidade de inserção de dados.

```go
func insertPost(db *sql.DB, content string) error {
	query := `INSERT INTO posts (content) VALUES ($1);`
	_, err := db.Exec(query, content)
	if err != nil {
		return fmt.Errorf("error inserting post: %w", err)
	}

	log.Println("Post inserted successfully")
	return nil
}
```

## Uso básico
Para testar essa função, podemos utilizar o `testcontainers` para subir um banco de dados (no caso, um Postgres) para os testes. Dessa forma, nós temos um container do DB exclusivo para o teste, garantindo que os testes gerenciem suas dependências e minimizando falhas causadas por interdependência entre os testes.

```golang
func TestInsertTable(t *testing.T) {
		postgresContainer, err := postgres.Run(context.Background(),
		"postgres:16-alpine",
		postgres.WithDatabase("test"),
		postgres.WithUsername("user"),
		postgres.WithPassword("password"),
		postgres.BasicWaitStrategies(),
	)

	if err != nil {
		t.Fatalf("Failed to start PostgreSQL container: %v", err)
		return nil, err
	}
	defer postgresContainer.Terminate(t.Context())

  // omiting DB connection and setup

	content := "Hello, Testcontainers!"
	err = insertPost(db, content)
	if err != nil {
		t.Fatalf("Failed to insert post: %v", err)
	}
}
```

## Módulo customizados
No exemplo acima, usamos um módulo já pronto de um Postgres, mas além dele existem inúmeros outros que podem ser encontrados [aqui.](https://testcontainers.com/modules/) Contudo, algumas vezes precisamos criar o nosso próprio módulo. Imagine agora uma função que consuma uma API e faça algo com ela.

```go
func getData(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return fmt.Errorf("Error fetching: %v", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		return fmt.Errorf("Error fetching: %v", resp.Status)
	}

	// do something with the response

	return nil
}
```

Para ter um teste de aceitação válido, precisamos consumir alguma API. Para este caso, vamos criar um container de teste que vai prover uma API qualquer.

```go
func TestGetData(t *testing.T) {
	ctr, err := testcontainers.GenericContainer(t.Context(), testcontainers.GenericContainerRequest{
		ContainerRequest: testcontainers.ContainerRequest{
			Image:        "mitchallen/random-server:latest",
			ExposedPorts: []string{"3100"},
			WaitingFor:   wait.ForLog("random-server:2.1.15 - listening on port 3100!"),
		},
		Started: true,
	})

	if err != nil {
		t.Fatalf("Failed to start container: %v", err)
	}

	defer ctr.Terminate(t.Context())

	url, err := ctr.Endpoint(t.Context(), "http")
	if err != nil {
		t.Fatalf("Failed to get container host: %v", err)
	}

	err = getData(url)
	if err != nil {
		t.Fatalf("Failed to get data: %v", err)
	}
}
```

Como pode ser notado, a criação é bem simples e lembra um Docker compose qualquer. Podemos configurar diversas opções como a imagem, portas expostas, Dockerfile para a build, healthcheck ou o que for necessário para o container. Dessa forma, garantimos que os testes gerem as suas dependências e que simulam um ambiente bem mais próximo do real, aumentando a confiabilidade deles.

## Conclusão
Essa biblioteca tem ajudado bastante nos testes e estou utilizando sempre que necessário. A documentação é bem completa e detalhada e não tive dificuldades ao configurar ou utilizar. Os diversos módulos prontos também facilitam bastante a vida do desenvolvedor, dispensando a recriação de containers de aplicações genéricas como banco de dados ou mockservers. 

Se você quiser ver o exemplo todo, recomendo acessar o [repositório no Github](https://github.com/mfbmina/poc_testcontainers). O que achou do post? Comente aqui abaixo suas impressões! 
