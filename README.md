# cockroachdb-testing

```
kind create cluster --config=kind-config.yaml \
  --name=test-cluster

kubectl cluster-info --context kind-test-cluster

helm repo add cockroachdb https://charts.cockroachdb.com/

helm repo update

kubectl create namespace db

kubectl config set-context --current --namespace=db

helm install cockroach-db --values my-values.yaml cockroachdb/cockroachdb

kubectl certificate approve db.node.cockroach-db-cockroachdb-0
kubectl certificate approve db.node.cockroach-db-cockroachdb-1
kubectl certificate approve db.node.cockroach-db-cockroachdb-2
kubectl certificate approve db.client.root

curl -OOOOOOOOO \
https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/client-secure.yaml
```

```
cockroach start-single-node \
--insecure \
--listen-addr=localhost:26257 \
--http-addr=localhost:8080 \
--background

cockroach sql --insecure
```
