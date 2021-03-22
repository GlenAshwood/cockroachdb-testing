# cockroachdb-testing

Orchestrate a Local Cluster with Kubernetes (Kind)

## Dependencies
- [Git](https://git-scm.com/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Helm v3.x](https://helm.sh/docs/intro/install/)

## Clone Repo

To follow along, please clone the **cockroachdb-testing** repo
```
git clone https://github.com/GlenAshwood/cockroachdb-testing.git
cd cockroachdb-testing
```
## Setup a Kind Cluster

Create a kind Cluster
```
kind create cluster --config=kind-config.yaml \
  --name=test-cluster
```

Create db namespace and set context
```
kubectl create namespace db
kubectl config set-context --current --namespace=db
```
Enable the *NGINX Ingress controller*:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```
Now the Ingress is all setup. Wait until it is ready to process requests:
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

Install the [Helm client](https://helm.sh/docs/intro/install/) (version 3.0 or higher) and add the cockroachdb chart repository
```
helm repo add cockroachdb https://charts.cockroachdb.com/
helm repo update
```
Install the CockroachDB Helm chart
```
helm install my-test --values my-values.yaml cockroachdb/cockroachdb
```

Get the names of the Pending CSRs:
```
kubectl get csr
```
Approve the node CSRs
```
kubectl certificate approve db.node.my-test-cockroachdb-0
kubectl certificate approve db.node.my-test-cockroachdb-1
kubectl certificate approve db.node.my-test-cockroachdb-2
kubectl certificate approve db.client.root
```
Confirm that three pods are Running successfully and the Init proccess has Completed(this can take 2 to 3 minutes):
```
kubectl get pods
```
expected output:
``` bash
NAME                             READY   STATUS      RESTARTS   AGE
my-test-cockroachdb-0            1/1     Running     0          2m31s
my-test-cockroachdb-1            1/1     Running     0          2m31s
my-test-cockroachdb-2            1/1     Running     0          2m31s
my-test-cockroachdb-init-d2sm9   0/1     Completed   0          2m31s
```
Confirm that the persistent volumes and corresponding claims were created successfully for all three pods:
```
kubectl get pv
```
expected output:
``` bash
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                              STORAGECLASS   REASON   AGE
pvc-767eb0c5-c0e3-4629-bf89-10f1dfa2edf0   2Gi        RWO            Delete           Bound    db/datadir-my-test-cockroachdb-2   standard                3m34s
pvc-80d4aa92-d339-44b5-b18e-c1b7813585dc   2Gi        RWO            Delete           Bound    db/datadir-my-test-cockroachdb-1   standard                3m35s
pvc-d46636cb-c2f4-4f32-ac3b-eb307cf00d11   2Gi        RWO            Delete           Bound    db/datadir-my-test-cockroachdb-0   standard                3m35s
```
## Use the built-in SQL client

Use the *client-secure.yaml* file to launch a pod running an SQL client and keep it running indefinitely:
```
kubectl create -f client-secure.yaml
```
expected output
``` bash
pod/cockroachdb-client-secure created
```
Confirm that client pod is Running successfully:
```
kubectl get pod cockroachdb-client-secure
```
expected output:
``` bash
NAME                        READY   STATUS    RESTARTS   AGE
cockroachdb-client-secure   1/1     Running   0          2m30s
```
Get a shell into the pod and start the CockroachDB [built-in SQL client](https://www.cockroachlabs.com/docs/v20.2/cockroach-sql):
```
kubectl exec -it cockroachdb-client-secure \
-- ./cockroach sql \
--certs-dir=/cockroach-certs \
--host=my-test-cockroachdb-public
```
expected output:
``` bash
#
# Welcome to the CockroachDB SQL shell.
# All statements must be terminated by a semicolon.
# To exit, type: \q.
#
# Server version: CockroachDB CCL v20.2.6 (x86_64-unknown-linux-gnu, built 2021/03/15 16:04:08, go1.13.14) (same version as client)
# Cluster ID: 65635dbb-e646-44cf-aed2-c4ff93a82b34
#
# Enter \? for a brief introduction.
#
root@my-test-cockroachdb-public:26257/defaultdb>
```

Run some basic CockroachDB SQL statements:
```
CREATE DATABASE contacts;

CREATE TABLE contacts.entries (id SERIAL PRIMARY KEY, name VARCHAR(30), email VARCHAR(30));

INSERT INTO contacts.entries (name, email)
  VALUES ('Jerry', 'jerry@example.com'), ('George', 'george@example.com');

SELECT * FROM contacts.entries;
```
expected output:
``` bash
          id         |  name  |       email
---------------------+--------+---------------------
  643090926644822017 | Jerry  | jerry@example.com
  643090926644953089 | George | george@example.com
(2 rows)
```
[Create a user with a password](https://www.cockroachlabs.com/docs/v20.2/create-user#create-a-user-with-a-password) (You will need this username and password to access the DB Console later):

```
CREATE USER roach WITH PASSWORD 'N0tS3cure';
```
Assign roach to the admin role (you only need to do this once):
```
GRANT admin TO roach;
```
Exit the SQL shell and pod:
```
\q
```
## Accessing the DB Console

Setup Ingress for DB Console:
```
kubectl apply -f ingress.yaml
```
Go to https://localhost and log in with the username and password you created earlier.

## Simulate node failure

Terminate one of the CockroachDB nodes:
```
kubectl delete pod my-test-cockroachdb-2
```
expected output
``` bash
pod "my-test-cockroachdb-2" deleted
```
In the DB Console, the Cluster Overview will soon show one node as Suspect. As Kubernetes auto-restarts the node, watch how the node once again becomes healthy.

Back in the terminal, verify that the pod was automatically restarted:
```
kubectl get pod my-test-cockroachdb-2
```
expected output
``` bash
NAME                    READY   STATUS    RESTARTS   AGE
my-test-cockroachdb-2   0/1     Running   0          10s
```
## Destroy Kind Cluster

Destroy kind Cluster
```
kind delete cluster --name=test-cluster
```

### TODO 

- front end to connect to database.
