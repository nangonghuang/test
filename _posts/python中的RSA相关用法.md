---
title: python中的RSA相关用法
date: 2018-07-24 02:03:20
tags: string类型公钥,RSA，公钥解密
categories: python
---

工作需要，用到了RSA算法，因为是python+flask搭建的服务器，所以也需要使用RSA相关的功能，但是网上搜了一圈，大都是在说 rsa 和 pycryptodome 这两个库。主要问题在于，我拿到的是string格式的公匙，而不是.pem文件，卡在了生产public key这个阶段。。<!--more-->



虽然 [RSA原理](https://nangonghuang.github.io/2018/07/30/RSA%E5%8A%A0%E8%A7%A3%E5%AF%86%E7%AE%97%E6%B3%95%E5%8E%9F%E7%90%86/) 大致看了一遍，但是实际使用的时候还涉及到key的生成和解析，加解密数据的填充等等，实际的使用中需要客户端和服务器统一RSA加密所用的填充方式。

------

RSA加密常用的填充方式有下面3种：

1.RSA_PKCS1_PADDING 填充模式，最常用的模式

在BouncyCastle实现RSA的PKCS1V1.5模式中，如果是公钥加密信息（forEncryption=true)，密钥长度为1024位，那么输出的密文块长度为128个字节，输入的明文块长度为127-10,即输入的明文块最大是117位，如果输入的明文块小于117位，比如输入的明文块长度为64位，那么会对这个明文块进行补位，在明文块前添加一位的0x02字节（代表公钥加密）然后后面的52位为随机的字节，在补位的最后一位，{即52（117-64-1），从零开始的},添加一位的字节0x00,在补位的后面添加实际的明文块。

这样做的目的就是使得明文块转化成与module差不多的大整数。

如果是私钥加密（forPrivateKey=true),密钥长度为1024位，那么输出 的密文块长度也是128字节，输入的明文块的长度为127-10，即输入的明文块最大是117位，如果输入的明文块小于117位，比如输入的明文块长度为64位，那么对这个明文块进行补位，在明文块前添加一位的0x01字节（代表私钥加密），然后在后面的52位为字节0xff，在最后一位｛即52（117-64-1）,从零开始)，添加一位的字节0x00，在补位的后面添加时间的明文块。

根据这个要求，对于512bit的密钥，　block length = 512/8 – 11 = 53 字节

2.RSA_PKCS1_OAEP_PADDING

输入：RSA_size(rsa) – 41

输出：和modulus一样长

3.for RSA_NO_PADDING　　不填充

输入：可以和RSA钥模长一样长，如果输入的明文过长，必须切割，　然后填充

输出：和modulus一样长

跟DES，AES一样，　RSA也是一个块加密算法（ block cipher algorithm），总是在一个固定长度的块上进行操作。

但跟AES等不同的是，　block length是跟key length有关的。

每次RSA加密的明文的长度是受RSA填充模式限制的，但是RSA每次加密的块长度就是key length。

需要注意：

假如你选择的秘钥长度为1024bit共128个byte：

1.当你在客户端选择RSA_NO_PADDING填充模式时，如果你的明文不够128字节

加密的时候会在你的明文前面，前向的填充零。解密后的明文也会包括前面填充的零，这是服务器需要注意把解密后的字段前向填充的

零去掉，才是真正之前加密的明文。

2.当你选择RSA_PKCS1_PADDING填充模式时，如果你的明文不够128字节

加密的时候会在你的明文中随机填充一些数据，所以会导致对同样的明文每次加密后的结果都不一样。

对加密后的密文，服务器使用相同的填充方式都能解密。解密后的明文也就是之前加密的明文。

3.RSA_PKCS1_OAEP_PADDING填充模式没有使用过， 他是PKCS#1推出的新的填充方式，安全性是最高的，

和前面RSA_PKCS1_PADDING的区别就是加密前的编码方式不一样。

------

