+++
title = 'RSA in Ruby for beginners'
date = 2018-12-13T09:22:18-03:00
draft = false
+++

RSA is a public/private encryption algorithm and it is based on the difficulty of the factorization of the product of two large prime numbers. It was named after its creators Rivest, Shamir, and Adleman.

It is an expensive algorithm, computationally speaking, and because of this, it is not common to use it directly, but it is still widely used in the market and is one of the most important encryption algorithms. As an example, OpenSSL implements this algorithm for generating keys and it is commonly used for encrypting SSL certificates or SSH keys.

In this post, we will try to understand how RSA works with examples written in Ruby. Take into consideration that it is a study case algorithm that was developed to be simple and has a lot of downsides, for example, it does not work for multibyte strings.

## In order to understand how it works, let's imagine this situation:

Alice and Bob are lovers and they want to change their love messages without anyone being able to read them, but they are in a public place. After a little research, they decided to use RSA to keep the messages safe.

The first step is done by Alice, asking Bob for his public keys. After having the keys, Alice encrypts the message and sends it to Bob. When he receives the message, he just has to use his private key to decrypt the text.


The cool thing here is that neither Alice nor Bob care about someone reading the message, because the only person capable of decrypting it is the owner of the private key, in this case, Bob.

## The algorithm

So, how can I generate my own keys? This is definitely the hardest part of the algorithm.

First of all, you should choose the values for **p** and **q**. For security purposes, these values must follow some rules, such as:
* be different from each other;
* be random integers;
* be similar in magnitude;
* be prime;
* be very large. You can use at least 512 digits, but to be considered safe you have to choose integers with 1024 digits;

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

>  Note: I'm breaking the rule of generating large numbers, just because it is a study case and I want a fast algorithm. If you want to understand the need of having prime and large numbers, please check **[here](https://pt.wikipedia.org/wiki/Fatora%C3%A7%C3%A3o_de_inteiros)**.

The second step is generating **n**, just multiplying **p** with **q** as you can see below:

```ruby
def n
    @n ||= p * q
end
```

You should also find the value of [Carmichael's totient function](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FCarmichael%2527s_totient_function). You can compute this by finding the least common multiplier between **p - 1** and **q - 1**. For older specifications of RSA, just calculate it by doing: **(p - 1) * (q - 1)**. To keep it simple, I choose to implement it using the old specification and will just call it by **totient**.

```ruby
def totient
    @totient ||= (p — 1) * (q — 1)
end
```

The next step is generating e, our public key. This key must be [coprime](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FCoprime) to our totient. In short, coprimes are numbers that just have 1 as a common divisor. A cool approach to find this value is to check if the number is a prime and if the rest of the division between the totient and the value is not zero.

```ruby
def e
    @e ||= totient.downto(2).find do |i|
        Prime.prime?(i) && totient % i != 0
    end
end
```

The last and the most important key is the private one, called **d**. d is the [modular multiplicative inverse](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FModular_multiplicative_inverse) of **e % totient**.

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

After generating the keys, we just have to encrypt our message. The function is **c = (Mˆe) % n**.

```ruby
def cipher(message)
    message.bytes.map do |byte|
    cbyte = ((byte.to_i ** e) % n).to_s
    missing_chars = n.to_s.size — cbyte.size
    '0' * missing_chars + cbyte
    end.join
end
```

Please note that **M** is the message converted to an integer and **M** should be lower than **n**.

The [padding scheme](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FPadding_%28cryptography%29) is also really important for the RSA algorithm. To avoid several problems, RSA implementations usually implement a structured padding into the value m before encrypting it. A great padding will ensure that a message doesn't turn into a predictable message structure. Two of the most used padding schemes are [PKCS#1](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FPKCS1) and [RSA-PSS](https://medium.com/r/?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRSA-PSS).

For this study case, I have created a simple padding scheme based on the string size and bytes. Mathematically saying, I know that **a % b** is never higher than **b**. Keeping this in mind, every encrypted char will have the same size and if not, I will fill them with zeros in the beginning.

In order to decrypt the message, you just call this function **M = (cˆd) % n** using the private key **d** for every char.

```ruby
def decipher(ciphed_message)
    ciphed_message.chars.each_slice(n.to_s.size).map do |arr|
        (arr.join.to_i ** d) % n
    end.pack('c*')
end
```

Eu já tratei sobre **bytes** e sobre o que método **pack* faz em minha postagem sobre ROT N. Caso ainda tenha dúvidas, é só checar .
I've already talked about what **pack** and **bytes** do in my post about ROT N. If you have some doubts about it, you can [check it out]({{< ref "caesar-cypher-ruby.md" >}}).

## Conclusion

At this point, you can notice that this algorithm is not 100% safe. One of the most common flaws is choosing small encryption exponents. Some people usually prefer to choose small values because it reduces the time to encrypt or decrypt a message significantly, but it can be easily broken down.

Another big flaw lies in the deterministic nature of RSA. You can generate values for **p** and **q** and therefore test if the values for **e** and **n** match with the given public ones. If positive, you will have the right value for **d** and can decrypt the message.

Also, padding schemes can be fragile. In my example, I am choosing a "dumb" padding scheme based on bytes and filling them with zeros. If you want to, you can easily check for frequency in the encrypted text and with this information, decrypt the message.

This is the second post about cipher, my idea is to implement some techniques and try to easily explain it here. If you want to take a look at the whole code, please check at [GitHub](https://github.com/mfbmina/cipher_studies/blob/master/rsa.rb).

You also can find me on **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)**, or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
