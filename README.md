# istio-liveproject

## Minikube installation

```
minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.23.12
```

## Create tunnel from minikube to localhost
```
sudo minikube tunnel
```

Let the terminal open after running the command.

## Check Kubernetes installation

```
kubectl get nodes
```

## Install istio

```
istioctl install --set profile=demo
```

## Verify Istio installation

```
istioctl verify-install
```

```
1 Istio control planes detected, checking --revision "default" only
✔ ClusterRole: istiod-istio-system.istio-system checked successfully
✔ ClusterRole: istio-reader-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istio-reader-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istiod-istio-system.istio-system checked successfully
✔ ServiceAccount: istio-reader-service-account.istio-system checked successfully
✔ Role: istiod-istio-system.istio-system checked successfully
✔ RoleBinding: istiod-istio-system.istio-system checked successfully
✔ ServiceAccount: istiod-service-account.istio-system checked successfully
✔ CustomResourceDefinition: wasmplugins.extensions.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: destinationrules.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: envoyfilters.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: gateways.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: proxyconfigs.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: serviceentries.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: sidecars.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: virtualservices.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: workloadentries.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: workloadgroups.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: authorizationpolicies.security.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: peerauthentications.security.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: requestauthentications.security.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: telemetries.telemetry.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: istiooperators.install.istio.io.istio-system checked successfully
✔ ClusterRole: istiod-clusterrole-istio-system.istio-system checked successfully
✔ ClusterRole: istiod-gateway-controller-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istiod-clusterrole-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istiod-gateway-controller-istio-system.istio-system checked successfully
✔ ConfigMap: istio.istio-system checked successfully
✔ Deployment: istiod.istio-system checked successfully
✔ ConfigMap: istio-sidecar-injector.istio-system checked successfully
✔ MutatingWebhookConfiguration: istio-sidecar-injector.istio-system checked successfully
✔ PodDisruptionBudget: istiod.istio-system checked successfully
✔ ClusterRole: istio-reader-clusterrole-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istio-reader-clusterrole-istio-system.istio-system checked successfully
✔ Role: istiod.istio-system checked successfully
✔ RoleBinding: istiod.istio-system checked successfully
✔ Service: istiod.istio-system checked successfully
✔ ServiceAccount: istiod.istio-system checked successfully
✔ EnvoyFilter: stats-filter-1.11.istio-system checked successfully
✔ EnvoyFilter: tcp-stats-filter-1.11.istio-system checked successfully
✔ EnvoyFilter: stats-filter-1.12.istio-system checked successfully
✔ EnvoyFilter: tcp-stats-filter-1.12.istio-system checked successfully
✔ EnvoyFilter: stats-filter-1.13.istio-system checked successfully
✔ EnvoyFilter: tcp-stats-filter-1.13.istio-system checked successfully
✔ ValidatingWebhookConfiguration: istio-validator-istio-system.istio-system checked successfully
✔ Deployment: istio-ingressgateway.istio-system checked successfully
✔ PodDisruptionBudget: istio-ingressgateway.istio-system checked successfully
✔ Role: istio-ingressgateway-sds.istio-system checked successfully
✔ RoleBinding: istio-ingressgateway-sds.istio-system checked successfully
✔ Service: istio-ingressgateway.istio-system checked successfully
✔ ServiceAccount: istio-ingressgateway-service-account.istio-system checked successfully
✔ Deployment: istio-egressgateway.istio-system checked successfully
✔ PodDisruptionBudget: istio-egressgateway.istio-system checked successfully
✔ Role: istio-egressgateway-sds.istio-system checked successfully
✔ RoleBinding: istio-egressgateway-sds.istio-system checked successfully
✔ Service: istio-egressgateway.istio-system checked successfully
✔ ServiceAccount: istio-egressgateway-service-account.istio-system checked successfully
Checked 15 custom resource definitions
Checked 3 Istio Deployments
✔ Istio is installed and verified successfully
```

## Create online-boutique namespace

```
kubectl create namespace online-boutique
```

## Label the namespace
```
kubectl label ns istio-injection=enabled
```

```
kubectl get ns -L istio-injection
```

```
NAME              STATUS   AGE     ISTIO-INJECTION
default           Active   7h55m   
istio-system      Active   7h51m   
kube-node-lease   Active   7h55m   
kube-public       Active   7h55m   
kube-system       Active   7h55m   
online-boutique   Active   32m     enabled
```

## Install online boutique microservices

```
curl -L https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml -o online-boutique-manifests.yaml
```

```
kubectl -n online-boutique apply -f online-boutique-manifests.yaml
```

## Verify all pods are running

```
kubectl get pods -n online-boutique
```

```
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-8587b48c5f-kwmrm               2/2     Running   0          33m
cartservice-5c65c67f5d-b6k46             2/2     Running   0          33m
checkoutservice-54c9f7f49f-7phgt         2/2     Running   0          33m
currencyservice-5877b8dbcc-jbqvf         2/2     Running   0          33m
emailservice-5c5448b7bc-4ckmg            2/2     Running   0          33m
frontend-67f6fdc769-vv4sq                2/2     Running   0          33m
loadgenerator-555c7c5c44-d68q5           2/2     Running   0          33m
paymentservice-7bc7f76c67-j5c7x          2/2     Running   0          33m
productcatalogservice-67fff7c687-x9kgm   2/2     Running   0          33m
recommendationservice-b49f757f-hsqjk     2/2     Running   0          33m
redis-cart-58648d854-85gtp               2/2     Running   0          33m
shippingservice-76b9bc7465-464zf         2/2     Running   0          33m
```