# Protecting Sensitive Information from Unauthorized Access

## Objective

```
Configure an Istio ingress gateway to only allow users with valid JWTs to access sensitive information in your online boutique store, such as viewing items in the cart. To protect user data, you will add a layer of authorization in your mesh to prevent unauthenticated access.
```

## Create authorization policy

```
apiVersion: "security.istio.io/v1beta1"
kind: "AuthorizationPolicy"
metadata:
  name: online-boutique
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  action: DENY
  rules:
  - from:
    - source:
        notRequestPrincipals: ["*"]
```

## Use the cURL command to issue an HTTP request with the Authorization header set to “Bearer” with the valid token defined in the previous section.
```
❯ curl -v -H "Authorization: Bearer $VALID_TOKEN" -H "Host: marketplace.boutiquestore.com" http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTM2NSwiaWF0IjoxNTk5MDA1MzY1LCJpc3MiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.pZHGJgEP0qsJ1pSBPU6nwpMETf3MqlWx-T0PED0BLb5aTnbXWklIzHV4JHgMk6Td4thI4cC4_mCvrFlgm1uqzO2U_DEdNOReNMGNnDSQ87hWEtOQxn50c83dVq3-w4FUrKBjS1lYhiabKWVfr3BC_Bmwefl1TVhgP_iNuQGsWsj_Tcl4TWP93yS93DazphAl0uIOw3WvmJSfhnaZ7cEIk0PzGmsLq96FLbGatZX1DL8OKhv2Hz-iN0V6KG_5GjjEdD7LVSprw4oedtwbahDVVuxh0due1DImXLhH9oH1_RkntrD98JlKYoNzG1-7apSV0xw5mlE8L2HXDo3TKE33WA
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=aefbf7e7-340f-420a-816f-87f64f7bf427; Max-Age=172800
< date: Mon, 30 Jan 2023 11:58:06 GMT
< content-length: 2
< content-type: text/plain; charset=utf-8
< x-envoy-upstream-service-time: 3
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
ok   
```

## Next, let’s issue the same command as above without the JWT

```
❯ curl -H "Host: marketplace.boutiquestore.com" -v http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< content-length: 19
< content-type: text/plain
< date: Mon, 30 Jan 2023 11:59:57 GMT
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
RBAC: access denied
```

##  Validate that a request with an invalid token continues to get a 401 unauthorized response code as before

```
❯ curl -H "Host: marketplace.boutiquestore.com" -H "Authorization: Bearer $INVALID_TOKEN" -v http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTI1MSwiaWF0IjoxNTk5MDA1MjUxLCJpc3MiOiJ3cm9uZy1pc3N1c2VyQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.SnQPGlkV66Q61zR8uZAzEhyPfynNmV_MGzvjnkxZhVv-elrKu7Wq50tj4SKGyjTJPR2-YFd_-p3eN4VveCH5NB3LjgyOliMQjnxSTN92CHXjoy6kHol2Lo-kFJmoNBvNBkKFpFJ3oD6ejTse7718r7WSUzeh4R_vV9QNEHNPucxpL3Yhm_EuYIMV-cfA_N58dA1YAjcZtlEM8PsFwDQGv5vTndGkQ_co0acuDBgXZsJ6xCaNvpgrx_ftpzlaA27PknKK6rrvTRSuxKP4Jn3GIB0nBa6uXUfMvlkUBvepwXooXO5XAlWRTa3J6ys2KkOVkDKMN-jdSv-K3_rLXxxb3Q
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 401 Unauthorized
< www-authenticate: Bearer realm="http://marketplace.boutiquestore.com/_healthz", error="invalid_token"
< content-length: 28
< content-type: text/plain
< date: Mon, 30 Jan 2023 12:01:29 GMT
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
Jwt issuer is not configured
```

## Vrify that the policy is taking effect by visiting “http://$INGRESS” in your browser

![Boutique Website](./images/access-denied.png)

## Configure an audience-scoped authorization policy
```
apiVersion: "security.istio.io/v1beta1"
kind: "AuthorizationPolicy"
metadata:
  name: online-boutique
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  action: ALLOW
  rules:
  - when:
    - key: "request.auth.audiences"
      values: ["boutiquestore.com"]
```

## Set the environment variable WRONG_AUDIENCE_TOKEN to the token generated below with an incorrect audience claim
```
export WRONG_AUDIENCE_TOKEN="eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJ3cm9uZy1hdWRpZW5jZS5jb20iLCJleHAiOjE5MTUyNTc5NzksImlhdCI6MTU5OTY4ODQ1OSwiaXNzIjoidGVzdGluZ0BzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyJ9.ELpcxqd9T6ZmdQ6Ilak26yiizi205az0hSmNnVALV_pCfbM2frMaNnESotFILe77Z3u-HeiVTomrgf-onAUW0bZJ3HWi_iPJZHQ5bq2-rvf5_E4IAr_p39argaVriCVRiPnBtuwzzptWOpS-wBe5uDr3V6otX7PcAH9s1FCCMnTxiEJzn_taTYLtpctoHDsNfw5RfDvVZbsuyEX1Ea0UUx1-VZrVV7n5_jXmo8yoHQ5DH6QDgpGJlYMebgLgoG7pL8N2oz7VLQQSO0x-rIdv_Icdee04fnrOzs01YdGWqfEfDjGbAjrqsq0e1HXhHF0AJNcsuNcFcQQv4nePyuKVyg"
```

## Inspect the contents of the JWT by extracting its payload.
```
❯ echo $WRONG_AUDIENCE_TOKEN | cut -d '.' -f2 | base64 --decode | awk '{print $0}' | jq .
{
  "aud": "wrong-audience.com",
  "exp": 1915257979,
  "iat": 1599688459,
  "iss": "testing@secure.istio.io",
  "sub": "testing@secure.istio.io"
}
```

