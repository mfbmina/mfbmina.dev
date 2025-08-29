+++
title = 'Automating tasks with Go'
date = 2022-03-11
draft = false
+++

As developers, we often come across monotonous, repetitive tasks, and the thought of how we can automate them always comes to mind. Here at Trybe, we have two really cool scenarios where Go helped us automate some of these tasks.

## First scenario
### trybe-cli
The first scenario is our **trybe-cli**. That's right, we wrote a client in Go to help our developers perform repetitive tasks with a few commands. Currently, our CLI allows us to:

- create all the files for the deployment pipeline;
- create flags and segments in our feature flags service;
- manage tasks in Jira: move a task to doing, move a task to review, and even get some statistics on ticket open time, etc.;

Two tools helped us a lot in writing this CLI. The first one is the [cobra](https://github.com/spf13/cobra) library, which helps in creating CLIs and is used by many places like Kubernetes, Hugo, GitHub cli, Docker, and all the other places listed [here](https://github.com/spf13/cobra/blob/master/projects_using_cobra.md). Among its many features, the coolest are:

- intelligent suggestions: (`app srver`... did you mean `app server`?);
- automatic help generation for your CLI;
- automatic auto-complete generation for `bash`, `zsh`, etc.;
- automatically and intelligently recognizes flags, for example `-h` and `--help`;

### goreleaser
The second tool is [goreleaser](https://github.com/goreleaser/goreleaser), which is responsible for building and generating binaries for all platforms: MacOS, Linux, and Windows. For us, this is extremely useful, since we don't need to worry about the specifics of each operating system, and our developers don't get stuck on one system or another. This is only possible because Go is a cross-platform language.

## Second scenario
The second scenario is a **micro-service** that is only responsible for receiving requests from GitHub. To explain why we do this, it's worth explaining a bit about our development environment. Today we have the following environments, which can be translated into Kubernetes clusters:

- _production_: the environment that students, candidates, and employees use;
- _staging_: an environment that simulates the production environment. Used to test larger features;
- _preview-app_: a test environment for specific functionalities. Each pull request opened in one of our repositories generates a pod in this cluster;
- _homologation_: a test environment used by our QAs;

As you can see, the preview-app cluster grows exponentially, since a pod is created there for each open pull request. To use fewer resources, our SREs created a background task to clean up pods that were older than 5 days. However, this management could still be improved.

When a pull request is closed, whether by merging or declining, that pod automatically becomes obsolete and dispensable. With that, a small micro-service was created with the sole responsibility of listening to these GitHub events and deleting the pod. This way, our cluster only keeps the pods that are actually being used.

## Conclusion
As we can see, Go is very versatile and can help us in various automation scenarios, from creating multi-platform CLIs to small and robust micro-services. It's definitely a language to consider for this type of work.

You can also find me on **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)**, or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
