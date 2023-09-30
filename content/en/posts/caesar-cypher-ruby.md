+++
title = 'Implementing the Caesar Cipher in Ruby'
date = 2018-11-25T09:22:18-03:00
draft = false
tags = ['ruby', 'rot13', 'rotn']
+++

Caesar cipher is the simplest and most widely known encryption technique. It is also known as **Caesar's cipher**, **shift cipher**, **Caesar's code**, **Caesar shift**, or **ROT N** (**ROT13** is the most famous one, shifting letters by 13).

It is very simple because it just works for letters between **A** and **Z**, ignoring all special characters, such as dots, whitespaces, question marks, and special letters, like **Ç** or **Á**.

Starting our implementation, we need to create a class that will know what we want to cipher and how many rotations we will do.

{{< gist mfbmina 5396859b9f4fad4723bce338d24a7ec7 >}}

Having done that, we need to shift how many chars we want, so we need to add the **shift** method:

{{< gist mfbmina 7589f0307cf727520452fa644d230e8c >}}

As you can see, we have a small problem at this point. If the **new_byte** (the old byte plus how many bytes do you want to rotate), is bigger than **Z** or **z**, we must start shifting it from **A** or **a** again. When this happens, we decrease one, because we have to take the initial char into consideration.

In this example, **initial_byte** is the byte for **A** or **a** and **limit_byte** is the byte for **Z** or **z**. In this way, we always rotate just between the letters.

So, we have to discover how we can find the bytes for each char inside of a **string**. Diving into Ruby, this is easily done by calling the method **bytes**. Knowing this, we have the following:
* **A** is represented by the byte 65.
* **Z** is represented by the byte 90.
* **a** is represented by the byte 97.
* **z** is represented by the byte 122.

{{< gist mfbmina 1878ef26ceb77929eab8cb0e4a625f73 >}}

The next step is to find our **initial_byte** and **limit_byte** or if we should ignore that char. In order to do that, we will check if the bytes present inside of a given text are between the range bytes for the letters that we want, if not, we will just return the byte.

{{< gist mfbmina c3068267b78e21a8f746a5c05f8a2e02 >}}

The fastest way to convert these bytes into a string again is by calling pack. In short, pack converts the given *array* into a *string*. We also give **c** as a parameter to the method. In this case, **c** means an 8-bit signed integer, and * means the whole array should be converted.
Now we are able to cipher a message using the Caesar technique, if we want to decipher it, we should shift it in the opposite direction, like here:

{{< gist mfbmina c3068267b78e21a8f746a5c05f8a2e02 >}}

It takes the same idea of the shift method, but does it in the "other" direction.

The **decipher's** method should look exactly the same as the **cipher**. Just duplicate **cipher** and change the method signature to **decipher**.

So, let's try our code!

{{< gist mfbmina 177a26ee79ee2b116d5aa82e52598e01 >}}

You also can **decipher the text**, like the following:

{{< gist mfbmina e3682c77b296abd8861f16dcb69b88d4 >}}

An interesting thing happens when you use rotation equal to 13. Between A and Z, we have exactly 26 letters, so does not matter if you call cipher or decipher, it will return the same result.

{{< gist mfbmina 957ca139dbbc316a31d48322059f2114 >}}

To have different results, you need to change the rotation to another value!

Unfortunately, this solution does not work for strings with multi-byte characters. Thanks for some guys on Slack that notice that! If you want a solution that works with multi-byte chars, you should implement a solution based on mod division.

This is the first post about cipher, my idea is to implement some techniques and try to easily explain it here. If you want to take a look at the whole code, please check at **[Github](https://github.com/mfbmina/cipher_studies/blob/master/caeser.rb)**.

You also can find me on **[Twitter](https://twitter.com/mfbmina)**, **[Github](https://github.com/mfbmina)**, or **[LinkedIn](https://www.linkedin.com/in/mfbmina/).**
