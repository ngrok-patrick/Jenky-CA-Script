# Jenky-CA-Script
Create a CA using OpenSSL and generate a Server certificate

Here's a sample script that utilizes OpenSSL to generate a Certificate Authority with the required extensions for ngrok to accept them. Additionally, it generates a Server certificate that can also be utilized.

```
#!/bin/sh

cat >extension_file.ext <<EOL
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth

subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost

EOL

# 1. Generate CA's private key and self-signed certificate
openssl req -x509 -newkey rsa:4096 -days 365 -nodes -keyout ca-key.pem -out ca-cert.pem -extensions ext -subj "/C=US/ST=main/L=last/O=ngrok/OU=geeks/CN=localhost/emailAddress=patrick@ngrok.com"

echo "CA's self-signed certificate"
openssl x509 -in ca-cert.pem -noout -text

# 2. Generate web server's private key and certificate signing request (CSR)
openssl req -newkey rsa:4096 -nodes -keyout server-key.pem -out server-req.pem -extensions ext -subj "/C=US/ST=maple/L=Florida/O=Macbook/OU=Computer/CN=localhost/emailAddress=patrickmacbook@ngrok.com"

# 3. Use CA's private key to sign web server's CSR and get back the signed certificate
openssl x509 -req -in server-req.pem -days 60 -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extension_file.ext

echo "Server's signed certificate"
openssl x509 -in server-cert.pem -noout -text
```
Once you have your keys, and created an Edge in ngrok, you can test the certificates like this:
```
curl https://yourEdge.ngrok.app --key server-key.pem --cert server-cert.pem
```
