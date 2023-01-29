# Expose the Online Boutique Store Securely on an HTTPS Port

## Objective
```
- Configure the istio ingress gateway to accept TLS encrypted traffic on port 443 so that users can transact securely at the online boutique store
```

## Create self-signed root certificate that will be used to sign the TLS certificate for the ingress gateway
```
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 \
  -subj '/O=boutiquestore Inc./CN=boutiquestore.com' \
  -keyout root.key -out root.crt
```

## Generate a private key and certificate signing request (CSR), which will be signed by the root CA and used for the TLS handshake
```
openssl req -out server.csr -newkey rsa:2048 -nodes \
  -keyout server.key -subj "/CN=marketplace.boutiquestore.com/O=boutique store"
```

## Sign the generated CSR from the previous step using the root CA from the first step. With this command, we now have all the pieces to create a Kubernetes secret that the Istio ingress gateway can use.
```
openssl x509 -req -days 365 -CA root.crt -CAkey root.key \
    -set_serial 0 -in server.csr -out server.crt
```

## Create the Kubernetes secret using the generated server private key and public key above.
```
kubectl -n istio-system create secret tls online-boutique-tls-credential \
    --key server.key --cert=server.crt
```

## Configure the TLS gateway resource

```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: frontend-gateway
  namespace: online-boutique
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        name: https
        number: 443
        protocol: HTTPS
      tls:
        mode: SIMPLE
        credentialName: online-boutique-tls-credential
      hosts:
      - "marketplace.boutiquestore.com"
    - port:
        name: http
        number: 80
        protocol: HTTP
      hosts:
      - "*"
```

## Deliverable
```
openssl s_client -connect $INGRESS:443 -servername marketplace.boutiquestore.com -CAfile root.crt
```
```
❯ openssl s_client -connect $INGRESS:443 -servername marketplace.boutiquestore.com -CAfile root.crt
CONNECTED(00000003)
depth=1 O = boutiquestore Inc., CN = boutiquestore.com
verify return:1
depth=0 CN = marketplace.boutiquestore.com, O = boutique store
verify return:1
---
Certificate chain
 0 s:/CN=marketplace.boutiquestore.com/O=boutique store
   i:/O=boutiquestore Inc./CN=boutiquestore.com
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIC7jCCAdYCAQAwDQYJKoZIhvcNAQEFBQAwOTEbMBkGA1UECgwSYm91dGlxdWVz
dG9yZSBJbmMuMRowGAYDVQQDDBFib3V0aXF1ZXN0b3JlLmNvbTAeFw0yMzAxMjkw
OTEwNTFaFw0yNDAxMjkwOTEwNTFaMEExJjAkBgNVBAMMHW1hcmtldHBsYWNlLmJv
dXRpcXVlc3RvcmUuY29tMRcwFQYDVQQKDA5ib3V0aXF1ZSBzdG9yZTCCASIwDQYJ
KoZIhvcNAQEBBQADggEPADCCAQoCggEBAL/xnXLbffYi0XadpEgESDYM2YB5O0Rw
Sz+LPpWCwnCyOE/1459NPdb3tIWkwwrSbgyY4gNJ3C0rlR9R94OwphK1fd4OeIng
6278vjurkPjSpcFfU9EjLSCJ4tsRTsaLko9rIVGR2VxjG7hA7Uz6W9ht36WJaPnQ
tCM+6s31cYeF63NBZdgnkUmapgGBNeKuIvbdUCImxloKYqKdekQjYBau/awKXeng
mEjO83Mdi9aG6HrNTjtoWyZkgY8buFif2jFw/+jCaCrgjeN7u/Jh4rN6n1WxqfSs
fsm88iul8wXpcs6in9Di6IF14d6AvaDTJ8mEZuyePCV1GCzE4dOD68ECAwEAATAN
BgkqhkiG9w0BAQUFAAOCAQEAwGtub0gc4gK16+q1BYXmy0blQQMfq+AnvEpqt+Kr
Jg8ZtCRMnHqGU5ZM/YT5e/iNM8/1aVAZtUiws1LnBBhibD5cU8iExiunXQgdPgfN
RQdD+UMAL0Q22aJf84ABPOpRorvrYMVvIh8hmOl/uKyjlo88Dx10K+iChLl9lq7c
2+0iWOLxh0PpjgjS/pN4XQktR8miy9P3fYi8e78jYmTteBPxEF/gdE2WOhY4gmxP
kEKYqoWF1leqnGrBVqyDAPGkBlRTaJWrGi13ojuwXpxFEElT6ogPneDO17d80TZ0
7p6ltNyMmizRsKFMt3D2uBoTe9FBocpFbOuPjQs9tXAT6w==
-----END CERTIFICATE-----
subject=/CN=marketplace.boutiquestore.com/O=boutique store
issuer=/O=boutiquestore Inc./CN=boutiquestore.com
---
No client certificate CA names sent
Server Temp Key: ECDH, X25519, 253 bits
---
SSL handshake has read 1397 bytes and written 319 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-CHACHA20-POLY1305
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-CHACHA20-POLY1305
    Session-ID: 276985BED35CD4704BAC5BA3C7E6B351B86C757F7FDE8F73D2CB88217D5B6326
    Session-ID-ctx: 
    Master-Key: FCEDEEBA467A272511578B4180884F5F94A4E3CA4CDB2682FEBC20D84E12C3C8B1025D66E69CE399E63C9084F20AC2BC
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 6a 19 49 1a d9 12 61 a0-93 fe 45 c4 a7 07 e6 e5   j.I...a...E.....
    0010 - d3 dc 41 f1 1b 24 eb 57-a7 b6 30 83 61 e0 92 b4   ..A..$.W..0.a...
    0020 - 47 1c 69 94 72 d6 e9 e1-fb ef c7 8b 43 d8 3e 66   G.i.r.......C.>f
    0030 - 70 89 21 4c 82 16 6f 35-18 13 3a e7 9c c6 a8 69   p.!L..o5..:....i
    0040 - 75 8a c8 59 95 98 a1 b3-17 9b 88 47 f3 3d 99 e8   u..Y.......G.=..
    0050 - 71 3d 1e 26 72 63 59 ba-82 c4 c2 b9 bc 40 5b 3d   q=.&rcY......@[=
    0060 - 0d 9d 44 f4 dd 5b 97 c2-96 01 4b dd e8 aa 82 6a   ..D..[....K....j
    0070 - af bb eb ed 8a 93 09 a5-89 e4 bd ee a3 23 62 ef   .............#b.
    0080 - a8 d8 6a 2a ee 94 54 db-d9 fb 1b 29 89 54 af c0   ..j*..T....).T..
    0090 - af 23 d0 8d 62 de d0 24-7a 28 09 2d 82 f5 b1 c5   .#..b..$z(.-....
    00a0 - 92 57 32 c9 91 8e b9 31-5c 13 e3 b9 23 6a ef d6   .W2....1\...#j..
    00b0 - 80 51 6a cf 6b c2 bd 9a-9c ce 05 e2 ed f3 b1 f4   .Qj.k...........

    Start Time: 1674988059
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
```