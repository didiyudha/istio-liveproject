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