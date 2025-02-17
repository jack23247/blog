There is something I really like about substitution ciphers: they are simple, enjoyable, and somewhat effective if you only have a pen and some paper. Their premise is very simple, but they can be broken by brute-forcing with little effort; still, I believe they are quite interesting.

## Substitution Tables

The simplest form of substitution cipher is a substitution table, whic uses a pre-shared keyphrase, which may be a random string of letters or a page from a book, along with a rule used to derive the keyphrase. 

Let's make a simple example. Suppose you have the following key phrase:

> The quick brown fox jumps over the lazy dog

I chose this because it contains every letter in the alphabet; using a real phrase is fun but requires an extra step, since we need to mark every letter's first occurrence and throw away the rest.

```
xxx xxxxx xxxxx x x x xxx  x       x xx x x
The quick brown fox jumps over the lazy dog
```

The same method can be used to derive a keyphrase from a book. For example, I might tell you that the keyphrase can be derived from Part 2 of Ray Bradbury's Fahrenheit 451, skipping every other word.

> I didn't try doing it as I'd need to dig out my copy to do so, and there may be small differences between each copy. Due to how the distribution o

If we throw away every duplicated letter, we obtain the following:

```
Thequickbrownfxjmpsvlazydg 
12345678911111111112222222
         01234567890123456
```

Now we can build a substitution table by assigning each letter its index in the keyphrase:

| Index | Letter | Key Index | Cipher |
| ----- | ------ | --------- | ------ |
| 1     | `a`    | 22        | `v`    |
| 2     | `b`    | 9         | `i`    |
| 3     | `c`    | 7         | `g`    |
| 4     | `d`    | 25        | `y`    |
| 5     | `e`    | 3         | `c`    |
| 6     | `f`    | 14        | `n`    |
| 7     | `g`    | 26        | `z`    |
| 8     | `h`    | 2         | `b`    |
| 9     | `i`    | 6         | `f`    |
| 10    | `j`    | 16        | `p`    |
| 11    | `k`    | 8         | `h`    |
| 12    | `l`    | 21        | `u`    |
| 13    | `m`    | 17        | `q`    |
| 14    | `n`    | 13        | `m`    |
| 15    | `o`    | 11        | `k`    |
| 16    | `p`    | 18        | `r`    |
| 17    | `q`    | 4         | `d`    |
| 18    | `r`    | 10        | `j`    |
| 19    | `s`    | 19        | `s`    |
| 20    | `t`    | 1         | `a`    |
| 21    | `u`    | 5         | `e`    |
| 22    | `v`    | 20        | `t`    |
| 23    | `w`    | 12        | `l`    |
| 24    | `x`    | 15        | `o`    |
| 25    | `y`    | 24        | `x`    |
| 26    | `z`    | 15        | `g`    |

Once we've obtained the substitution table, we can encode and decode phrases at will. For example:

> Impressive. Very nice.

...becomes:

> FQRJCSSFTCTCJXMFGC

Decoding the phrase is as easy as substituting back the letters in the reverse order.

This method is trivial, but it's weak to frequency analysis as it always maps one letter to another; this problem is solved by polyalphabetical substitution which was introduced by Bellaso in the 1500s.

## The Bellaso Cipher

The Bellaso cipher fixes that by changing the alphabet for each letter.

## One-Time Pads

OTP Ciphers are technically unbreakable, but they rely on a Pre-Shared Key (PSK). OTP ciphers were used extensively in the past, and can still be used in cases where the ciphered message is 