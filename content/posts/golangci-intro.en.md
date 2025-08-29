+++
title = 'First steps with Go linters'
date = 2024-02-17T17:39:22-03:00
draft = false
+++

Linter is a static code analysis tool used to find programming errors, bugs, leaks of code standards, and even security flaws. These tools help developers because they save time by identifying issues before they happen in the production environment. It also keeps developers from unnecessarily checking if your colleague used the team standards.

Each programming language has their own tools: Ruby has [Rubocop](https://github.com/rubocop/rubocop), JS has [Eslint](https://eslint.org/), and Go can't be different. Searching at [Awesome Go](https://github.com/avelino/awesome-go?tab=readme-ov-file#code-analysis), a curated list of Go software, many tools can lint your code, but my favorite is [**golangci-lint**](https://github.com/golangci/golangci-lint/tree/master). It is an aggregator of linting tools, meaning you only need one tool for multiple linters in your project.

## Install & configure
You can run it during your development lifecycle by having it on your machine. The way to go is to [install](https://golangci-lint.run/usage/install/#local-installation) the official distribution for your operation system. After installing, you are ready to go and can run `golangci-lint run` at your terminal. Easy peasy! If you wish, you can also setup your [favorite IDE](https://golangci-lint.run/usage/integrations/) to use it as default!

The coolest about this tool is being able to configure it using configuration files, being able to even choose which linting will be executed, what is the desired parallelism, rules for each linter, etc. To configure your project, create a `.golangci.yml` file and place your settings there. There is a [reference file](https://github.com/golangci/golangci-lint/blob/master/.golangci.reference.yml) from the developers that shows all the possibilities, but it is too complex. [@maratori](https://github.com/maratori) made an excellent curation of the best configuration, and you can find it [here](https://gist.github.com/maratori/47a4d00457a92aa426dbd48a18776322)!

## Conclusion
Having a configuration file for your project, you can ensure that everyone has the same checkings as you during their development lifecycle. You can also use it within your CI, ensuring extra confidence when reviewing your colleagues' PR. If your team uses GitHub Actions, there is one [ready-to-go action](https://github.com/golangci/golangci-lint-action).

This is a small blog post presenting the [golangci-lint](https://golangci-lint.run/) tool and demonstrating how to use it. If you wish to know more about it, visit their [website](https://golangci-lint.run/).
