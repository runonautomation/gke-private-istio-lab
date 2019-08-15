# Excercise - helm releases management 
## Installing helm
#### Steps
- In Cloud Shell, download and install helm binary
- Use the below BASH script, copy entire block and paste in Cloud Shell
```
mkdir -p ~/.tools/helm
pushd ~/.tools/helm
export ISTIOCTLVERSION=https://get.helm.sh/helm-v2.14.1-linux-amd64.tar.gz
wget $ISTIOCTLVERSION
tar -xvf ${ISTIOCTLVERSION##*/}
mv linux-amd64/helm .
mv linux-amd64/tiller .
export PATH="$PATH:$HOME/.tools/helm"
echo 'export PATH="$PATH:$HOME/.tools/helm"' >> ~/.bashrc
popd
echo Helm Installation DONE
```
- Create admin binding for current user for the cluster
```
export USER=$(gcloud config get-value account)
kubectl create clusterrolebinding cluster-admin-binding-$(whoami) --clusterrole=cluster-admin --user=$USER
```
- Create helm service account and bind it
```
kubectl create serviceaccount tiller --namespace kube-system
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```
- Initialize helm server component - tiller
```
helm init --service-account=tiller
helm update
```
- Execute "helm version"
#### Expected result: 
- Client and Server versions are returned on "helm version command"
```
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
```



## Installing a helm chart for an environment
#### Steps
- In Cloud Shell, ensure helm is installed and condigured
- Disclaimer: reccomended approach for non testing is using operators, e.g. https://github.com/banzaicloud/kafka-operator
- Update helm to point to a helm charts repository 
```
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
```
- Create a namespace for deploying target workload
```
kubectl create ns kafka
```
- Install kafka, and capture STDOUT of the installation
```
helm install --name kafka --namespace kafka incubator/kafka
```
- Read through the STDOUT of kafka installation for the next step and copy 
the content to a local text editor
- Get resources that were created by the helm chart and review created objects
```
kubectl -n kafka get pods
```
- Go to google cloud console and check newly created resources and service in the UI
#### Expected result: 
- Application definitions are applied

## Testing the kafka infrastructure
#### Steps
- From the output of kafka installation, copy the content of the test client pod
in clipboard
- Create a file kafka-testclient.yaml and paste the content of the pod
- kubectl apply -f kafka-testclient.yaml
- Open shell in container interactively
```
kubectl -n kafka exec -it testclient bash
```
- Work with the test client
```
# Create topic
kafka-topics --zookeeper kafka-zookeeper:2181 --topic test1 --create --partitions 1 --replication-factor 1
# Create some messages and hit CTRL_C to exit
kafka-console-producer --broker-list kafka-headless:9092 --topic test1
# Get the messages
kafka-console-consumer --bootstrap-server kafka:9092 --topic test1 --from-beginning
```
#### Expected result: 
- Kafka infrastructure is working

## Interact with the release
#### Steps
- Check installed releases
```
helm ls
```
- Delete kafka 
```
helm delete --purge kafka
```
#### Expected result: 
- Kafka is deleted