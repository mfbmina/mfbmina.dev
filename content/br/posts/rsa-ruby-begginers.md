+++
title = 'RSA em Ruby para iniciantes'
date = 2018-12-13T09:22:18-03:00
draft = false
tags = ['ruby','rsa','introduction']
+++

## Introdução

![Foto tirada por [rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) e disponível em [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/8000/0*jySVcIvNiplrN6kx)

RSA é um algoritmo de encriptação baseado em chaves públicas e privadas , que foi desenvolvido pensando na dificuldade de se fatorar dois números primos grandes. O seu nome vem das iniciais de seus criadores Rivest, Shamir e Adleman.

Computacionalmente falando, ele é um algoritmo caro, mas mesmo assim é amplamente utilizado no mercado e um dos mais importantes algoritmos de criptografia. Seu uso é geralmente realizado de maneira indireta, como por exemplo pelo OpenSSL, que utiliza RSA na geração de suas chaves ou até mesmo quanto utilizamos chaves SSH ou certificados SSL, que também são encriptados por este algoritmo.

Nesta publicação, vamos tentar entender como o RSA funciona através de exemplos escritos em Ruby. Leve em consideração que o código escrito é um estado de caso, desenvolvido para ser simples e tem diversos pontos baixos, como por exemplo, não aceitar multibyte strings.

## Para entender o funcionamento, vamos imaginar a seguinte situação:

Alice e Bob são amantes e querem trocar cartas de amor sem que ninguem seja capaz de as ler, mas infelizmente eles estão em um local público e com vários curiosos a sua volta. Após um pouco de pesquisa, ambos decidem utilizar o RSA para manter as cartas seguras.

O primeiro passo é feito pela Alice, pedindo ao Bob suas chaves públicas. Ao receber as chaves, Alice encripta a carta e a envia ao Bob, que ao receber, só precisa utilizar sua chave privada para desencriptar o texto e poder ler.

O mais legal desse algoritmo é que nem a Alice ou o Bob se preocupam se alguem externo irá ler a mensagem, pois a única pessoa capaz de desencriptar é o dono da chave privada, neste caso, Bob.

## O algoritmo

Agora você deve estar se perguntando: como posso gerar minhas própias chaves? Esta é definitivamente a parte mais díficil e complexa do algoritmo.

Primeiramente, devemos escolher valores para **p** e **q**. Por motivos de segurança estes valores devem seguir as seguintes regras:

* devem ser diferentes um do outro;

* devem ser números inteiros e randômicos;

* devem possuir magnitude similar;

* devem ser primos;

* devem ser grandes em magnitude. Pode-se utilizar números de até 1024 bytes, mas para ser considerado seguro é recomendado utilizar números de 2048 bytes.

```ruby
def p
    @p ||= random_prime_number
end

def q
    @q ||= random_prime_number
end

def random_prime_number
    number = Random.rand(10..100)
    until Prime.prime?(number) || number == p || number == q do
      number = Random.rand(10..100)
    end

    number
end
```

>  Nota: neste estudo de caso eu estou quebrando a regra de se gerar grandes números primos, pois eu quero um algoritmo que seja rápido. Para entender o por quê de se gerar números grandes leia *[aqui](https://pt.wikipedia.org/wiki/Fatora%C3%A7%C3%A3o_de_inteiros)*.

O próximo passo é gerar a chave **n**, facilmente calculado ao multiplicar **p** por **q**, como demonstrado a seguir:

```ruby
def n
    @n ||= p * q
end
```

Outro valor a ser encontrado é o da *[função de Carmichael](https://pt.wikipedia.org/wiki/Fun%C3%A7%C3%A3o_de_Carmichael)*. Este valor pode ser encontrado através do menor multiplicador comum entre **p—1** e **q—1**. Algumas implementações antigas do RSA calculam este valor simplesmente multiplicando **p—1** e **q—1** e para manter este algoritmo simples, vamos fazer o mesmo. Neste exemplo, nosso método Ruby irá se chamar totient em menção ao nome da função em inglês.

```ruby
def totient
    @totient ||= (p — 1) * (q — 1)
end
```

Chegou a hora de gerarmos nossa chave pública **e**. Esta chave necessita ter um valor que seja [coprimo](https://pt.wikipedia.org/wiki/N%C3%BAmeros_primos_entre_si) ao valor da função de Carmichael. Explicando superficialmente, números coprimos são aqueles que só possuem o número 1 como um divisor comum. Uma maneira legal de se encontrar o valor para esta chave é verificar se o número é primo e caso seja, verificar se o totienté divisível pelo mesmo.

```ruby
def e
    @e ||= totient.downto(2).find do |i|
        Prime.prime?(i) && totient % i != 0
    end
end
```

O último valor a ser encontrado, e possivelmente o mais importante de todos, é a chave privada **d**. **d** é o [inverso multiplicativo](https://pt.wikipedia.org/wiki/Inverso_multiplicativo) do resto da divisão entre **e** e o totient.

```ruby
def d
    @d ||= invmod(e, totient)
end

# Credits to [https://rosettacode.org/wiki/Modular_inverse#Ruby](https://rosettacode.org/wiki/Modular_inverse#Ruby)
def extended_gcd(a, b)
    last_remainder, remainder = a.abs, b.abs
    x, last_x, y, last_y = 0, 1, 1, 0
    while remainder != 0
        last_remainder, (quotient, remainder) = remainder, last_remainder.divmod(remainder)
        x, last_x = last_x — quotient*x, x
        y, last_y = last_y — quotient*y, y
    end

    return last_remainder, last_x * (a < 0 ? -1 : 1)
end

# Credits to [https://rosettacode.org/wiki/Modular_inverse#Ruby (https://rosettacode.org/wiki/Modular_inverse#Ruby)
def invmod(e, et)
    g, x = extended_gcd(e, et)
    raise ‘The maths are broken!’ if g != 1
    x % et
end
```

Após termos todas as chaves, chegou a hora de finalmente encriptarmos nossa mensagem. A função que realiza isso é: **c= (Mˆe)%n**.

```ruby
def cipher(message)
    message.bytes.map do |byte|
    cbyte = ((byte.to_i ** e) % n).to_s
    missing_chars = n.to_s.size — cbyte.size
    '0' * missing_chars + cbyte
    end.join
end
```

Vale ressaltar que **M** é a mensagem convertida para um inteiro e que o seu valor deve ser menor que **n**.

O **padding scheme**, ou pela tradução literal, esquema de preenchimento, é extremamente importante para este algoritmo. Para evitar diversos problemas, as implementações do RSA também contam com algum **padding scheme** para o valor de **M** antes de encriptá-lo. Por exemplo, um bom esquema irá garantir que a mensagem não se transforme uma estrutura previsível e com isso irá alguns tipos de ataque. Dois dos padrões mais utilizados para este fim são o [PKCS#1](https://en.wikipedia.org/wiki/PKCS_1) e o [RSA-PSS](https://en.wikipedia.org/wiki/Probabilistic_signature_scheme).

Para este estudo de caso, foi criado um esquema realmente simples, baseado no tamanho da **string** e nos **bytes**. A matemática nos permite dizer que o resto da divisão de um valor A por um valor B nunca será maior que B. Tendo isso em mente, é possível assumir que cada caractere encriptado vai ter a mesma magnitude de **n** e se não tiver, é só preencher com zeros no começo.

Ter um esquema definido ajuda ao decriptar a mensagem, pois sabemos exatamente qual é a sua estrutura e somente nos resta aplicar a função **M=(cˆd)%n** para cada caractere.

```ruby
def decipher(ciphed_message)
    ciphed_message.chars.each_slice(n.to_s.size).map do |arr|
        (arr.join.to_i ** d) % n
    end.pack('c*')
end
```

Eu já tratei sobre **bytes** e sobre o que método **pack* faz em minha postagem sobre ROT N. Caso ainda tenha dúvidas, é só checar [aqui]({{< ref "/caesar-cypher-ruby.md" >}} "Implementando a Cifra de César em Ruby").

## Conclusão

Neste ponto é possível que você tenha percebido algumas falhas neste algoritmo. Uma delas é justamente escolher valores pequenos para os expoentes de encriptação. Algumas pessoas o fazem, pois isto reduz drasticamente o tempo de encriptação ou decriptação de uma mensagem, porém deixa o algoritmo facilmente quebrável.

Outra grande falha se dá a partir da natureza determinística do RSA. É possível gerar valores para **p** e **q** e consequentemente gerar valores para **e** e **n.** Se estes valores baterem com os valores públicos, é possível gerar a chave privada e decriptografar a mensagem.

Por último, os **padding schemes** podem ser frágeis. No exemplo dado, o esquema é "burro" e você pode facilmente testar a frequência de aparecimento dos valores na mensagem encriptada e ir chutando os valores até ter a mensagem decriptada.

Este é minha segunda postagem sobre criptografia e minha ideia é implementar algumas das técnicas conhecidas e tentar explicar facilmente aqui. Se você tiver interesse em olhar o código completo, por favor acesse o [repositório no Github](https://github.com/mfbmina/cipher_studies/blob/master/rsa.rb).

Você também pode me encontrar no **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)** ou **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
