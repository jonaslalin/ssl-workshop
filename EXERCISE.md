# Exercise

1. Create a self-signed root certificate authority.
2. Create an intermediate certificate authority, signed by the root CA.
3. Create a domain certificate for example.com, signed by the intermediate CA.
4. Start an nginx server hosting example.com locally, with forced SSL, supplying the domain's key and certificate.
5. Use curl and the root CA to verify that a https request to example.com works as expected.

# Bonus Exercises

1. Add www.example.com to the domain certificate and redirect https://example.com to https://www.example.com.
2. Make sure the intermediate CA cannot create other certificate authoritys, only the root CA has this power.
3. Make a wildcard certificate for example.com, host multiple example.com subdomains with nginx, and verify using curl. Make sure nginx's SSL hostname routing works as expected.
4. Redirect http traffic to https traffic (http://example.com -> https://example.com).

# Resources

**N.B.: Read the fucking manual! Twice!**

1. https://docs.openssl.org/1.1.1/man1/genrsa/
2. https://docs.openssl.org/1.1.1/man1/req/
3. https://docs.openssl.org/1.1.1/man5/x509v3_config/ (IMPORTANT!!!)
4. https://docs.openssl.org/1.1.1/man1/x509/
5. https://docs.openssl.org/1.1.1/man1/verify/
6. https://hub.docker.com/_/nginx
7. https://docs.podman.io/en/stable/markdown/podman-run.1.html
8. https://curl.se/docs/manpage.html

# Hints

* The `-addext` argument for the certificate signing request does not work as expected in OpenSSL 1.1.1, the `-extfile` argument for the x509 command works.
* X509 V3 certificate extensions are required for CAs.
* Subject Alternative Name (SAN) have replaced the Common Name (CN) attribute (deprecated through https://www.rfc-editor.org/rfc/rfc2818.html).
* Curl can use a virtual `/etc/hosts` file with the `--resolve` argument (see https://curl.se/docs/manpage.html#--resolve).
