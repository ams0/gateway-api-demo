# gateway-api-demo
Gateway API demo with Istio



## As a Infrastructure Engineer
istioctl install --set profile=minimal -y

kubectl create namespace external-dns
kubectl create secret generic azure-config-file -n external-dns --from-file=azure.json
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm upgrade --install external-dns external-dns/external-dns \
  --namespace external-dns \
  --set domainFilters={gloo.love} \
  --set provider=azure \
  --values external-dns-values.yaml

kubectl apply -f demo.yaml
kubectl apply -f gw.yaml
kubectl get gateways -A
NAMESPACE      NAME     CLASS   ADDRESS        PROGRAMMED   AGE
istio-system   gw-api   istio   20.76.208.33   True         11s

az network dns record-set a add-record \
    --resource-group dns \
    --zone-name gloo.love \
    --record-set-name "*" \
    --ipv4-address 20.76.208.33