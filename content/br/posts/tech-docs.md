+++
title = 'A importância das documentações técnicas'
date = 2024-09-12T19:11:16-03:00
draft = false
+++

Quando optei pela carreira em computação, aos 15 anos, foi uma decisão baseada em gostar bastante de matemática e física.  Queria me ver longe de escrever textos, redações e etc. Com o passar do tempo, essa visão foi mudando e agora uma das coisas na qual mais valorizo em uma equipe/empresa madura é a existência de documentação técnica.

Documentações nos ajudam a ter conhecimento histórico de decisões na empresa, nos ajudam a revisitar pontos e a entender melhor os pontos fortes e fracos dessas decisões. Além disso, elas são extremamente valiosas para novos membros da equipe, pois contam uma história de como vocês chegaram lá. Quanto mais detalhada, mais rica a história é.

Existem diversas formas e padrões para se criar uma boa documentação técnica. As que mais gosto são as Design Docs, ADRs e RFCs. Vamos então falar melhor sobre cada uma delas!

# Design Docs

As Design docs, em português documentos de design, fornecem detalhes da solução que a equipe tomou. Ela visa documentar as decisões tomadas, dar visibilidade e fomentar o estudo de alternativas e transmitir conhecimento.

Uma boa design doc deve conter um cabeçalho com informações do autor e do tema a ser discutido, contexto do problema, quais as metas a serem atingidas, o que está fora do escopo da decisão, as opções avaliadas, a solução escolhida e até mesmo preocupações gerais e evoluções futuras.

Para se aprofundar mais nas design docs, recomendo visitar [https://www.designdocs.dev/](https://www.designdocs.dev/). Lá você encontra templates e dicas de como estruturar uma boa design doc. Além disso, o Gabriel Mendes publicou um ótimo [post](https://medium.com/inside-picpay/uma-introdu%C3%A7%C3%A3o-aos-design-docs-8590f28f4cc1) no Medium do PicPay sobre as Design Docs, que são bem difundidas aqui dentro!

# ADRs

ADR, ou Architecture Decision Records (Registro de decisões arquiteturais em português livre), visam documentar as decisões tomadas no processo de desenvolvimento de software ou de um projeto. Uma boa ADR fornece um histórico das decisões tomadas e os porquês associados a elas. 

Uma boa ADR deve conter um título, o contexto do problema, quais as opções foram avaliadas e qual a decisão tomada, assim como sua justificativa.

Para quem quiser se apronfudar mais sobre ADRs, recomendo visitar [https://adr.github.io/](https://adr.github.io/). Também tem um ótimo [post](https://medium.com/@jhonywalkeer/guia-completo-sobre-architecture-decision-records-adr-defini%C3%A7%C3%A3o-e-melhores-pr%C3%A1ticas-f63e66d33e6) do Jhonny Walker só sobre ADRs para quem quiser saber mais sobre o tema!

# RFCs

RFC é uma sigla em inglês para `Request for Comments`. Seu objetivo é especificar em detalhes uma solução, padrão ou projeto. Geralmente é um processo extremamente estruturado e que dado o nível de detalhes, pode levar um certo tempo para concluir o documento. Contudo, o seu nível de profundidade e detalhamento não deixa dúvidas sobre o projeto.

A [IETF](https://datatracker.ietf.org/) (Internet Engineering Task Force) mantém grande parte das RFCs de diversos projetos e é uma grande fonte de como devem ser escritas com o [guia de RFCs](https://www.ietf.org/process/rfcs/). O Jhonny Walker também escreveu um [post](https://medium.com/@jhonywalkeer/entendo-a-rfc-o-guia-definitivo-para-padr%C3%B5es-da-internet-e-desenvolvimento-de-protocolos-384e9f7abf58) bem massa sobre RFCs.

# Considerações finais

Espero que com essas informações e os links sobre algumas formas de documentação técnica, você perceba que não importa qual modelo você e sua equipe desejem seguir, mas o importante é que exista alguma forma de documentação. Todas elas tem seus pontos fortes e seus pontos fracos, cabe a você entender os trade-offs de cada uma delas e ver o que se aplica melhor no cénario da sua empresa ou da sua equipe. Também é importante dosar a rigidez do processo e o nível de detalhamento, pois como é consenso geral, documentações tendem a envelhecer rápido.

Por exemplo, aqui no PicPay utilizamos RFCs para decisões que afetam a empresa como um todo. São documentos formais, com revisores e processos muito bem definidos. Já nas decisões que afetam uma equipe em particular, ou uma área menor, geralmente se é utilizado design docs, por justamente ser mais simples.

Para fechar, quanto mais você avança em sua carreira técnica, é esperado que grande parte do seu papel seja de gerar impacto e tenho certeza que saber como escrever boas documentações vai te fazer atingir outros patamares. Espero que tenha gostado do post e me conta: Qual padrão você utiliza para suas documentações técnicas?
