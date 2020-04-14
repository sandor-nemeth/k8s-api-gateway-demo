# API Gateway demos

This document tries to achieve a couple of goals: 

1. Deploy [Kong], [Tyk] and [Ambassador] as kubernetes ingresses. 
2. Provide the following features to test:
  1. Ingress-based, domain and path driven access to services
  2. graphical UI 
  3. Authentication and access control configured in the gateway
3. Use [Keycloak] as an authentication source

## Setup

All deployments must happen on `Kubernetes 1.14.10` as a version, to
be able to reproduce it on the UP42 production environment. Where
applicable and provided, the deployments can utilize `helm` charts.

All prerequisites can be set up using the following (assuming that
[homebrew] is installed and available):

```shell script
brew install minikube helm@2
minikube start --kubernetes-version v1.14.10 --memory 4096 --cpus 2

# init helm and add repositories
helm init
helm repo add codecentric https://codecentric.github.io/helm-charts
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add kong https://charts.konghq.com
helm repo update
```

This gives us a fully set up kubernetes environment with helm also
initialized. To test the gateways, we will utilize a simple echo
service, which provides us with a simple response containing the
request properties:

```shell script
kubectl apply -f common/echoserver.yaml
```

To support the authentication test cases, a [Keycloak] instance will be
set up: 

```shell script
helm install --name keycloak codecentric/keycloak

# get the generated password
kubectl get secret -n default keycloak-http -o jsonpath="{.data.password}" \
    | base64 --decode; echo
```

### Required environment variables

After finished with the `Setup` section, executing the script below
will provide the variables the further sections of the README
reference.

```shell script
export KUBE_IP=$(minikube ip)
```

Set up all resolvable names in the `/etc/hosts` file:

```shell script
echo "$KUBE_IP  foo.kong.local\n$KUBE_IP  bar.kong.local" \
    | sudo tee -a /etc/hosts
echo "$KUBE_IP  foo.tyk.local\n$KUBE_IP  bar.tyk.local" \
    | sudo tee -a /etc/hosts
echo "$KUBE_IP  foo.ambassador.local\n$KUBE_IP  bar.ambassador.local" \
    | sudo tee -a /etc/hosts
```

## Kong

To install kong, use `helm`: 

```shell script
# install kong
helm install --name kong kong/kong \
    --set admin.enabled=true \
    --set admin.http.enabled=true \
    --set admin.tls.enabled=false

# export the port where it is running
export KONG_PORT=$(minikube service -n default kong-kong-proxy --url \
    | head -n 1 | awk -F: '{print $NF}')
```

Install and configure konga to connect to the Kong admin API:

```shell script
kubectl apply -f kong/konga.yaml
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


[Kong]: https://konghq.com/kong/
[Tyk]: https://tyk.io/
[Ambassador]: https://www.getambassador.io/
[Keycloak]: https://www.keycloak.org/
[homebrew]: https://brew.sh/
