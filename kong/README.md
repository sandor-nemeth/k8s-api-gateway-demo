## Kong
Create a namespace for kong:

```shell script
kubectl create namespace kong

kubectl apply -n kong -f kong/kong-oidc-plugin.yaml
```

To install kong, use `helm`: 

```shell script
# install kong
kubectl apply -f kong/kong.yaml

# export the port where it is running
export KONG_PORT=$(minikube service -n kong kong-proxy --url \
    | head -n 1 | awk -F: '{print $NF}')
```

Configure konga to connect to the Kong admin API on the `http://kong-admin` endpoint.

Then install the prepared ingress:

```shell script
kubectl apply -f kong/ingress.yaml
```

Test accessing the secured service:

```shell script
curl -XPOST \
    -F 'grant_type=password' \
    -F 'client_id=account' \
    -F 'client_secret=0ec2950c-a80d-49f5-be83-0a76bcf195ed' \
    -F 'username=sandor' \
    -F 'password=sandor' \
    
```