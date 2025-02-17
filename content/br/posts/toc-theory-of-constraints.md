+++
title = 'A teoria das restrições'
date = 2024-11-11T18:19:03-03:00
draft = false
tags = ['tech', 'theory', 'continous improvement', 'book']
+++

No fim do ano passado, minha equipe começou o seu projeto mais importante até então e acabei assumindo a liderança na organização do projeto. Fui responsável por coordenar as reuniões, o processo de entrega, etc. Durante os primeiros meses, o projeto foi andando bem, tomamos algumas decisões difíceis, provamos alguns conceitos e iniciamos o rollout da primeira fase.

Contudo, eu sentia que estava atrasando o time em alguns aspectos e que estava sendo um gargalo no processo todo. Em um 1x1 com o head da área, contei a ele sobre essa frustração e a resposta que recebi foi essa:

> Sim, você é um gargalo na equipe hoje, mas isso não é necessariamente ruim.

Tentei argumentar, pois não entendia como um gargalo não seria ruim e ele começou a me falar da teoria das restrições. No fim de nosso papo, ele me indicou a leitura de um livro chamado The Goal: A Process of Ongoing Improvement, do Eliyahu M. Goldratt e quero compartilhar o que aprendi aqui.

## Métricas

Para entender a teoria das restrições, precisamos entender antes de mais nada qual é o objetivo de qualquer empresa. Pode-se argumentar que as empresas tentam mudar o mundo, prover serviços melhores, mas no fim o foco sempre é o mesmo: fazer dinheiro. E se esse é o objetivo final de cada empresa, todo nosso foco e métricas devem mirar para este objetivo.

Contudo, como fazer dinheiro é um objetivo bem genérico, o autor define três termos para focarmos, sendo eles:
- Produção: como a empresa gera dinheiro;
- Inventário: o dinheiro investido e que eventualmente vai retornar mais dinheiro;
- Despesa operacional: o dinheiro gasto em transformar o inventário em dinheiro;

Nesse sentido, precisamos sempre visar aumentar a produção, reduzindo o inventário e as despesas operacionais e as métricas mais importantes devem se basear em analisar o impacto nesses pontos.

Extrapolando para o mundo de software, temos:
- Produção: funcionalidades, produtos e tudo que possa ser vendido a um cliente;
- inventário: servidores, cloud e tudo que vai ser parte do software na entrega final;
- Despesas operacionais: programadores, P.Os, heads;

Ou seja, precisamos entregar mais funcionalidades e produtos, reduzindo ou mantendo a quantidade de hardware necessária e a quantidade de funcionários.

## Conceito

Agora que já temos um objetivo claro, podemos finalmente falar sobre o que é a Teoria das Restrições (em inglês, ToC ou Theory of Constraints). Ela é um processo de melhoria contínua que visa encontrar os gargalos, ou restrições, de um sistema e trabalhar ao redor deles. É importante frisar que não existe um sistema sem gargalos, eles sempre vão existir e sempre vão regular o fluxo de entregas do seu sistema.

Voltamos então ao ponto inicial do que me fez ler o livro: gargalos não são fundamentalmente ruins! Eles apenas existem e até mesmo são necessários. Um semáforo é um ótimo exemplo de gargalos que não são ruins, já que eles limitam a quantidade de carros em algumas vias e organizam o fluxo. No processo de desenvolvimento de software, temos alguns gargalos que também são considerados essenciais: code review e testes. Ambos diminuem a velocidade de entrega, mas visam garantir uma melhor qualidade final.

Contudo, vale a pena mencionar que a única capacidade que importa é a dos gargalos, já que a velocidade do sistema nunca vai ser maior do que a velocidade dele. A teoria das restrições visa então entender quais são os gargalos do seu sistema e como você pode contorná-los, ou até mesmo os remover, através de um processo sistemático:
1. Identificar as restrições ou os gargalos do sistema;
2. Decidir como tirar o máximo proveito desses gargalos.
3. Evite produzir mais do que o gargalo pode suportar, ou seja, contorne o gargalo.
4. Eleve os gargalos do sistema ou expanda a capacidade com mais investimentos.
5. Se o gargalo tiver sido eliminado, volte à etapa 1 e comece novamente. Se o processo apresentar melhorias, o gargalo pode mudar de lugar e você deve estar preparado para outros gargalos surgirem.

Como citado anteriormente, você pode até mesmo colocar restrições forçadas no seu sistema, para garantir um fluxo contínuo de entrega, no caso do desenvolvimento de software, code reviews, por exemplo. Mas elas não são as únicas! Posso listar, por exemplo, banco de dados, rate-limit em APIs, capacidade do time, pair programming e uma infinidade de outros possíveis gargalos que enfrentamos ao construir um software.

## Conclusão

Apesar de o livro ser um romance e se passar numa fábrica, é possível traçar diversos paralelos com o que passamos no dia a dia de uma equipe de desenvolvimento de software. É importante perceber que nosso trabalho e nosso produto fazem parte de uma cadeia maior, de um sistema, e que entender esse sistema e as restrições deles nos ajudam a manter um fluxo constante e de qualidade nas nossas entregas. Antes de tomar decisões precipitadas de otimização, valide as hipóteses! Tenha certeza de que de fato você está sendo produtivo e que está aumentando o desempenho de um gargalo.

Alguns aprendizados que tirei lendo o livro são:
- Produtividade não é o mesmo que ocupação. Você só é produtivo se seu trabalho está te levando mais perto das suas metas. Ou seja, se você está trabalhando em algo que não vai mexer no ponteiro das metas, talvez isso não seja importante.
- Os gargalos são o ponto mais importante do processo. Para aumentar a eficiência geral, você precisa aumentar as eficiências dos gargalos.
- Qualquer melhoria que não seja num gargalo é uma otimização prematura que não aumenta a eficiência do sistema. Ter ótimos locais não ajuda a ter um sistema ótimo.
- Para identificar seus gargalos, aplique o método científico. Sugira uma hipótese, baseada em dados ou até mesmo em conhecimento empírico. Dada a hipótese, se ela for verdade, algo deve acontecer. Se não acontecer, ela é descartada e uma nova ideia é validada.
- A melhoria é sempre contínua, quando surgirem novos gargalos, aplique o processo sistemático novamente. E sempre vão existir novas restrições e novos pontos de melhoria.

Por fim, aprendi mais sobre o meu papel na equipe e no projeto e entendi como posso moldar a minha participação para reduzir o impacto de ser um gargalo. Essa então é a teoria das restrições e quem gostou eu recomendo fortemente ler o livro!
