# Create a Self Signed Certificate


## Create root CA (Do this once)

### Create key for root CA
```bash
# Generate the private key for CA, which is used to sign certificates from others.
# Remove `-des3` if dont want to protect the key with password.
openssl genrsa -des3 -out root_ca.key 4096
```

### Create root CA certificate and self sign it
```bash
openssl req -x509 -new -nodes -key root_ca.key -sha256 -days 1024 -out root_ca.crt
# Put this CA cert to the trust store of each server.
```

## Create certificate for each server (Do this for each server)

### Create key for certificate
```bash
openssl genrsa -out my.server.com.key 2048
```

### Create certificate sign request
```bash
# Be careful when setting Commn Name, it must be the same as the server IP or URL.
openssl req -new -key my.server.com.key -out my.server.com.csr
```

### Sign the CSR with root CA
```bash
openssl x509 -req -in my.server.com.csr -CA root_ca.crt -CAkey root_ca.key -CAcreateserial -out my.server.com.crt -days 500 -sha256
```

## More openssl commands reference:
https://www.sslshopper.com/article-most-common-openssl-commands.html

