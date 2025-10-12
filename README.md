# tuto-k8up

## Install prerequisites 

Install `kubectl`, `helm` et `kind` (We suppose docker is already insattaled)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version

[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64
chmod 700 kind 
sudo mv ./kind /usr/local/bin/kind

```

Create a cluster and set the context
```bash
kind
kind create cluster
kubectl cluster-info --context kind-kind
kubectl get ns
```

## Install k8up

Install the CRDs K8up
```bash
kubectl apply -f https://github.com/k8up-io/k8up/releases/download/v2.13.1/k8up-crd.yaml --server-side
```

Install K8up in Minikube:
```bash
helm repo add k8up-io https://k8up-io.github.io/k8up
helm repo update
# helm install k8up-io/k8up --generate-name
helm upgrade --install k8up k8up-io/k8up --set k8up.envVars[0].name=PROM_URL --set k8up.envVars[0].value=""

```

## Deploy databases and Apps

```bash
create ns test
kubectl apply -f 000-secrets.yaml
kubectl apply -f 110-postgres-statefulset.yaml
kubectl apply -f 120-mysql-statefulset.yaml
kubectl apply -f 140-wordpress-pvc.yaml
```

For **Odoo**
```bash
kubectl apply -f 150-odoo-deployment.yaml
kubectl -n test port-forward svc/odoo 8069:8069
```
Open a browser at [http://localhost:8069](http://localhost:8069)
Set Odoo master password to `odoo` and create a new database named `odoo` 


## Backup databases, PVCS, and Odoo 

### Manual backup
```bash
kubectl apply -f 200-backup.yaml

kubectl get all
kubectl logs -f --since=60 deployment.apps/k8up-<........>

kubectl -n test get all
kubectl -n test logs -f pod/backup-backup-test-<.............>

restic snapshots
```

### Scheduled backup

```bash
kubectl apply -f 300-schedule.yaml

kubectl -n test describe  schedule  schedule-test

restic snapshots
```


## Debug rustic 

```bash
apt-get install restic
export RESTIC_REPOSITORY="s3:https://s3.adexcloud.dz/backup-test"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"

# Initialize repository (only once)
restic init

# Backup a single file
restic backup /path/to/local/file.txt

# List snapshots
restic snapshots
```

## Restore databases, PVCs and others

```bash
export RESTIC_REPOSITORY="s3:https://s3.adexcloud.dz/backup-test"
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
restic snapshots
restic restore XXXXX --target ./restore
```


## Useful links 
- [https://docs.k8up.io/k8up/2.13/tutorials/tutorial.html](https://docs.k8up.io/k8up/2.13/tutorials/tutorial.html)