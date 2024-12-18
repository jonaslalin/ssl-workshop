# ssl-workshop

## Root Certificate Authority

### 1. Create a Private Key

```sh
openssl genrsa -out acmecorprootca.key 2048
```

### 2. Create a Certificate Signing Request

```sh
openssl req -out acmecorprootca.csr -key acmecorprootca.key -new -subj "/C=US/O=Acme Corp./CN=Acme Corp. Root Certificate Authority"
```

### 3. Inspect the Certificate Signing Request

```sh
openssl req -in acmecorprootca.csr -noout -text
```

### 4. Create a Self-Signed Certificate

```sh
openssl x509 -in acmecorprootca.csr -out acmecorprootca.crt -days 365 -signkey acmecorprootca.key -req -extfile acmecorprootca.ext
```

### 5. Inspect the Certificate

```sh
openssl x509 -in acmecorprootca.crt -noout -text
```

## Intermediate Certificate Authority

### 1. Create a Private Key and a Certificate Signing Request

```sh
openssl req -out acmecorpintermediateca.csr -new -keyout acmecorpintermediateca.key -newkey rsa:2048 -nodes -subj "/C=US/O=Acme Corp./CN=Acme Corp. Intermediate Certificate Authority"
```

### 2. Inspect the Certificate Signing Request

```sh
openssl req -in acmecorpintermediateca.csr -noout -text
```

### 3. Create a Certificate

```sh
openssl x509 -in acmecorpintermediateca.csr -out acmecorpintermediateca.crt -days 180 -req -CA acmecorprootca.crt -CAkey acmecorprootca.key -CAcreateserial -extfile acmecorpintermediateca.ext
```

### 4. Inspect the Certificate

```sh
openssl x509 -in acmecorpintermediateca.crt -noout -text
```

### 5. Verify the Certificate

```sh
openssl verify -CAfile acmecorprootca.crt acmecorpintermediateca.crt
```

## `example.com`

### 1. Create a Private Key and a Certificate Signing Request

```sh
openssl req -out examplecom.csr -new -keyout examplecom.key -newkey rsa:2048 -nodes -subj "/C=RU/O=Spyware Inc./CN=example.com"
```

### 2. Inspect the Certificate Signing Request

```sh
openssl req -in examplecom.csr -noout -text
```

### 3. Create a Certificate

**N.B.: Signed by the intermediate certificate authority.**

```sh
openssl x509 -in examplecom.csr -out examplecom.crt -days 30 -req -CA acmecorpintermediateca.crt -CAkey acmecorpintermediateca.key -CAcreateserial -extfile examplecom.ext
```

### 4. Inspect the Certificate

```sh
openssl x509 -in examplecom.crt -noout -text
```

### 5. Verify the Certificate

```sh
openssl verify -CAfile acmecorprootca.crt -untrusted acmecorpintermediateca.crt examplecom.crt
```

## nginx

### 1. Start a Web Server

```sh
HTTPS_PROXY=http://your.companyproxy.com:8080 \
podman run --publish 8443:443 \
           --rm \
           --security-opt label=disable \
           --volume ./default.conf:/etc/nginx/conf.d/default.conf \
           --volume ./examplecom.key:/etc/nginx/examplecom.key \
           --volume ./examplecomchain.crt:/etc/nginx/examplecomchain.crt \
           docker.io/library/nginx:1.27.3
```

### 2. Try It Out!

```sh
curl --cacert acmecorprootca.crt \
     --resolve example.com:8443:127.0.0.1 \
     --verbose \
     https://example.com:8443
```

**Expected output:**

```
* Added example.com:8443:127.0.0.1 to DNS cache
* Rebuilt URL to: https://example.com:8443/
* Hostname example.com was found in DNS cache
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to example.com (127.0.0.1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: acmecorprootca.crt
  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, [no content] (0):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=RU; O=Spyware Inc.; CN=example.com
*  start date: Dec 18 13:49:35 2024 GMT
*  expire date: Jan 17 13:49:35 2025 GMT
*  subjectAltName: host "example.com" matched cert's "example.com"
*  issuer: C=US; O=Acme Corp.; CN=Acme Corp. Intermediate Certificate Authority
*  SSL certificate verify ok.
* TLSv1.3 (OUT), TLS app data, [no content] (0):
> GET / HTTP/1.1
> Host: example.com:8443
> User-Agent: curl/7.61.1
> Accept: */*
> 
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, [no content] (0):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS app data, [no content] (0):
< HTTP/1.1 200 OK
< Server: nginx/1.27.3
< Date: Wed, 18 Dec 2024 14:48:11 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Tue, 26 Nov 2024 15:55:00 GMT
< Connection: keep-alive
< ETag: "6745ef54-267"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host example.com left intact
```
