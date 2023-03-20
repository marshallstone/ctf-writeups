# keyexchange

### Category: Crypto
### Points: 120

We get a short problem description stating:

"Diffie-Hellman is secure right"

followed by a server:

nc keyexchange.wolvctf.io 1337

and a python script called challenge.py with the following contents:

```
#!/opt/homebrew/bin/python3

from Crypto.Util.strxor import strxor
from Crypto.Util.number import *
from Crypto.Cipher import AES

n = getPrime(512)

s = getPrime(256)

a = getPrime(256)
# n can't hurt me if i don't tell you
print(pow(s, a, n))
b = int(input("b? >>> "))

secret_key = pow(pow(s, a, n), b, n)

flag = open('/flag', 'rb').read()

key = long_to_bytes(secret_key)
enc = strxor(flag + b'\x00' * (len(key) - len(flag)), key)
print(enc.hex())
```

The program first prints s^a(modn), which is the public key for this key exchange. 

```
print(pow(s, a, n))
```

The program then asks for the b value, which is the private exponent.

```
b = int(input("b? >>> "))
```

This is an issue because we can trivially choose the value 1 as the exponent, evaluating the
private key to be the same as the public key. The program then xor's the private key and the flag, and returns 
the result. 

Connecting to the server:

```
153536822134698410826861879633201811424829134730955706160305579986850330073527731208828374018617207599715965826232778551746145213912829692315353893995264
b? >>> 1
758d0c8a570b82fa57457814889d96f4582ba3f99dca34e7ef25b7f411ba0696e1a6e8ba15de29152b792a38ad37f77bc98bb19189303c04203143ce7b3a8300
```

I wrote a python program to get the flag from these results:

```
from Crypto.Util.number import long_to_bytes
from Crypto.Util.strxor import strxor

pub = long_to_bytes(153536822134698410826861879633201811424829134730955706160305579986850330073527731208828374018617207599715965826232778551746145213912829692315353893995264)
enc = "758d0c8a570b82fa57457814889d96f4582ba3f99dca34e7ef25b7f411ba0696e1a6e8ba15de29152b792a38ad37f77bc98bb19189303c04203143ce7b3a8300"
enc = bytes.fromhex(enc)

pub += b'\x00' * (len(enc) - len(pub))
pt = strxor(pub, enc)

print(str(pt))
```

```
$ python3 dec.py
b'wctf{m4th_1s_h4rd_but_tru5t_th3_pr0c3ss}\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
```



