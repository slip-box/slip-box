# Crypto

### SSL 和OpenSSL

Secure Sockets Layer，这是其全名，他的作用是协议，定义了用来对网络发出的数据进行加密的格式和规则

OpenSSL是对SSL协议的实现

Node.js 是完全采用 OpenSSL 进行加密的，其TLS HTTPS 服务器模块和 Crypto 加密模块都是通过 C++ 在底层调用 OpenSSL 。

## 加密

- 信息摘要是一些采用哈希算法的加密方式：MD5 和 SHA (建议采用更稳定的 SHA1, MD5通过查表大法已经不再单向)。
- 非对称加密:DH | RSA | DSA | EC
- 对称加密：AES(128)【目前最推荐】 | DES(64) | Blowfish(64) | CAST(64) | IDEA(64) | RC2(64) | RC5(64)

## 总结:

密钥长度增加一位，暴力破解的难度指数级增加；

