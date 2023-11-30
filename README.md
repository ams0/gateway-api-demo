# gateway-api-demo

Gateway API demo with Istio


## As a Infrastructure Engineer

Install GatewayAPI CRDs
```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/experimental-install.yaml
```

Deploy Istio without ingress-gateway

```bash
istioctl install --set profile=minimal -y
```

## As a Cluster Operator

(Optional) Deploy external-dns (have an `azure.json` file ready with the credentials of a Service Principal that can operate on the DNS zone ): 
```bash
kubectl create namespace external-dns
kubectl create secret generic azure-config-file -n external-dns --from-file=azure.json
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm upgrade --install external-dns external-dns/external-dns \
  --namespace external-dns \
  --set domainFilters={gloo.love} \
  --set provider=azure \
  --values external-dns-values.yaml
```

Deploy a Gateway

```bash
kubectl apply -f gw.yaml
```

Verify the Gateway is created

```bash
kubectl get gateways -A
NAMESPACE      NAME     CLASS   ADDRESS        PROGRAMMED   AGE
istio-system   gw-api   istio   20.76.208.33   True         11s
```

## As an Application Developer

Deploy the demo app

```bash
kubectl apply -f demo.yaml
```

Deploy the route

```bash
kubectl apply -f demo-route.yaml
```

Test (replace the IP with the one from the gateway, unless external-dns has already updated the DNS record):

```bash
curl -I -H person:alice -H "Host: demo.gloo.love"  20.76.208.33/echo
curl -I -H person:bob -H "Host: demo.gloo.love"  20.76.208.33/echo
```
