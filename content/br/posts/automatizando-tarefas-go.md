+++
title = 'Automatizando tarefas com Go'
date = 2022-03-11
draft = false
tags = ['go']
+++

Na vida de uma pessoa desenvolvedora, nos deparamos diversas vezes com tarefas monótonas, repetitivas e sempre nos vem a ideia na cabeça de como podemos automatizar isso. Aqui na Trybe, temos dois cenários bem legais de como Go nos ajudou a automatizar algumas dessas tarefas.

O primeiro cenário é o nosso **trybe-cli**. Isso mesmo, escrevemos um client em Go, para ajudar nossas pessoas desenvolvedoras e executar tarefas repetitivas com alguns comandos. Neste momento, nosso cli nos permite:

- criar todos os arquivos da esteira de deploy;
- criar flags e segmentos no nosso serviço de feature flags;
- gerenciar as tarefas no Jira: mover a tarefa pra doing, mover a tarefa pra review e até mesmo ter algumas estatísticas de tempo do ticket aberto e etc;

Para escrever esse cli, duas ferramentas nos ajudaram muito. A primeira delas é a lib [cobra](https://github.com/spf13/cobra) e ela auxilia na criação de CLIs e é utilizada por diversos lugares como Kubernetes, Hugo, GitHub cli, Docker e mais todas os lugares apontados [nessa lista](https://github.com/spf13/cobra/blob/master/projects_using_cobra.md). Das diversas funcionalidades, as mais legais são:

- sugestões inteligentes: (`app srver`... did you mean `app server`?);
- criação automática de um help para seu cli;
- criação automática de auto-complete para `bash`, `zsh` e etc;
- reconhece flags de forma automática e inteligente, por exemplo `-h` e `--help`;

A segunda ferramenta é a [goreleaser](https://github.com/goreleaser/goreleaser), responsável por buildar e gerar os binários em todas as plataformas: MacOS, Linux e Windows. Para nós, isso é extremamente útil, já que não precisamos nos preocupar com as particularidades de cada sistema operacional e nossas pessoas desenvolvedoras não ficam travadas em um sistema ou outro. Isso só é possível porque Go é uma linguagem cross-plataform.

O segundo cenário, é um **micro-serviço** responsável somente por receber requisições do GitHub.  Para explicar o porque fazemos isso, vale a pena explicar um pouco sobre nosso ambiente de desenvolvimento. Hoje temos os seguintes ambientes, que podem ser traduzidos em clusters kubernetes:

- _produção_: ambiente que as pessoas estudantes, candidatas e empregadas utilizam;
- _staging_: ambiente que simula o ambiente produtivo. Utilizado para testar funcionalidades maiores;
- _preview-app_: ambiente de teste para funcionalidades especificas. Cada pull request aberto, em cada repositório nosso, gera um pod neste cluster;
- _homologação_: ambiente de testes utilizados por nossos QAs;

Como podem ver, o cluster das preview-apps cresce exponencialmente, já que pra cada pull request aberto, um pod lá é criado. Para consumir menos recursos, nossos SREs criaram uma tarefa em background para limpar os pods que eram mais velhos que 5 dias. Contudo, ainda era possível melhorar este gerenciamento.

Ao fechar um pull request, seja ao fazer um merge ou ao recusar, aquele pod se torna automaticamente obsoleto e dispensável. Com isso, um pequeno micro-serviço foi criado com a única responsabilidade de escutar estes eventos do GitHub e apagar o pod. Assim nosso cluster só mantem os pods que estão de fato sendo utilizados.

Como podemos ver, Go é bem versátil e que pode nos ajudar em diversos cenários de automação, criando desde clis multi-plataformas até micro-serviços pequenos e robustos. É definitivamente uma linguagem a se avaliar neste cenário.

Você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
