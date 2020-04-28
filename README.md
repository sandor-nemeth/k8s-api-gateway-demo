# API Gateway demos

This project tries to achieve a couple of goals: 

1. Deploy [Kong], [Tyk] and [Ambassador] as kubernetes ingresses. 
2. Provide the following features to test:
    1. Ingress-based, domain and path driven access to services
    2. A graphical UI (if exist)
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
initialized. To test the gateways, we will utilize:

- a simple `echoservice`, which provides us with a simple response
  containing the request properties, and
- a `helloservice`, which provides a second service for us to route to

```shell script
kubectl apply -f common/echoserver.yaml
kubectl apply -f common/helloserver.yaml
```

Replace the IP address with `minikube ip`, and then add these entries
to the `/etc/hosts` file:
```
192.168.99.108  api.nginx.local
192.168.99.108  api.tyk.local
192.168.99.108  api.kong.local
192.168.99.108  api.ambassador.local
192.168.99.108  secure.nginx.local
192.168.99.108  secure.tyk.local
192.168.99.108  secure.kong.local
192.168.99.108  secure.ambassador.local
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

[Kong]: https://konghq.com/kong/
[Tyk]: https://tyk.io/
[Ambassador]: https://www.getambassador.io/
[Keycloak]: https://www.keycloak.org/
[homebrew]: https://brew.sh/
