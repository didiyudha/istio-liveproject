# Verifying User Identity at the Ingress Gateway Using JSON Web Tokens

## Objectives
```
Configure your mesh to verify users’ identities using the JSON Web Token (JWT) authentication mechanism so that you can validate the identities of your users and enforce access control to sensitive information. You will update the Istio ingress gateway to perform JWT validation of the user requests by configuring Istio’s RequestAuthentication resource.
```

## Configure RequestAuthentication on Istio ingress gateway
```
apiVersion: "security.istio.io/v1beta1"
kind: "RequestAuthentication"
metadata:
  name: online-boutique
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  jwtRules:
  - issuer: "testing@secure.istio.io"
    jwksUri: "https://raw.githubusercontent.com/istio/istio/release-1.9/security/tools/jwt/samples/jwks.json"
```

## Verify that the configured RequestAuthentication resource has taken effect by the following steps
### Set the environment variable VALID_TOKEN to the token genrated bellow for demo purposes
```
export VALID_TOKEN="eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTM2NSwiaWF0IjoxNTk5MDA1MzY1LCJpc3MiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.pZHGJgEP0qsJ1pSBPU6nwpMETf3MqlWx-T0PED0BLb5aTnbXWklIzHV4JHgMk6Td4thI4cC4_mCvrFlgm1uqzO2U_DEdNOReNMGNnDSQ87hWEtOQxn50c83dVq3-w4FUrKBjS1lYhiabKWVfr3BC_Bmwefl1TVhgP_iNuQGsWsj_Tcl4TWP93yS93DazphAl0uIOw3WvmJSfhnaZ7cEIk0PzGmsLq96FLbGatZX1DL8OKhv2Hz-iN0V6KG_5GjjEdD7LVSprw4oedtwbahDVVuxh0due1DImXLhH9oH1_RkntrD98JlKYoNzG1-7apSV0xw5mlE8L2HXDo3TKE33WA"
```

### Inspect the contents of the JWT by extracting its payload
```
❯ echo $VALID_TOKEN | cut -d '.' -f2 | base64 --decode | awk '{print $0"}"}' | jq .
{
  "aud": "boutiquestore.com",
  "exp": 1914365365,
  "iat": 1599005365,
  "iss": "testing@secure.istio.io",
  "sub": "user1@xyz.com"
}
```

### Use the following cURL command to issue an HTTP request with the Authorization header set to “Bearer” with the token above

```
❯  curl -v -H "Authorization: Bearer $VALID_TOKEN" http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.79.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTM2NSwiaWF0IjoxNTk5MDA1MzY1LCJpc3MiOiJ0ZXN0aW5nQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.pZHGJgEP0qsJ1pSBPU6nwpMETf3MqlWx-T0PED0BLb5aTnbXWklIzHV4JHgMk6Td4thI4cC4_mCvrFlgm1uqzO2U_DEdNOReNMGNnDSQ87hWEtOQxn50c83dVq3-w4FUrKBjS1lYhiabKWVfr3BC_Bmwefl1TVhgP_iNuQGsWsj_Tcl4TWP93yS93DazphAl0uIOw3WvmJSfhnaZ7cEIk0PzGmsLq96FLbGatZX1DL8OKhv2Hz-iN0V6KG_5GjjEdD7LVSprw4oedtwbahDVVuxh0due1DImXLhH9oH1_RkntrD98JlKYoNzG1-7apSV0xw5mlE8L2HXDo3TKE33WA
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Sun, 29 Jan 2023 12:58:11 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #0 to host 127.0.0.1 left intact
❯  curl -v -H "Authorization: Bearer $VALID_TOKEN" -H "Host: marketplace.boutiquestore.com" http://$INGRESS:80/_healthz
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
< set-cookie: shop_session-id=073f54fa-ce8f-4fd0-b397-db019eb8fb05; Max-Age=172800
< date: Sun, 29 Jan 2023 12:59:36 GMT
< content-length: 2
< content-type: text/plain; charset=utf-8
< x-envoy-upstream-service-time: 3
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
ok
```

### Validate that the ingress gateway is verifying the JWT at all ports (80 and 443) by using the following command, which sends HTTPS traffic to port 443.
```
❯ curl -H "Authorization: Bearer $VALID_TOKEN" \
  -H "Host: marketplace.boutiquestore.com" \
  --cacert root.crt \
  --resolve "marketplace.boutiquestore.com:443:$INGRESS_IP" \
  "https://marketplace.boutiquestore.com:443/_healthz"
ok
```

