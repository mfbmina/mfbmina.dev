+++
title = 'Primeiros passos com linters em Go'
date = 2024-02-17T17:39:22-03:00
draft = false
tags = ['go','linter','introduction']
+++

Linter é uma ferramenta de análise estática de código que é usada para encontrar erros de programação, bugs, falta de padronização de código e até mesmo falhas de segurança. Essas ferramentas ajudam os desenvolvedores pois permitem que bastante tempo seja salvo, já que identificam problemas antes mesmo de chegarem em produção. Além disso, são extremamente utéis no processo de code review, já que salvam os desenvolvedores de desnecessáriamente avaliar se o colega utilizou os padrões da equipe ou não.

Cada linguagem de programação possui seus própios linters: Ruby tem o [Rubocop](https://github.com/rubocop/rubocop), JS possui o [Eslint](https://eslint.org/) e com Go não podia ser diferente. Realizando uma busca no [Awesome Go](https://github.com/avelino/awesome-go?tab=readme-ov-file#code-analysis), uma lista curada de softwares em Go, existem diversas ferramentas que realizam lint do seu código. Contudo minha favorita é a [**golangci-lint**](https://github.com/golangci/golangci-lint/tree/master), pois ela é na verdade um agregador de ferramentas de lint, ou seja, você só precisa de uma ferramenta para ganhar inúmeros tipos de lint em seu projeto.

Para rodar o linter durante seu ciclo de desenvolvimento você deve primeiramente instalar ele em sua máquina. A forma recomendada pelos desenvolvedores é [instalar](https://golangci-lint.run/usage/install/#local-installation) atráves da distribuição oficial para seu sistema operacional. Após a instalação é só rodar o comando `golangci-lint run` em seu terminal e pronto! Simples e fácil! Se você preferir pode até [configurar sua IDE](https://golangci-lint.run/usage/integrations/) preferida para utilizar o golangci-lint por padrão!

O mais legal é que essa ferramenta é extremamente configurável através de arquivos de configuração, sendo possível escolher, por exemplo, quais linters serão executados, qual o paralelismo desejado, quais as regras para cada linter e etc. Basta criar um arquivo `.golangci.yml` em seu projeto e colocar as configurações lá. Existe um [arquivo de referência](https://github.com/golangci/golangci-lint/blob/master/.golangci.reference.yml) do própio projeto que ensina a configurarmos todas as opções existes, porém ele é extremamente complexo. O [@maratori](https://github.com/maratori) fez uma excelente curadoria sobre quais a melhor configuração e ela pode ser encontrada [aqui](https://gist.github.com/maratori/47a4d00457a92aa426dbd48a18776322)!

Tendo um arquivo de configuração em seu projeto, é possível garantir que todos seu colegas estão com as mesmas verificações que você durante o ciclo de desenvolvimento. Além disso, também é possível executar a ferramenta em um CI, garantindo ainda mais confiança ao revisar os PRs automaticamente. Se sua equipe utilizar o GitHub Actions, existe até uma [action pronta](https://github.com/golangci/golangci-lint-action) para só utilizar!

Este é um post curto que visa apresentar o [golangci-lint](https://golangci-lint.run/) e mostrar como começar a utilizar essa ferramenta! Se deseja conhecer mais sobre a ferramenta, visite o seu [site](https://golangci-lint.run/).
