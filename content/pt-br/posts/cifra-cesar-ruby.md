+++
title = 'Implementando a Cifra de César em Ruby'
date = 2018-11-25T09:22:18-03:00
draft = false
+++

A Cifra de César é a mais simples e conhecida técnica de encriptação. Ela também é conhecida por **cifra de troca**, **código de César, troca de César**, **ROT N** ou **ROT13**, sendo esse o nome mais comum, trocando as letras em treze rotações.

![](https://cdn-images-1.medium.com/max/2000/1*ere0aTX1tmsyruzHMtbtlg.png)

Este algoritmo é extremamente simples, pois somente funciona para letras entre **A** e **Z,** ignorando todos os caracteres especiais como pontos, espaços em branco e letras como **Ç** or **Á**.

Começando nossa implementação, vamos precisar criar uma classe responsável por saber o que queremos encriptar e quantas trocas vamos fazer.

{{< gist mfbmina 5396859b9f4fad4723bce338d24a7ec7 >}}

Feito isso, precisamos trocar os caracteres pelo correspondente depois de N rotações, então vamos adicionar o método **shift**:

{{< gist mfbmina 7589f0307cf727520452fa644d230e8c >}}

Neste ponto, nos deparamos com um pequeno problema. Se o valor de **new_byte** (o valor original em bytes somado com quantos bytes que desejamos trocar) for maior que **Z** ou **z,** é necessário voltar ao valor de **A** ou **a** novamente. Quando isso acontece, também é necessário subtrair um, pois temos que levar o valor do caractere inicial em consideração.

Neste código, **initial_byte** é referente ao valor de **A** ou **a** e **limit_byte** é o byte referente a **Z** ou **z**. Desta maneira, sempre vamos rotacionar entre as letras.

Vamos agora descobrir como é possível encontrar os bytes presentes em cada caractere de uma **string**. Olhando mais a fundo no Ruby, isto é facilmente feito pelo método **[bytes](https://ruby-doc.org/core-2.5.1/String.html#method-i-bytes). ** Tendo conhecimento disto, temos o seguinte:
* **A** é representado pelo byte 65.
* **Z** é representado pelo byte 90.
* **a** é representado pelo byte 97.
* **z** é representado pelo byte 122.

{{< gist mfbmina 1878ef26ceb77929eab8cb0e4a625f73 >}}

Então, o próximo passo é encontrar qual o valor que vamos usar para **initial_byte** e **limit_byte** ou se vamos ignorar este caractere. Para isso, vamos checar se os bytes dentro do texto fornecido estão dentro da variação de bytes para as letras que queremos, se não, vamos simplesmente retornar o valor do byte.

{{< gist mfbmina c3068267b78e21a8f746a5c05f8a2e02 >}}

Em Ruby, a maneira mais fácil de converter estes bytes em uma *string* é chamando o método **[pack](https://ruby-doc.org/core-2.5.1/Array.html#method-i-pack)**. Também é necessário fornecer ao método a opção **c** como um parâmetro. Neste caso, **c** significa um valor inteiro positivo de 8-bit e significa que todo o **array** deve ser convertido.

Com estes dois métodos somos capazes de encriptar uma mensagem utilizando a Cifra de César. Se quisermos decifrar uma mensagem, devemos trocar as letras na direção oposta, como código abaixo:

{{< gist mfbmina 177a26ee79ee2b116d5aa82e52598e01 >}}

Já o método *decipher* deve ser exatamente igual ao método **cipher**, portanto só é necessário duplicar o método **cipher** e trocar a assinatura do método para **decipher**.

Agora vamos finalmente testar o nosso código!

{{< gist mfbmina b5d8073fb9160f5980a786f4ae8d59bc >}}

Também é possível decifrar uma mensagem:

{{< gist mfbmina e3682c77b296abd8861f16dcb69b88d4 >}}

Uma coisa interessantíssima acontece quando nós utilizamos a quantidade de rotações igual a 13. Como temos 26 letras entre **A** e **Z**, não faz diferença nenhuma se chamamos o método **cipher** ou **decipher**, pois ele sempre vai retornar exatamente o mesmo resultado.

{{< gist mfbmina 957ca139dbbc316a31d48322059f2114 >}}

Para ter resultados diferentes, é necessário trocar a quantidade de rotações.

Infelizmente essa implementação tem um problema. Os caracteres em Ruby podem ser multi-bytes e originalmente não levei isto em consideração. Para que funcione ignorando os caracteres multi-bytes deve se implementar a solução utilizando divisão modular.

Este é minha primeira postagem sobre criptografia e minha ideia é implementar algumas das técnicas conhecidas e tentar explicar facilmente aqui. Se você tiver interesse em olhar o código completo, por favor acesse o [repositório no Github](https://github.com/mfbmina/cipher_studies/blob/master/caeser.rb).

Você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
