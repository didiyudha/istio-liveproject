# Exposing Multiple Domains at the Ingress Gateway

## Objective
```
- Configure an Istio ingress gateway to expose multiple domains with unique certificates at port 443 so users can access different applications that are all exposed securely by the same gateway.
```

## Download script to generate TLS certificate
```
curl -L https://raw.githubusercontent.com/aspenmesh/liveproject/master/section_3/generate_tls_credentials.sh -o generate_tls_credentials.sh && chmod +x generate_tls_credentials.sh
```

##  Create Kubernetes secret with the TLS credentials
```
./generate_tls_credentials.sh boutiquestore '.com' 'httpbin' 'httpbin-tls-credential'
```

## Deploy httpbin application

```
kubectl label namespace default --overwrite "istio-injection=enabled"
```

```
kubectl apply -n default -f - << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: httpbin
---
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  labels:
    app: httpbin
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 80
  selector:
    app: httpbin
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpbin
      version: v1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      serviceAccountName: httpbin
      containers:
      - image: docker.io/kennethreitz/httpbin
        imagePullPolicy: IfNotPresent
        name: httpbin
        ports:
        - containerPort: 80
---
EOF
```

## Create Istio gateway configuration
```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: httpbin-gateway
  namespace: default
spec:
  selector:
    # use Istio default gateway implementation
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: httpbin-tls-credential
    hosts:
    - "httpbin.boutiquestore.com"
```

## Create virtual service configuration
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin-virtual-service
  namespace: default
spec:
  hosts:
  - "httpbin.boutiquestore.com"
  gateways:
  - httpbin-gateway
  http:
  - route:
    - destination:
        host: httpbin
        port:
          number: 8000
```

## Validate that HTTPS traffic can be sent to httpbin application
```
❯ curl --cacert httpbin-tls-credential-root.crt \
  -H "Host: httpbin.boutiquestore.com" \
  --resolve "httpbin.boutiquestore.com:443:$INGRESS_IP"\
  "https://httpbin.boutiquestore.com:443/get"
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.boutiquestore.com", 
    "User-Agent": "curl/7.79.1", 
    "X-B3-Parentspanid": "8ab794af57ac18f0", 
    "X-B3-Sampled": "1", 
    "X-B3-Spanid": "da5d9bde1b7409e2", 
    "X-B3-Traceid": "5c8e2adff1cda0ce8ab794af57ac18f0", 
    "X-Envoy-Attempt-Count": "1", 
    "X-Envoy-Internal": "true", 
    "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/default/sa/httpbin;Hash=b9907740ff57c1e03e784bcebd60d2408cacc1a6795d14980ceba48c54a9bd5d;Subject=\"\";URI=spiffe://cluster.local/ns/istio-system/sa/istio-ingressgateway-service-account"
  }, 
  "origin": "172.17.0.1", 
  "url": "https://httpbin.boutiquestore.com/get"
}
```