+++
title = 'The Mythical Man Month'
date = 2025-02-10T11:39:42-03:00
draft = false
tags = ['tech', 'book', 'software engineering']
+++

Após ler o livro [The Goal]({{< relref toc-theory-of-constraints >}}) fui atrás de uma próxima leitura e me recomendaram o The Mythical Man-Month, do Fred Brooks. Esse livro é um clássico da literatura da computação, desconhecido por mim até agora, mas que descreve o ciclo de desenvolvimento de softwares na década de 70.

O tema central do livro já é descrito no título: o mítico homem-mês. Homem-mês é uma métrica de produtividade assim como as que são utilizadas hoje, por exemplo, pontos, tamanho de camisetas ou qualquer coisa que as equipes utilizam. O argumento é de que a métrica é falha e o número de tarefas, linhas de código ou funcionalidades que uma pessoa entrega não é constante entre projetos. 

O autor até cita dois projetos: programas de controle/operação e um compilador/tradutor.  É visível que a métrica de homem-mês distorce dependendo da complexidade das atividades envolvidas, já que nos programas de controle/operação o resultado foi de cerca de 550 palavras por homem/ano e, no outro tipo de projeto, esse número aumentou para cerca de 2250 palavras por homem/ano.

Em busca de mais produtividade, o primeiro impulso é de adicionar mais pessoas na equipe, já que se entregamos 10 funcionalidades por mês, em uma equipe com 5 pessoas, assumimos que com mais uma pessoa vamos entregar pelo menos 12. Contudo, isso pode gerar menos produtividade, pois o custo para garantir uma comunicação eficiente entre os envolvidos no projeto aumenta significativamente, uma vez que a fórmula para comunicação entre nós é dada por **n * (n - 1) / 2**. Ainda podemos adicionar o custo de treinamento de novas pessoas. Isso ficou conhecido como lei de Brooks.

Portanto, podemos assumir que a comunicação já era um desafio naquela época, um desafio que ainda não conseguimos resolver.  Na época, ainda existiam manuais físicos e ainda exigia um custo extra da manutenção e atualização desses documentos. Hoje temos mais tecnologias como Confluence, Jira, Trello e qualquer outra que visa organizar o trabalho e passar a informação para frente, porém ainda pecamos bastante em como documentar nossos projetos. 

Uma dica importante do Brooks é que todo software deve ter três tipos de documentação:
1. Para usar um programa.
2. Para confiar no programa.
3. Para modificar o programa.

O primeiro tipo é a documentação básica do projeto, como o seu propósito, quais os requisitos do sistema, quais formatos de entrada são aceitos e quais são retornados, quais opções ele aceita e tudo o que o usuário precisa para poder usar seu programa sem surpresas. Ela é importante, por ser a mais valiosa para seu usuário ao tirar todas as suas dúvidas.

A documentação para se confiar no programa são os exemplos, demonstrações, casos de uso e de testes relevantes que vão demonstrar o funcionamento correto do programa. Ela também tem foco no usuário, ao conferir credibilidade ao que foi escrito na documentação básica.

O último tipo é o que é mais comum de vermos: a documentação técnica do projeto. Por exemplo, as responsabilidades de cada função, qual a linguagem utilizada e sua versão, como rodar a suíte de testes, quais as bibliotecas. O foco dela são as pessoas que trabalham ou que vão trabalhar no projeto, ajudando a entender as decisões técnicas tomadas, facilitando a manutenção a longo prazo e garantindo que o software tenha uma vida longa.

A visão de Brooks sobre os projetos também é interessante. Ele comenta que a primeira versão do projeto geralmente vai ser descartada por ser lenta, complexa, incompleta ou qualquer outro problema. Contudo, ele cita que devemos nos preocupar de verdade com a segunda versão do projeto, já que há a tendência de tentar incluir todas as boas ideias descartadas na primeira versão, criando a famosa bola gigante de lama. Testes de regressão, documentação e comunicação evitam o problema. O autor ainda postula que é melhor se ter um projeto pequeno e com funcionalidades concisas do que cheio de funcionalidades e bagunçado.

Segundo o autor, existem dois tipos de atrasos em projetos, os visíveis e os invisíveis. Os visíveis são aqueles que não são necessários justificar: desastres, problemas globalizados e afins. Os atrasos invisíveis são os problemáticos: uma pessoa fica doente, a funcionalidade demora mais do que deveria para ser implementada, uma equipe não entrega o combinado e por aí vai. Assim, um projeto atrasa um ano, atrasando um dia de cada vez. É necessário ter a visão dos projetos com horizontes claros e bem definidos.  O autor reforça bastante sua importância, citando as técnicas utilizadas para garantir as entregas, como ter tabelas com as estimativas iniciais e com estimativas dadas por quem está de fato no dia a dia do projeto, para tentar controlar os atrasos.

O livro também mostra alguns outros pontos que valem ser mencionados, como a necessidade de papéis claros na equipe, como os de liderança técnica ou de produto. Também foi dito que, na época, o uso de linguagens de alto nível melhorou as métricas de produtividade, e que alguns programadores da época achavam ruim essas linguagens por serem lentas, fáceis ou qualquer outra coisa que provavelmente você já ouviu sobre uma linguagem de programação qualquer. E até mesmo sobre a divisão de pessoas em equipes pequenas e cirúrgicas.

Para concluir, é um ótimo livro para se ler por mostrar a visão de como era a vida de uma equipe de desenvolvimento de software nos primórdios da computação. Ainda podemos traçar diversos paralelos com os desafios que temos hoje em dia. É interessante ver as técnicas que eram utilizadas para testar software, medir produtividade ou até mesmo das reclamações da época. Também irá conhecer diversos outros autores e pesquisadores, pois Brooks cita diversos deles. Pode ler que não vai se arrepender!
 