### Let’s verify whether an invalid JWT is rejected by the ingress gateway. Set the environment variable $INVALID_TOKEN to an invalid token generated for demo usage.
```
export INVALID_TOKEN="eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTI1MSwiaWF0IjoxNTk5MDA1MjUxLCJpc3MiOiJ3cm9uZy1pc3N1c2VyQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.SnQPGlkV66Q61zR8uZAzEhyPfynNmV_MGzvjnkxZhVv-elrKu7Wq50tj4SKGyjTJPR2-YFd_-p3eN4VveCH5NB3LjgyOliMQjnxSTN92CHXjoy6kHol2Lo-kFJmoNBvNBkKFpFJ3oD6ejTse7718r7WSUzeh4R_vV9QNEHNPucxpL3Yhm_EuYIMV-cfA_N58dA1YAjcZtlEM8PsFwDQGv5vTndGkQ_co0acuDBgXZsJ6xCaNvpgrx_ftpzlaA27PknKK6rrvTRSuxKP4Jn3GIB0nBa6uXUfMvlkUBvepwXooXO5XAlWRTa3J6ys2KkOVkDKMN-jdSv-K3_rLXxxb3Q"
```

### You can inspect the contents of the payload in the invalid JWT.
```
❯ echo $INVALID_TOKEN | cut -d '.' -f2 | base64 --decode | awk '{print $0"}"}' | jq .
{
  "aud": "boutiquestore.com",
  "exp": 1914365251,
  "iat": 1599005251,
  "iss": "wrong-issuser@secure.istio.io",
  "sub": "user1@xyz.com"
}
```

### Verify that issuing the cURL command as before using the invalid token returns an HTTP 401 unauthorized status code.
```
❯ curl -v -H "Authorization: Bearer $INVALID_TOKEN" http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.79.1
> Accept: */*
> Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkRIRmJwb0lVcXJZOHQyenBBMnFYZkNtcjVWTzVaRXI0UnpIVV8tZW52dlEiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJib3V0aXF1ZXN0b3JlLmNvbSIsImV4cCI6MTkxNDM2NTI1MSwiaWF0IjoxNTk5MDA1MjUxLCJpc3MiOiJ3cm9uZy1pc3N1c2VyQHNlY3VyZS5pc3Rpby5pbyIsInN1YiI6InVzZXIxQHh5ei5jb20ifQ.SnQPGlkV66Q61zR8uZAzEhyPfynNmV_MGzvjnkxZhVv-elrKu7Wq50tj4SKGyjTJPR2-YFd_-p3eN4VveCH5NB3LjgyOliMQjnxSTN92CHXjoy6kHol2Lo-kFJmoNBvNBkKFpFJ3oD6ejTse7718r7WSUzeh4R_vV9QNEHNPucxpL3Yhm_EuYIMV-cfA_N58dA1YAjcZtlEM8PsFwDQGv5vTndGkQ_co0acuDBgXZsJ6xCaNvpgrx_ftpzlaA27PknKK6rrvTRSuxKP4Jn3GIB0nBa6uXUfMvlkUBvepwXooXO5XAlWRTa3J6ys2KkOVkDKMN-jdSv-K3_rLXxxb3Q
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 401 Unauthorized
< www-authenticate: Bearer realm="http://127.0.0.1/_healthz", error="invalid_token"
< content-length: 28
< content-type: text/plain
< date: Sun, 29 Jan 2023 13:13:13 GMT
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
Jwt issuer is not configured
```

### Since the RequestAuthentication resource only configures the ingress gateway to reject requests if an Authorization header is set, you can verify that the request is allowed if no JWT is passed in the request.
```
❯ curl -v http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Sun, 29 Jan 2023 13:14:28 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #0 to host 127.0.0.1 left intact
❯ curl -H "Host: marketplace.boutiquestore.com" -v http://$INGRESS:80/_healthz
*   Trying 127.0.0.1:80...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET /_healthz HTTP/1.1
> Host: marketplace.boutiquestore.com
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=13f0f92b-53a8-47cc-9890-53ac61320481; Max-Age=172800
< date: Sun, 29 Jan 2023 13:14:58 GMT
< content-length: 2
< content-type: text/plain; charset=utf-8
< x-envoy-upstream-service-time: 1
< server: istio-envoy
< 
* Connection #0 to host 127.0.0.1 left intact
ok
```