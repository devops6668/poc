## helm value

Create tls secret "nginx-default-tls" on NS kube-system
```
controller:
.
.
.
  extraArgs:
    default-ssl-certificate: $(POD_NAMESPACE)/nginx-default-tls
```

## helm cli
```
helm install ingress-nginx ingress-nginx/ingress-nginx \
    --create-namespace --namespace ingress-nginx \
    --set controller.extraArgs.default-ssl-certificate="<namespace>/<secret_name>"
```
