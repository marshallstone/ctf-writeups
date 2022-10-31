# DENJI EX-MAKIMA
## Category: Reverse Engineering

Nick is a huge anime fan. Recently, he started watching チェンソーマン and got really fascinated by it.
He couldn't wait for the new episodes, and downloaded an application which would "apparently" get him all the episodes straight away.
Sadly, it turned out to be a malware, and encrypted all the files in his system.
He needs your help. Provided is the malware and one of his files.

WARNING! THIS CHALLENGE CONTAINS A LIVE MALWARE

We are provided with two files: "FILE.fun" and "ChainsawMan_Downloader.zip (contains a binary)".

## Method:

Looking at the contents of `file.fun` I see the string:

`%AÊ§8ÀÍ,gDÝeG!nÆÖ#ößÏòDª€}ÿÿèwýW/0>ücÀn`

I assume this is the encrypted flag since the malware encrypted all of the files with the `.fun` extension.

Now I want to look inside of `ChainsawMan_Downloader.exe`.

`file ChainsawMan_Downloader.exe`
`ChainsawMan_Downloader.exe: PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows...`

The output from `file` shows that I am working with a binary compiled in .NET. My usual goto tool Ghidra will not work for the .NET binary so I load the binary
into dnSpy, a .NET disassembler inside of my Windows 10 virtual machine.

The C# code is very readable and it does not take long to find the function that encrypts the files on our operating system:

`private static bool EncryptFile(string path, string encryptionExtension)`

Inside of the function I see the following lines of code:

```
aesCryptoServiceProvider.Key = Convert.FromBase64String("OoIsAwwF32cICQoLDA0ODe==")
aesCryptoServiceProvider.IV = new byte[]
{
    0,
    1,
    0,
    3,
    5,
    3,
    0,
    1,
    0,
    0,
    2,
    0,
    6,
    7,
    6,
    0
}
```

I have now found the private key and initialization vector used to encrypt the file using AES encryption.

My final step is writing a script to decrypt the file from the beginning that contains our flag:

```python
import base64
from Crypto.Cipher import AES

iv = bytes([0, 1, 0, 3, 5, 3, 0, 1, 0, 0, 2, 0, 6, 7, 6, 0])
key = base64.b64decode('OoIsAwwF32cICQoLDA0ODe==')

ct = open('file.fun', 'rb').read()
dec = AES.new(key, AES.MODE_CBC)

print(dec.decrypt(iv + ct))
```

The script outputs the flag contents. 
