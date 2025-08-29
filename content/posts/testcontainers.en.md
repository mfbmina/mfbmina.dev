+++
title = 'testcontainers: improving E2E tests'
date = 2025-07-16T15:26:36-03:00
draft = false
+++

In a [previous post]({{< relref api-testing >}}) I showed some ways to improve tests by creating API mocks that we call. However, that's not always enough, and we might need E2E tests (or acceptance tests), for example, to test integrations with databases, messaging services, or anything else. For these cases, I'm here to introduce the Testcontainers tool.

The [Testcontainers](https://testcontainers.com/) library is an open-source tool that lets you create containers during test execution. The idea is for your application's test dependencies to be part of the code, avoiding the need for mocks and even local dependency installation. Another big advantage is that it makes it easier to achieve test isolation and replicability among developers. The tool supports various programming languages like Go, Ruby, Elixir, Java, etc.

## Installation
To get started, you need to have [Docker](https://www.docker.com/) installed on your machine or an alternative like [Rancher](https://www.rancher.com/) or [Colima](https://github.com/abiosoft/colima). Depending on your setup, you might need to configure some additional steps on your machine, which you can find [here](https://golang.testcontainers.org/system_requirements/docker/). With the environment configured, let's write a test to demonstrate its use.

Imagine there's a table called `posts`, which has an `id` and a `content` field, and you want to test the data insertion functionality.

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

## Basic usage
To test this function, we can use `testcontainers` to spin up a database (in this case, a Postgres) for the tests. This way, we have a dedicated DB container for the test, ensuring tests manage their dependencies and minimizing failures caused by interdependency between tests.

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

## Custom modules
In the example above, we used a ready-made Postgres module, but there are countless others you can find [here.](https://testcontainers.com/modules/) However, sometimes we need to create our own module. Now, imagine a function that consumes an API and does something with it.

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

To have a valid acceptance test, we need to consume an API. For this case, we'll create a test container that will provide any API.

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

As you can see, creating it is quite simple and resembles a Docker Compose setup. We can configure various options like the image, exposed ports, Dockerfile for the build, health check, or whatever is needed for the container. This way, we ensure that the tests manage their dependencies and simulate an environment much closer to reality, increasing their reliability.

## Conclusion
This library has been very helpful in my tests, and I'm using it whenever necessary. The documentation is very complete and detailed, and I haven't had any difficulties setting it up or using it. The various ready-made modules also make a developer's life much easier, eliminating the need to recreate containers for generic applications like databases or mock servers.

If you want to see the full example, I recommend checking out the [Github repository](https://github.com/mfbmina/poc_testcontainers). What did you think of the post? Share your thoughts below!