回到string类型公匙的问题上来。虽然加密密匙有多种格式，比如jar包里面的校验文件，中间还有什么乱七八糟的RSA PKCS#1,PKCS#8,X.509之类的东西与这里说的内容无关，暂时忽略掉。但我们在RSA里面通常遇到的公匙通常有两种类型，一种是.pem的文件，另外一种是纯string类型的公匙。大致的看过文档之后(官方文档的代码也有错误跑不起来，非常误导....), pycryptodome 和 rsa 库的getting started文档并没有很明确的说明，
1. pem文件类型的公匙的两种格式：

    ```
    -----BEGIN RSA PUBLIC KEY-----
    MIIBCgKCAQEAhU9EQ3z/zNCH+68tTN6Vkhnq8jaCx+9iT+zfq6rmwGzdNcErkt8x
    gqD5iuuQmpnFtF0fM3aKyCdztbYcJcscggceg9Wf14Nu5xitWsG47Mpd4M13RIGX
    0p5lMuE2zI4x5YCUSsTLV9vNtRK73tnUq8HYcHrRCK+RmylRcXvyIDjpVcGbwT37
    MebFuufCGqo0rYfZl7bb4kDMgKk5gULGBDxc5tz7YktMXQ0FOa7g9WXsxIWmoy0q
    OUMZVyHY+55b8vGUT2EevqrpusrnUJvwYRTtTiHHU+ftSV3VwY+1g38CWTk148Lg
    UL0pIrnVxjPKUbUuoHA+YNbLL/BLpJr7jwIDAQAB
    -----END RSA PUBLIC KEY-----

    ```

    上面是一种类型的pem文件公匙的内容(这里可以叫做 标准pem 格式)，到具体的库来说，就是rsa库所产出的公匙：
    ```python
    import rsa

    (pubkey, privkey) = rsa.newkeys(2048)

    # 保存密钥
    with open('public.pem','w+') as f:
        f.write(pubkey.save_pkcs1().decode())

    with open('private.pem','w+') as f:
        f.write(privkey.save_pkcs1().decode())
    ```
    而crypto库产生的.pem文件格式为:(这里可以叫做 openssl 格式)
    ```
    -----BEGIN PUBLIC KEY-----
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA59JFc2huRupfSvTmI96e
    VuAWRAIziNG/sTm0Z+XvWqWFMk4s9WIAu9Pi1d1VKhhwJoUSZKdFvMtKxQ2nACOU
    u01ykiqKmxYpU2jgKrKdAuM4R9hMnp5eqN7ql1zg5PA/cF07AM0nG/Z2y6XZUSz8
    mnAv7F0ePf0Ba1CmUAk2r+VwxA2rvpR0HM87T2Qq8b3ZieUuCyp0bxtQmHcbSAEV
    ALHSucFxPVHKQ7b/cjp5Teh4yWJnyhAOU8rphFY0fGehQpWsKTWTZFlxm+TEcH7c
    9mlFu290z0DE4xzYHv7kKWhConqJAphcNAExm7j/qhvQF27DT90tj0oWtjz/tCNj
    GQIDAQAB
    -----END PUBLIC KEY-----
    ```
    产生的方法则是:
    ```python
    from Crypto.Random import get_random_bytes
    import base64
    from Crypto.Signature import pkcs1_15
    from Crypto.Hash import SHA256,SHA1
    from Crypto.PublicKey import RSA

    key = RSA.generate(2048)
    private_key = key.export_key()
    file_out = open("private.pem", "wb")
    file_out.write(private_key)

    public_key = key.publickey().export_key()
    file_out = open("public.pem", "wb")
    file_out.write(public_key)
    ```
    不过这两种pem格式，都是同一种构成规则，就是
    ```
    前缀行
    对DER格式的公匙进行base64后的结果
    后缀行
    ```
2. DER格式的公匙：
    DER格式的公匙，通常是byte字节流。我们所遇到的string类型的公匙，就是对这种字节流进行base64之后得到的字符串。而 openssl 格式 和 标准pem 格式 内容中所包含的 DER字节流，在没有前缀来区分的情况下，我们很难知道它是　openssl 格式 还是 标准pem 格式　中的那一段DER，而不同的类型，要用不同的处理方法来解析，这也是库方法解不出public key的最主要的原因之一。

    DER格式1:
    ```
    MIIBCgKCAQEAhU9EQ3z/zNCH+68tTN6Vkhnq8jaCx+9iT+zfq6rmwGzdNcErkt8x
    gqD5iuuQmpnFtF0fM3aKyCdztbYcJcscggceg9Wf14Nu5xitWsG47Mpd4M13RIGX
    0p5lMuE2zI4x5YCUSsTLV9vNtRK73tnUq8HYcHrRCK+RmylRcXvyIDjpVcGbwT37
    MebFuufCGqo0rYfZl7bb4kDMgKk5gULGBDxc5tz7YktMXQ0FOa7g9WXsxIWmoy0q
    OUMZVyHY+55b8vGUT2EevqrpusrnUJvwYRTtTiHHU+ftSV3VwY+1g38CWTk148Lg
    UL0pIrnVxjPKUbUuoHA+YNbLL/BLpJr7jwIDAQAB
    ```
    DER格式2：
    ```
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA59JFc2huRupfSvTmI96e
    VuAWRAIziNG/sTm0Z+XvWqWFMk4s9WIAu9Pi1d1VKhhwJoUSZKdFvMtKxQ2nACOU
    u01ykiqKmxYpU2jgKrKdAuM4R9hMnp5eqN7ql1zg5PA/cF07AM0nG/Z2y6XZUSz8
    mnAv7F0ePf0Ba1CmUAk2r+VwxA2rvpR0HM87T2Qq8b3ZieUuCyp0bxtQmHcbSAEV
    ALHSucFxPVHKQ7b/cjp5Teh4yWJnyhAOU8rphFY0fGehQpWsKTWTZFlxm+TEcH7c
    9mlFu290z0DE4xzYHv7kKWhConqJAphcNAExm7j/qhvQF27DT90tj0oWtjz/tCNj
    GQIDAQAB
    ```

