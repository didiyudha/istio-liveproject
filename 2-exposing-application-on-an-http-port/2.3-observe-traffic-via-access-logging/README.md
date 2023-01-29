# Observe traffic via access logging

## Turn on Envoy access logging globally mesh wide
```
istioctl install --set profile=demo --set meshConfig.accessLogFile="/dev/stdout" --set meshConfig.accessLogEncoding=JSON -y
```

## See the logs of istio ingress gateway deployment
```
k -n istio-system logs deploy/istio-ingressgateway -f --tail 1 | jq
```

```
{
  "connection_termination_details": null,
  "route_name": null,
  "bytes_sent": 10766,
  "upstream_service_time": "26",
  "protocol": "HTTP/1.1",
  "duration": 27,
  "x_forwarded_for": "172.17.0.1",
  "upstream_transport_failure_reason": null,
  "start_time": "2023-01-29T07:01:33.906Z",
  "upstream_cluster": "outbound|80||frontend-v2.online-boutique.svc.cluster.local",
  "bytes_received": 0,
  "response_code_details": "via_upstream",
  "path": "/",
  "upstream_local_address": "172.17.0.2:59704",
  "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36",
  "request_id": "cfa56c3a-0fe2-936e-beaf-4b8296a9a716",
  "authority": "online-boutique.com",
  "downstream_local_address": "172.17.0.2:8080",
  "requested_server_name": null,
  "response_flags": "-",
  "downstream_remote_address": "172.17.0.1:13507",
  "upstream_host": "172.17.0.18:8080",
  "method": "GET",
  "response_code": 200
}
```

## Deliverable 
The deliverable for this milestone is the output of the command curl http://$INGRESS/wrongpath, the corresponding access log captured from the ingress gateway pod in JSON format, and an explanation of whether or not the request is proxied by the ingress gateway to the frontend service based on the access log message.

### Generate the logs
```
while true; do curl -H "Host: online-boutique.com" http://127.0.0.1/wrongpath ; sleep 1; done
```

### Istio gateway log
```
 kubectl -n istio-system logs deploy/istio-ingressgateway --tail 1 -f | jq
```
```
{
  "response_flags": "-",
  "upstream_host": "172.17.0.18:8080",
  "authority": "online-boutique.com",
  "request_id": "d858d620-6866-9901-8290-13d773c97093",
  "start_time": "2023-01-29T08:38:33.934Z",
  "upstream_cluster": "outbound|80||frontend-v2.online-boutique.svc.cluster.local",
  "downstream_local_address": "172.17.0.2:8080",
  "path": "/wrongpath",
  "upstream_service_time": "1",
  "method": "GET",
  "user_agent": "curl/7.79.1",
  "connection_termination_details": null,
  "x_forwarded_for": "172.17.0.1",
  "downstream_remote_address": "172.17.0.1:54792",
  "bytes_received": 0,
  "bytes_sent": 19,
  "upstream_transport_failure_reason": null,
  "response_code_details": "via_upstream",
  "requested_server_name": null,
  "upstream_local_address": "172.17.0.2:49342",
  "route_name": null,
  "protocol": "HTTP/1.1",
  "response_code": 404,
  "duration": 1
}
```
### Frontend logs

```
kubectl -n online-boutique logs deploy/frontend --tail 1 -f | jq
```

```
{
  "http.req.id": "89578c27-d659-4e2e-a6d0-00b2b0fb64c3",
  "http.req.method": "GET",
  "http.req.path": "/wrongpath",
  "message": "request started",
  "session": "242e2b7e-2757-4f2c-9ec3-cd9a5f0070de",
  "severity": "debug",
  "timestamp": "2023-01-29T08:37:54.964694567Z"
}
```

*So we can say that the request is proxied by the ingress gateway to the frontend service based on the access log message*