## Issue the cURL command with the authorization header set to this token.
```
❯ curl -H "Host: marketplace.boutiquestore.com" -H "Authorization: Bearer $WRONG_AUDIENCE_TOKEN" -v http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJ3cm9uZy1hdWRpZW5jZS5jb20iLCJleHAiOjE5MTUyNTc5NzksImlhdCI6MTU5OTY4ODQ1OSwiaXNzIjoidGVzdGluZ0BzZWN1cmUuaXN0aW8uaW8iLCJzdWIiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyJ9.ELpcxqd9T6ZmdQ6Ilak26yiizi205az0hSmNnVALV_pCfbM2frMaNnESotFILe77Z3u-HeiVTomrgf-onAUW0bZJ3HWi_iPJZHQ5bq2-rvf5_E4IAr_p39argaVriCVRiPnBtuwzzptWOpS-wBe5uDr3V6otX7PcAH9s1FCCMnTxiEJzn_taTYLtpctoHDsNfw5RfDvVZbsuyEX1Ea0UUx1-VZrVV7n5_jXmo8yoHQ5DH6QDgpGJlYMebgLgoG7pL8N2oz7VLQQSO0x-rIdv_Icdee04fnrOzs01YdGWqfEfDjGbAjrqsq0e1HXhHF0AJNcsuNcFcQQv4nePyuKVyg
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< content-length: 19
< content-type: text/plain
< date: Mon, 30 Jan 2023 12:14:02 GMT
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
RBAC: access denied%
```

## Configure an audience- and path-scoped authorization policy
```
apiVersion: "security.istio.io/v1beta1"
kind: "AuthorizationPolicy"
metadata:
  name: online-boutique
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  action: DENY
  rules:
  - when:
    - key: "request.auth.audiences"
      notValues: ["boutiquestore.com"]
    to:
    - operation:
        paths: ["/cart"]
```

## Verifying your audience- and path-scoped authorization policy

```
❯ curl -H "Host: marketplace.boutiquestore.com"  -v http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=0313f6d8-1859-41c5-84e5-f95bf89609d7; Max-Age=172800
< date: Mon, 30 Jan 2023 12:23:14 GMT
< content-length: 2
< content-type: text/plain; charset=utf-8
< x-envoy-upstream-service-time: 4
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
ok
```

```
❯ curl -H "Host: marketplace.boutiquestore.com"  -v http://$INGRESS:80/cart
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /cart HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 403 Forbidden
< content-length: 19
< content-type: text/plain
< date: Mon, 30 Jan 2023 12:23:39 GMT
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
RBAC: access denied
```

## Adding a valid JWT to the command above should ensure that the request is allowed.
```
❯ curl -sS -H "Host: marketplace.boutiquestore.com" -H "Authorization: Bearer $VALID_TOKEN" --write-out "%{http_code}" -o /dev/null  -v http://$INGRESS:80/cart
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /cart HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTM2NSwiaWF0IjoxNTk5MDA1MzY1LCJpc3MiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.pZHGJgEP0qsJ1pSBPU6nwpMETf3MqlWx-T0PED0BLb5aTnbXWklIzHV4JHgMk6Td4thI4cC4_mCvrFlgm1uqzO2U_DEdNOReNMGNnDSQ87hWEtOQxn50c83dVq3-w4FUrKBjS1lYhiabKWVfr3BC_Bmwefl1TVhgP_iNuQGsWsj_Tcl4TWP93yS93DazphAl0uIOw3WvmJSfhnaZ7cEIk0PzGmsLq96FLbGatZX1DL8OKhv2Hz-iN0V6KG_5GjjEdD7LVSprw4oedtwbahDVVuxh0due1DImXLhH9oH1_RkntrD98JlKYoNzG1-7apSV0xw5mlE8L2HXDo3TKE33WA
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=7f8ad6c0-caa7-4f88-8de3-6c593b23cd21; Max-Age=172800
< date: Mon, 30 Jan 2023 12:25:16 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 35
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [4865 bytes data]
* Connection #0 to host 127.0.0.1 left intact
200% 
```

## Deliverable
```
The deliverable for this milestone is the Authorization policy resource online-boutique, updated to scope it to only affect requests to HTTPS port 443 and not port 80. Verify that traffic to path /cart on port 80 is allowed without a JWT but rejected on port 443.
```

*Authorzation Policy*

```
apiVersion: "security.istio.io/v1beta1"
kind: "AuthorizationPolicy"
metadata:
  name: online-boutique
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  action: DENY
  rules:
  - when:
    - key: "request.auth.audiences"
      notValues: ["boutiquestore.com"]
    to:
    - operation:
        paths: ["/cart"]
        ports:
          - "443"
```

*HTTP*
```
❯ curl -sS -H "Host: marketplace.boutiquestore.com"  --write-out "%{http_code}" -o /dev/null  -v http://$INGRESS:80/cart
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /cart HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=f7161a1e-5ee6-437e-932e-0af1b9463f04; Max-Age=172800
< date: Mon, 30 Jan 2023 12:36:40 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 13
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [3885 bytes data]
* Connection #0 to host 127.0.0.1 left intact
200
```

*HTTPS*
```
❯ curl -H "Authorization: Bearer $VALID_TOKEN" \
  -H "Host: marketplace.boutiquestore.com" \
  --cacert root.crt \
  --resolve "marketplace.boutiquestore.com:443:$INGRESS_IP" \
  --write-out "%{http_code}" -o /dev/null "https://marketplace.boutiquestore.com:443/cart"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4852    0  4852    0     0   267k      0 --:--:-- --:--:-- --:--:--  278k
200
```