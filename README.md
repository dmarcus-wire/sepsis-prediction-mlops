# sepsis-prediction-mlops
Demonstrate MLOps with a healthcare binary classifier to predict patient sepsis

# cluster config
These steps are assuming you have cluster admin access to a Red Hat OpenShift cluster.

## fork this repo into your own GitHub

## airflow

Login to the cluster from a terminal with cluster admin

### Create a new project airflow
```
oc new-project airflow
```

The following steps assume you are working in the airflow project from your terminal.

### Create a secret
```
export GIT_SERVER=github.com
export GITHUB_USER=[add-your-git-username]
export GITHUB_TOKEN=[add-your-token]

apiVersion: v1
data:
  password: "$(echo -n ${GITHUB_TOKEN} | base64)"
  username: "$(echo -n ${GITHUB_USER} | base64)"
kind: Secret
metadata:
  name: git-auth
type: kubernetes.io/basic-auth
```

### ignore symlinks

```
echo "# ignore the symlinked directory" > .airflowignore
echo "my-dags.git" >> .airflowignore
git add .
git commit -m "add ignorefile"
git push
```

Expected output
```
Enumerating objects: 6, done.
```

### install helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Expected output
```
Downloading https://get.helm.sh/helm-v3.11.2-darwin-arm64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
Password:
helm installed into /usr/local/bin/helm
```

### add the eformat repo

```
helm repo add eformat https://eformat.github.io/helm-charts
```

Expected output
```
"eformat" already exists with the same configuration, skipping
```

### update the chart
```
helm repo up eformat
```

Expected output
```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "eformat" chart repository
Update Complete. ⎈Happy Helming!⎈
```

### install airflow

Update the repo to your fork

```
helm upgrade --install airflow \
--set gitSync.repo="https://github.com/dmarcus-wire/sepsis-prediction-mlops" \
--set gitSync.branch="main" \
--set gitSync.wait="10" \
--namespace airflow \
eformat/airflow
```

Expected output
```
Release "airflow" does not exist. Installing it now.
NAME: airflow
LAST DEPLOYED: Fri Apr  7 16:03:20 2023
NAMESPACE: airflow
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### check pods

```
oc get po
```

Expected output
```
NAME                                         READY   STATUS    RESTARTS   AGE
airflow-airflow-postgresql-0                 1/1     Running   0          4m32s
airflow-airflow-redis-0                      1/1     Running   0          4m32s
airflow-airflow-scheduler-6d6bd7fc7f-kcqs6   2/2     Running   0          4m32s
airflow-airflow-web-65f77d5b84-m8cr9         1/1     Running   0          4m32s
airflow-airflow-worker-0                     2/2     Running   0          4m32s
```

### check and visit the route

```
oc get routes
```

## minio

### get this file

```
wget https://raw.githubusercontent.com/redhat-na-ssa/demo-argocd-gitops/main/components/demos/instance/minio/minio.yaml
```

### apply the file to ocp

```
oc apply -f minio.yaml
```

Expected output
```
deployment.apps/minio created
route.route.openshift.io/minio created
route.route.openshift.io/minio-console created
service/minio created
```

### check and visit the minio route

```
oc get route
```
