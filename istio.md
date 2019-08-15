# Excercise - istio 

## Deploy the TLS ingress
#### Steps
- Explore the content of definitions/istio_ingress_001.yaml and find where the key 
and certificate are defined
- The deployment nginx-healthcheck-only is needed to back the Ingress health checks,
but it will not receive traffic headed to mytraininghostname.com
- Apply routing excercise 001
```
kubectl apply -f definitions/istio_ingress_001.yaml
```
- Add the IP of the ingress to a variable to verify the ingress is working
```
export IP_OF_THE_INGRESS=127.0.0.1
curl -kv https://$IP_OF_THE_INGRESS/ -H HOST:mytraininghostname.com
curl -kv https://$IP_OF_THE_INGRESS/ -H HOST:mytraininghostname.com -H app:httpd
```
#### Expected result: 
- A secure Ingress is created and does TLS termination


## Installing istio
#### Steps
- In Cloud Shell, download and install helm binary
- Use the below BASH script, copy entire block and paste in Cloud Shell
```
mkdir -p ~/.tools/istioctl
pushd ~/.tools/istioctl
export ISTIOCTLVERSION=https://github.com/istio/istio/releases/download/1.1.8/istio-1.1.8-linux.tar.gz
wget $ISTIOCTLVERSION
tar -xvf ${ISTIOCTLVERSION##*/}
mv istio-1.1.8/bin/istioctl .
export PATH="$PATH:$HOME/.tools/istioctl"
echo 'export PATH="$PATH:$HOME/.tools/istioctl"' >> ~/.bashrc
popd
echo Istioctl Installation DONE
```
- Run "istioctl get all" 
#### Expected result: 
- istioctl is functional

## Deploy and test sample workloads
#### Steps
- Go to Cloud Shell and verify kubectl is pointed to this cluster
- Configure istio sidecar injects
```
kubectl label namespace default istio-injection=enabled
```
- Apply routing excercise 001
```
kubectl apply -f definitions/istio_routing_001.yaml
```
- Go to GCP console, and open pod details. You should see istio sidecar being injected
- Go to GCP console, and open frontend service, no backend pods are registered
- get into centos container
```
kubectl exec -it $(kubectl get pod -l app=centos -o jsonpath="{.items[0].metadata.name}") bash
```
- Access the frontend endpoint inside the mesh and repeat multiple times - nginx and httpd responses are returned
```
curl frontend.default.svc.cluster.local
```
- Edit frontend virtual service and place nginx weight to 90 and 10 to httpd
```
kubectl edit virtualservice/frontend
```
verify that doing "curl frontend.default.svc.cluster.local" responses from nginx are returned more often.

- Verify how the application can route requests based on Header - only HTTPD responses should be returned
```
curl frontend.default.svc.cluster.local -H app:httpd
```
- Go to GCP console and copy the external IP address of istio-ingressgateway	load balancer
and try to curl the endpoint from your system
```
export IP_OF_THE_LB=127.0.0.1
curl http://$IP_OF_THE_LB/ 
```
- Add host header to verify the routing
```
curl http://$IP_OF_THE_LB/ -H HOST:mytraininghostname.com
curl http://$IP_OF_THE_LB/ -H HOST:mytraininghostname.com -H app:httpd
```

#### Expected result: 
- Application and istio definitions are applied
- Routing works as expected


## Check the security feature - MTLS
#### Steps
- Let's assume an unexpected internal endpoint was exposed accidentaly with a node
port or a Load Balancer
```
kubectl apply -f definitions/istio_mtls_001.yaml
```
- Go to GCP console and copy the external IP address of the httd
- Try to open it in browser - blocked
- Access it with https
```
export IP_OF_THE_EXPOSED_LB=34.73.132.45
curl -kv https://$IP_OF_THE_EXPOSED_LB:80
```
- get into centos container
```
kubectl exec -it $(kubectl get pod -l app=centos -o jsonpath="{.items[0].metadata.name}") bash
```
- Curl the exposed endpoint - should be successful, as it is inside the mesh

```
curl httpd-lb

```
#### Expected result: 
- MTLS needs to be applied


## Check the security feature - JWT AUTH
#### Steps
- Let's assume we need to protect an endpoint and enable JWT AUTH
```
kubectl apply -f definitions/istio_jwt_001.yaml
```
- Let's try to access the endpoint
```
export IP_OF_THE_INGRESS=127.0.0.1
curl -k https://$IP_OF_THE_INGRESS/ -H HOST:httpd.com
```
- Let's try to JWT
```
export IP_OF_THE_INGRESS=127.0.0.1
TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.1/security/tools/jwt/samples/demo.jwt -s)
curl -H "Authorization: Bearer $TOKEN" -H HOST:httpd.com https://$IP_OF_THE_INGRESS/
```
- Clean the routing example
```

kubectl delete -f definitions/
```
#### Expected result: 
- You can access endpoint with a token but cannot without it


## Check the security feature - RBAC
#### Steps
- Deploy services and enable RBAC
```
kubectl apply -f definitions/istio_service_role_001.yaml
```

- Get into centos container
```
kubectl -n sr exec -it $(kubectl -n sr get pod -l app=centossr -o jsonpath="{.items[0].metadata.name}") bash
```