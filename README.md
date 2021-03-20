# cockroachdb-testing

```
kind create cluster --config=kind-config.yaml \
  --name=crdb-cluster

kubectl cluster-info --context kind-crdb-cluster

helm repo add cockroachdb https://charts.cockroachdb.com/

helm repo update

kubectl create namespace crdb

kubectl config set-context --current --namespace=crdb

helm install my-release --values my-values.yaml cockroachdb/cockroachdb

kubectl certificate approve crdb.node.my-release-cockroachdb-0
kubectl certificate approve crdb.node.my-release-cockroachdb-1
kubectl certificate approve crdb.node.my-release-cockroachdb-2
kubectl certificate approve crdb.client.root

curl -OOOOOOOOO \
https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/client-secure.yaml
```
