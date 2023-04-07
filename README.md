# sepsis-prediction-mlops
Demonstrate MLOps with a healthcare binary classifier to predict patient sepsis

# cluster config

## airflow

### Create a new project airflow
```
oc new-project airflow
```

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

### install helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```


## minio