最后给出完整代码:
> rsa库的可以根据解密过程自己写公钥解密的方法。而crypto库在解密的过程中也用到了
> pqn等参数，不太清楚为什么。所以就没有写出公钥解密的方法。

```python
import requests
from Crypto.Random import get_random_bytes
import base64
from Crypto.Signature import pkcs1_15
from Crypto.Hash import SHA256, SHA1
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP
import rsa

signature = 'n6KsB/rBh3DzRjWVuocnzqyPm6jxyLp31kF80PewabKKsgfO8OaYtWjdwBOLld0E9Ynx0MiH81KYfGrBZv6Ef+gtl55Kkkfezol3UheNtmOhYE+LprI78blh55SeYnixEmd439tLCfQNuvftDYSDkEnlMXZ6Emz8X0J1AnfhUHU='
rsa_key_str = 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDDnI3kW16XWPxx2y8vJwX58/3gNqpe3sgxIBjwhY8HKAbrZOHffcpkPYPO2uZZZSchIUd05E7x3daRbD6dmMVjbYzrs7U9L0m+5l8yKvNdVa9wMG3iWG/4a/e6V8tCDXuTx0i14MZoRYWmZBKwuWDkZCBtd1hq4xiU4fvMRfOg9wIDAQAB'
message = 'encryp_data=Lziso+i0rAjZSZMh/OfdVkACvSVLLUHZVkiLkqOEMGOWPuXCw65y1jr8Az5rC1H0NxkWtPKlSTMHxBM8dSnP1cbOqfdgHSiL5nVZq8k9yXvcShh4OfwZACvikDjDRsQo9nL+G4MjB7q34N5H/8pBZeN4cg9DuCg7ycXF0YG5V+w=&extends_info_data=当前是最强的游戏扩展数据&game_area=当前是最强的游戏区服&game_level=49&game_orderid=X7123456789&game_role_id=x7id&game_role_name=我是最强的角色名称&sdk_version=2.0&subject=大元宝&xiao7_goid=18634202'
encryp_data = 'Lziso+i0rAjZSZMh/OfdVkACvSVLLUHZVkiLkqOEMGOWPuXCw65y1jr8Az5rC1H0NxkWtPKlSTMHxBM8dSnP1cbOqfdgHSiL5nVZq8k9yXvcShh4OfwZACvikDjDRsQo9nL+G4MjB7q34N5H/8pBZeN4cg9DuCg7ycXF0YG5V+w='

rsa_public_key = '''-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEAhU9EQ3z/zNCH+68tTN6Vkhnq8jaCx+9iT+zfq6rmwGzdNcErkt8x
gqD5iuuQmpnFtF0fM3aKyCdztbYcJcscggceg9Wf14Nu5xitWsG47Mpd4M13RIGX
0p5lMuE2zI4x5YCUSsTLV9vNtRK73tnUq8HYcHrRCK+RmylRcXvyIDjpVcGbwT37
MebFuufCGqo0rYfZl7bb4kDMgKk5gULGBDxc5tz7YktMXQ0FOa7g9WXsxIWmoy0q
OUMZVyHY+55b8vGUT2EevqrpusrnUJvwYRTtTiHHU+ftSV3VwY+1g38CWTk148Lg
UL0pIrnVxjPKUbUuoHA+YNbLL/BLpJr7jwIDAQAB
-----END RSA PUBLIC KEY-----
'''
rsa_public_key_str = 'MIIBCgKCAQEAhU9EQ3z/zNCH+68tTN6Vkhnq8jaCx+9iT+zfq6rmwGzdNcErkt8xgqD5iuuQmpnFtF0fM3aKyCdztbYcJcscggceg9Wf14Nu5xitWsG47Mpd4M13RIGX0p5lMuE2zI4x5YCUSsTLV9vNtRK73tnUq8HYcHrRCK+RmylRcXvyIDjpVcGbwT37MebFuufCGqo0rYfZl7bb4kDMgKk5gULGBDxc5tz7YktMXQ0FOa7g9WXsxIWmoy0qOUMZVyHY+55b8vGUT2EevqrpusrnUJvwYRTtTiHHU+ftSV3VwY+1g38CWTk148LgUL0pIrnVxjPKUbUuoHA+YNbLL/BLpJr7jwIDAQAB'


crypto_public_key = '''-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA59JFc2huRupfSvTmI96e
VuAWRAIziNG/sTm0Z+XvWqWFMk4s9WIAu9Pi1d1VKhhwJoUSZKdFvMtKxQ2nACOU
u01ykiqKmxYpU2jgKrKdAuM4R9hMnp5eqN7ql1zg5PA/cF07AM0nG/Z2y6XZUSz8
mnAv7F0ePf0Ba1CmUAk2r+VwxA2rvpR0HM87T2Qq8b3ZieUuCyp0bxtQmHcbSAEV
ALHSucFxPVHKQ7b/cjp5Teh4yWJnyhAOU8rphFY0fGehQpWsKTWTZFlxm+TEcH7c
9mlFu290z0DE4xzYHv7kKWhConqJAphcNAExm7j/qhvQF27DT90tj0oWtjz/tCNj
GQIDAQAB
-----END PUBLIC KEY-----
'''

crypto_public_key_str = 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA59JFc2huRupfSvTmI96eVuAWRAIziNG/sTm0Z+XvWqWFMk4s9WIAu9Pi1d1VKhhwJoUSZKdFvMtKxQ2nACOUu01ykiqKmxYpU2jgKrKdAuM4R9hMnp5eqN7ql1zg5PA/cF07AM0nG/Z2y6XZUSz8mnAv7F0ePf0Ba1CmUAk2r+VwxA2rvpR0HM87T2Qq8b3ZieUuCyp0bxtQmHcbSAEVALHSucFxPVHKQ7b/cjp5Teh4yWJnyhAOU8rphFY0fGehQpWsKTWTZFlxm+TEcH7c9mlFu290z0DE4xzYHv7kKWhConqJAphcNAExm7j/qhvQF27DT90tj0oWtjz/tCNjGQIDAQAB'


def crypto_generate_keys():
    key = RSA.generate(2048)
    private_key = key.export_key()
    file_out = open("private.pem", "wb")
    file_out.write(private_key)

    public_key = key.publickey().export_key()
    file_out = open("public.pem", "wb")
    file_out.write(public_key)


def rsa_generate_keys():
    (pubkey, privkey) = rsa.newkeys(2048)

    with open('public2.pem', 'w+') as f:
        f.write(pubkey.save_pkcs1().decode())

    with open('private2.pem', 'w+') as f:
        f.write(privkey.save_pkcs1().decode())


def rsa_get_pubkey(pub_string):
    try:
        pubkey = rsa.PublicKey.load_pkcs1(pub_string)
        print(pubkey)
    except:
        print('1 failed')
        try:
            pubkey = rsa.PublicKey.load_pkcs1(pub_string, format='DER')
            print(pubkey)
        except:
            print('2 failed')
            try:
                pubkey = rsa.PublicKey.load_pkcs1_openssl_der(pub_string)
                print(pubkey)
            except:
                print('3 failed')
                try:
                    pubkey = rsa.PublicKey.load_pkcs1_openssl_pem(pub_string)
                    print(pubkey)
                except:
                    print('4 failed')
                    try:
                        pubkey = rsa.PublicKey.load_pkcs1(
                            base64.b64decode(pub_string), format='DER')
                        print(pubkey)
                    except:
                        print('5 failed')
                        try:
                            pubkey = rsa.PublicKey.load_pkcs1_openssl_der(
                                base64.b64decode(pub_string))
                            print(pubkey)
                        except:
                            print('6 failed')
    if pubkey:
        print('rsa_get_pubkey success')
        return pubkey


def crypto_get_pubkey(pub_string):
    try:
        pubkey = RSA.import_key(pub_string)
        print(pubkey.n)
        print(pubkey.e)
    except:
        print('1 failed')
        try:
            pubkey = RSA.import_key(base64.b64decode(pub_string))
            print(pubkey.n)
            print(pubkey.e)
        except:
            print('2 failed')
    if pubkey:
        print('crypto_get_pubkey success')
        return pubkey


def crypto_sign_verify(pubkey_str,message_str,signature_str):
    '''
    如果signature是从外部获得的string格式，则需要base64解码一下。否则签名的时候直接用即可
    '''
    # prikey = RSA.import_key(open('private.pem').read())
    # missing_padding = 4 - len(message) % 4
    # if missing_padding:
    #     message += '='* missing_padding
    # h = SHA.new(message.encode())
    # signature = pkcs1_15.new(prikey).sign(h)

    pubkey = crypto_get_pubkey(pubkey_str)
    h = SHA1.new(message_str.encode())
    try:
        pkcs1_15.new(pubkey).verify(h, base64.b64decode(signature_str))
        print("The signature is valid.")
    except (ValueError, TypeError):
        print("The signature is not valid.")


def rsa_sign_verify(pubkey_str,message_str,signature_str):
    '''
    如果signature是从外部获得的string格式，则需要base64解码一下。否则签名的时候直接用即可
    '''
    # (pubkey, privkey) = rsa.newkeys(512)
    # message = 'Go left at the blue tree'
    # signature = rsa.sign(message.encode(), privkey, 'SHA-1')
    # message = 'Go right at the blue tree'

    pubkey = rsa_get_pubkey(pubkey_str)
    result = rsa.verify(message_str.encode(), base64.b64decode(signature_str), pubkey)
    print(result)

def rsa_encrypt_decrypt():
    (bob_pub, bob_priv) = rsa.newkeys(512)
    message = 'hello Bob!'.encode()
    crypto = rsa.encrypt(message, bob_pub)
    print(base64.b64encode(crypto).decode())
    message = rsa.decrypt(crypto, bob_priv)
    print(message.decode())


def get_max_out_len(key_n):
    '''
    根据key的长度获取可以一次解密输出的密文数据的长度
    密文数据的长度-11 就是一次解密的明文数据的最大长度
    '''
    bits = 0
    while key_n >> bits:
        bits += 1
    r, q = divmod(bits, 8)
    if q != 0:
        r += 1
    return r

def get_max_input_len(key_n):
    return get_max_out_len(key_n)

def crypto_decrypt_with_public():
    # crypto 不允许公钥解密，算法有点看不懂，所以也不太会改
    pubkey = crypto_get_pubkey(rsa_key_str)
    cipher_rsa_decrypt = PKCS1_OAEP.new(pubkey)
    if len(base64.b64decode(encryp_data)) <= get_max_out_len(pubkey.n):
        data = cipher_rsa_decrypt.decrypt(base64.b64decode(encryp_data))
    else:
        print('需要分段解密')
        pass
    print(data)

def decrypt_with_public_key(crypto, pub_key):
    blocksize = rsa.common.byte_size(pub_key.n)
    encrypted = rsa.transform.bytes2int(crypto)

    # blind_r = rsa.randnum.randint(pub_key.n - 1)
    # blinded = (encrypted * pow(blind_r, pub_key.e, pub_key.n)) % pub_key.n 
    # decrypted = rsa.core.decrypt_int(blinded, pub_key.e, pub_key.n)
    decrypted = rsa.core.decrypt_int(encrypted, pub_key.e, pub_key.n)

    cleartext = rsa.transform.int2bytes(decrypted, blocksize)

    # If we can't find the cleartext marker, decryption failed.
    if cleartext[0:2] != b'\x00\x01':  #这里表示是私钥加密，公钥加密这里是\x02
        raise Exception('Decryption failed,the message is not encrypted by private key')

    # Find the 00 separator between the padding and the message
    try:
        sep_idx = cleartext.index(b'\x00', 2)   #寻找填充数据的最后一个byte
    except ValueError:
        raise Exception('Decryption failed,cannot find end of padding space')
    return cleartext[sep_idx + 1:]

def rsa_decrypt_with_public():
    pubkey = rsa_get_pubkey(rsa_key_str)
    if len(base64.b64decode(encryp_data)) <= get_max_out_len(pubkey.n):
        decrypt = decrypt_with_public_key(base64.b64decode(encryp_data), pubkey)
        decrypt_dict = {}
        for pair_str in decrypt.decode().split('&'):
            (key, value) = pair_str.split('=')
            decrypt_dict[key] = value
    else:
        print('需要分段解密')
        pass

if __name__ == '__main__':
    processes = [rsa_public_key,rsa_public_key_str,crypto_public_key,crypto_public_key_str]
    for process in processes:
        pubkey = rsa_get_pubkey(process)
        pubkey = crypto_get_pubkey(process)
    rsa_decrypt_with_public()
   
```

最后的总结：

pycryptodome 不支持公钥解密

rsa 支持公钥解密

两者都支持公钥加密私钥解密，验证签名等
