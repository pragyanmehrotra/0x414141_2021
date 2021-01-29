# Challenge Name: Encrypt0r

## First Glance

We are given a remote service, and no other description. Thus we nc that service and observe the following repsonse.

```
Heres the flag!
248460643464675800653780615843208617730874812788255456931910
Enter a value for me to encrypt
> 
```

## Approach

Obviously, the flag is encrypted so I didn't even bothered decrypting it directly. Rather I wanted to first check out the encryption service before I get disconnected. 

So, a total black blox no hints at all, we need to give it a "value" for encryption. So it's highly likely (more like obvious) that the flag is encrypted using the encryption technique used to provide the encryption service.

Now, entering random values, will just result in some random output. So we try some trivial cases and draw some conclusions and possible candidates for encryption.

```
Enter a value for me to encrypt
> 2
15316765420522776956567713808649930958674151786266484041867
```
We see that for a small value like 2 the output is very large and comparable to the encrypted flag, when I say comparable I mean - O(encrypted(flag)) ~ O(encrypted(2)). The other outputs for random values were also comparable to the encrypted flag.

From this we conclude that there must be some modular operation guiding the calculations.

```
Enter a value for me to encrypt
> 1
1
```
```
Enter a value for me to encrypt
> 0
0
```

Encrypting 1 results in 1 and 0 results in 0. From these 2 trivial cases we can conclude that no noise/constant is being added to the value and the value isn't multiplied by a constant. Therefore it must be of some x^y mod n form.

Therefore, to derive meaningful information we try more cases, now, our first priority is to find out the modulo. So we try a tricky input i.e. -1.

```
Enter a value for me to encrypt
> -1
722316164267119792405604455117204641944448497312096064056838
```
GREAT! From this we can infer that, y is odd and the value we recieved is -1 mod n = n-1. Thus from this we derive - 

`n = 722316164267119792405604455117204641944448497312096064056838 + 1 = 722316164267119792405604455117204641944448497312096064056839`

We can quickly check that this isn't a prime number. But since the number is small we are able to quickly factorize it using yafu.

```
===============================================================
======= Welcome to YAFU (Yet Another Factoring Utility) =======
=======             bbuhrow@gmail.com                   =======
=======     Type help at any time, or quit to quit      =======
===============================================================

>> factor(722316164267119792405604455117204641944448497312096064056839)

factoring 722316164267119792405604455117204641944448497312096064056839

div: primes less than 10000
fmt: 1000000 iterations
rho: x^2 + 1, starting 1000 iterations on C60 
rho: x^2 + 3, starting 1000 iterations on C60 
rho: x^2 + 2, starting 1000 iterations on C60 
pp1: starting B1 = 20K, B2 = 1M on C60, processed < 1000003
pp1: starting B1 = 20K, B2 = 1M on C60, processed < 1000003
pp1: starting B1 = 20K, B2 = 1M on C60, processed < 1000003
pm1: starting B1 = 100K, B2 = 5M on C60, processed < 5000011
ecm: 25 curves on C60 input, at B1 = 2K, B2 = 200K
ecm: 90 curves on C60 input, at B1 = 11K, B2 = 1100K
ecm: 4 curves on C60 input, at B1 = 50K, B2 = 5M

starting SIQS on c60: 722316164267119792405604455117204641944448497312096064056839

==== sieving in progress (1 thread):    3718 relations needed ====
====           Press ctrl-c to abort and save state           ====
2734 rels found: 1461 full + 1273 from 13585 partial, (4596.09 rels/sec)

SIQS elapsed time = 4.1334 seconds.
Total factoring time = 11.3080 seconds


***factors found***

PRP30 = 730573343524370733896529924509
PRP30 = 988697672409703112206678896371
```

Now, this encryption scheme seems extremely close to [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)). And since even for a small input such as 2 we get a large output. Thus, we suspect that the exponent used is probably 65537 (This is the most common exponent used for RSA encryption.). Our suspicions proved to be correct and now we can easily decrypt the flag (We checked it by comparing the output recieved locally by encrypting 2 and from the black box service).

```python
#env python2.7
import gmpy
c = 248460643464675800653780615843208617730874812788255456931910
p = 730573343524370733896529924509
q = 988697672409703112206678896371
phi = (p-1)*(q-1)
e = 65537
d = gmpy.invert(e, phi)
m = pow(c, d, p*q)
print ('flag{' + hex(m)[2:-1].decode('hex') + '}')
```                           

`flag{y0u_d0nt_n33d_4nyth1ng}`