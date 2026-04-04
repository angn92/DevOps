# Certificate and OpenSSL in K8s, Harbor, etc

1. Private key - used to prove identity, decrypt traffic (TLS handshake) and sign things (CSR)

Generate:
```bash
openssl genrsa -out server.key 2048
```

2. CSR - Certificate Signing Request
- public key
- identity
- signed by private key 
This is what send to CA like Let's encrypt and Sectigo

Create CSR:
```bash
openssl req -new -key server.key -out server.csr
```

3. Certificate 
- contains public key
- contains identity (domain)
- is signed by a CA

Example: server.crt
It looks like 
```bash
-----BEGIN CERTIFICATE-----
(base64 data)
-----END CERTIFICATE-----
```

4. PEM, CRT, CER

- PEM format (PEM = Base64 + headers)
Extensions: .pem, .crt, .cer

5. Certificate Authority (CA)

A CA ( like Let's Encrypt, Sectigo or own CA - it can be any VM)
- sign certificate
- proves you own the domain

Types of CA:
a) Root CA - top-level trusted authority
b) intermediate CA - used by root to sign certs
c) leaf (server cert) - your actual certificate 

6. Certificate chain
When client connects: our cert -> Intermediate -> Root  it's called a chain of trust
Files you’ll see:
- server.crt → your cert
- ca.crt → CA cert
- fullchain.pem → server + intermediate(s)