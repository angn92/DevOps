# Certificate and OpenSSL in K8s, Harbor, Docker, etc

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


---

View CSR details:
```bash
openssl req -in server.csr -text -noout
```

Turn CSR into a cert:
```bash
openssl x509 -req -in server.csr -signkey server.key -out server.crt -days 365 # this create self-signed certificate so now we have serve.key and server.crt
```

now having crt file we can inspect:
```bash
openssl x509 -in server.crt -text -noout
```

Check expiration and validity:
```bash
openssl x509 -in server.crt -noout -dates
# output: Not Before / Not After
```

Check issuer:
```bash
openssl x509 -in server.crt -noout -issuer
```

Check fingerprint (optional for security):
```bash
openssl x509 -in server.crt -noout -fingerprint
```

Check SAN in certificate:
```bash
openssl x509 -in server.crt -noout -text | grep -A1 "Subject Alternative Name"
```

Vrify certificate against CA:
```bash
openssl verify -CAfile ca.crt harbor.crt
# Output should be: harbor.crt: OK
```

Quick checks:
```bash
openssl req -in server.csr -noout -subject # only subject

openssl req -in server.csr -noout -text | grep -A1 "Subject Alternative Name" # only SAN
```

To use self-signed certificate (Harbor for example) in Kubernetes need do this: manually trust it on every node to pull images for example.
for k3s:
```bash
sudo mkdir -p /etc/rancher/k3s/certs.d/harbor.example.com
sudo cp server.crt /etc/rancher/k3s/certs.d/harbor.example.com/ca.crt # rename it to ca.crt because it acts as its own CA
sudo systemctl restart k3s
```


---
1. Self-signed certificate
When I do:
```bash
openssl x509 -req -in server.csr -signkey server.key -out server.crt -days 365 # it means server.crt is signed by its own key
```

Added self-signed server.crt to all nodes corectly, Kubernetes, Docker, OpenShift can trust for example Harbor registry.

The trust model
When a container runtime (Docker / containerd) connects to a registry:
- It checks the TLS certificate chain
- Verifies that the certificate is issued by a trusted CA
- Validates the hostname matches the certificate

Self-signed = the cert signed itself, not root CA is known to the system example error: ```bash  x509: certificate signed by unknown authority ```

Solutions: Use server.crt as CA for nodes

Docker
```bash
sudo mkdir -p /etc/docker/certs.d/harbor.example.com
sudo cp server.crt /etc/docker/certs.d/harbor.example.com/ca.crt
sudo systemctl restart docker
```

Kubernetes: containerd
```bash
sudo mkdir -p /etc/rancher/k3s/certs.d/harbor.example.com
sudo cp server.crt /etc/rancher/k3s/certs.d/harbor.example.com/ca.crt
sudo systemctl restart k3s
```

2. Create private CA

A private Certificate Authority (CA) is:

- An internal CA that your organization controls
- Can issue certificates for your internal services (Harbor, k3s, internal apps)
- Works like a public CA (Sectigo, Let’s Encrypt) but only trusted inside your infrastructure


Commands to create a private CA:

1. Generate the CA private key
```bash
openssl genrsa -out ca.key 4096
```

2. Create the root CA certificate
```bash
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt -subj "/C=US/ST=CA/L=SF/O=MyOrg/CN=MyPrivateCA" # ca.crt root certificate
```


Extra commands for CA, For private CA, you could also include creating intermediate CA:
```bash
# Generate intermediate key
openssl genrsa -out intermediate.key 4096

# Create intermediate CSR
openssl req -new -key intermediate.key -out intermediate.csr -subj "/C=US/ST=CA/O=MyOrg/CN=IntermediateCA"

# Sign intermediate with root CA
openssl x509 -req -in intermediate.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out intermediate.crt -days 1825 -sha256
```


---
Example for Harbor with private CA:

I have ca.key and ca.crt (look above)

next: this steps to harbor.csr on harbor vm

Generate service key:
```bash
openssl genrsa -out harbor.key 2048
```

Create CSR with SAN (important!) ```bash harbor.csr.cnf ```

```INI
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = req_ext

[ dn ]
C  = US
ST = CA
L  = SF
O  = MyOrg
CN = harbor.example.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = harbor.example.com
DNS.2 = harbor
IP.1  = 10.0.0.5
```

Generate CSR:
```bash
openssl req -new -key harbor.key -out harbor.csr -config harbor.csr.cnf
```


Sign the CSR with your private CA:
```bash
openssl x509 -req -in harbor.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out harbor.crt -days 825 -sha256 -extfile harbor.csr.cnf -extensions req_ext
```

-CAcreateserial → creates ca.srl (serial number file)
-extfile and -extensions req_ext → include SANs

Now you have:

harbor.key → private key
harbor.crt → server certificate signed by your private CA
ca.crt → root CA certificate

Last steps:

Install CA on Kubernetes / Docker nodes

For Docker:
```bash
sudo mkdir -p /etc/docker/certs.d/harbor.example.com
sudo cp ca.crt /etc/docker/certs.d/harbor.example.com/ca.crt
sudo systemctl restart docker
```

For k3s / containerd:
```bash
sudo mkdir -p /etc/rancher/k3s/certs.d/harbor.example.com
sudo cp ca.crt /etc/rancher/k3s/certs.d/harbor.example.com/ca.crt
sudo systemctl restart k3s
```

In Harbor config:
```yaml
tls:
  certificate: /path/to/harbor.crt
  private_key: /path/to/harbor.key
```


---
Any commands:
```bash
# Generate intermediate key
openssl genrsa -out intermediate.key 4096

# Create intermediate CSR
openssl req -new -key intermediate.key -out intermediate.csr -subj "/C=US/ST=CA/O=MyOrg/CN=IntermediateCA"

# Sign intermediate with root CA
openssl x509 -req -in intermediate.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out intermediate.crt -days 1825 -sha256
```