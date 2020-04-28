## Kong
Create a namespace for kong:

```shell script
kubectl create namespace kong

kubectl apply -n kong -f kong/kong-oidc-plugin.yaml
```

To install kong, use `helm`: 

```shell script
# install kong
helm install \
    --namespace kong \
    --name kong \
    --values kong/kong.helm.yaml \
    kong/kong

# export the port where it is running
export KONG_PORT=$(minikube service -n default kong-kong-proxy --url \
    | head -n 1 | awk -F: '{print $NF}')
```

Install and configure konga to connect to the Kong admin API:

```shell script
kubectl apply -n kong -f kong/konga.yaml
```

And then use the `http://kong-kong-admin:8001` URL to connect to the
Kong gateway.

Then install the prepared ingress:

```shell script
kubectl apply -f kong/kong-ingress.yaml
```

And then test all routes:

```shell script
curl  $KUBE_IP:$KONG_PORT/foo
curl  $KUBE_IP:$KONG_PORT/bar #TODO this does not work for some reason...
curl foo.kong.local:$KONG_PORT
curl bar.kong.local:$KONG_PORT
```