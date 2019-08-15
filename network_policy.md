# Excercise - network policies
## Installing helm
#### Steps
- Check the file content definitions/netpolicy_001.yaml
- In Cloud Shell, apply the definition from 
```
kubectl apply -f definitions/netpolicy_001.yaml
```
- Try to curl httpd from centos
```
kubectl -n netpolicy exec -it $(kubectl get pod -n netpolicy -l app=centos -o jsonpath="{.items[0].metadata.name}") curl httpd
```
- Edit the network policy to correctly rappresent the from label,
change centos2 to centps and apply
```
kubectl apply -f definitions/netpolicy_001.yaml
```
Try to ping http from centos again

```
kubectl -n netpolicy exec -it $(kubectl get pod -n netpolicy -l app=centos -o jsonpath="{.items[0].metadata.name}") curl httpd
```
#### Expected result: 
- Without correct network policy rule with enabled network policy apps 
cannot communicate.