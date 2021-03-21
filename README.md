# cockroachdb-testing

### Clone Repo

To follow along, please clone the **cockroachdb-testing** repo
```
git clone https://github.com/GlenAshwood/cockroachdb-testing.git
cd cockroachdb-testing
```
## Setup Kind Cluster

Create kind Cluster
```
kind create cluster --config=kind-config.yaml \
  --name=test-cluster
```

Create db namespace and set context
```
kubectl create namespace db

kubectl config set-context --current --namespace=db
helm repo add cockroachdb https://charts.cockroachdb.com/
```

Install the [Helm client](https://helm.sh/docs/intro/install/) (version 3.0 or higher) and add the cockroachdb chart repository
```
helm repo add cockroachdb https://charts.cockroachdb.com/
helm repo update
```
Install the CockroachDB Helm chart
```
helm install cockroach-db --values my-values.yaml cockroachdb/cockroachdb
```
Get the names of the Pending CSRs:
```
kubectl get csr
```
Approve the CSRs
```
kubectl certificate approve db.node.cockroach-db-cockroachdb-0
kubectl certificate approve db.node.cockroach-db-cockroachdb-1
kubectl certificate approve db.node.cockroach-db-cockroachdb-2
kubectl certificate approve db.client.root
```


