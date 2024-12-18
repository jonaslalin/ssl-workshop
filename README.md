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
