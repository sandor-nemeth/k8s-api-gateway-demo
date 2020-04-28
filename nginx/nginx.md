Enable NGINX ingress

```shell script
minikube addons enable ingress
```

Apply the config:

```
kubectl apply -f nginx/ingress.yaml
```

And test:
```
curl api.nginx.local/foo
```