# Generate TLS Certs for Secure Remote Access to Docker Engine

See [the Docker docs](https://docs.docker.com/engine/security/https/) and [Stefan Scherer's blog post](https://stefanscherer.github.io/protecting-a-windows-2016-docker-engine-with-tls/).

## CA

Create a Certificate Authority to sign other certs:

```
mkdir certs
cd certs

openssl rand -base64 32 > docker-ca.password

openssl genrsa -aes256 -passout file:docker-ca.password -out docker-ca-key.pem 4096

openssl req -subj "/C=UK/ST=COUNTY/L=CITY/O=Org/OU=." -new -x509 -days 3650 -passin file:docker-ca.password -key docker-ca-key.pem -sha256 -out docker-ca.pem
```

## Client

Issue a client cert for the Docker CLI, using the CA:

```
mkdir client
cd client

openssl genrsa -out key.pem 4096

openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile-client.cnf

openssl x509 -req -days 3650 -sha256 -in client.csr -CA ../docker-ca.pem -CAkey ../docker-ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf -passin file:../docker-ca.password
```

## Server

Issue a server cert for the Docker Engine, using the CA.

> Replace these with your own values:

```
hostname="rock64-01"
domain=".sixeyed"
ipAddress="192.168.2.189"
```

```
mkdir ../$hostname
cd ../$hostname

openssl genrsa -out server-key.pem 4096

openssl req -subj "/CN=$hostname$domain" -sha256 -new -key server-key.pem -out server.csr
echo subjectAltName = DNS:$hostname$domain,IP:$ipAddress,IP:127.0.0.1 >> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf

openssl x509 -req -days 3650 -sha256 -in server.csr -CA ../docker-ca.pem -CAkey ../docker-ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf -passin file:../docker-ca.password
```
