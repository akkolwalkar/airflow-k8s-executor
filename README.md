# Tutorial install Airflow chart with Kubernetes Executor

## Start Minikube
```
minikube start --kubernetes-version=v1.15.4 --memory=4096 --cpus=2
```
## Start Helm
```
helm init
kubectl get pods -n kube-system
```
## Build and install airflow image
```
eval $(minikube docker-env)

docker pull puckel/docker-airflow:latest
docker tag puckel/docker-airflow:latest puckel/docker-airflow:1.10.4
docker build --rm -t puckel/docker-airflow:latest .

helm install --namespace "airflow" --name "airflow" --version 4.1.2 -f airflowk8s_values.yaml stable/airflow

helm ls

kubectl get pods -n airflow

export POD_NAME=$(kubectl get pods --namespace airflow -l "component=web,app=airflow" -o jsonpath="{.items[0].metadata.name}")
echo http://127.0.0.1:8080
kubectl port-forward --namespace airflow $POD_NAME 8080:8080
```

## Copy DAGs manually

```
COPY_THIS_DAG='tutorial_dag.py'
kubectl get pods -n airflow -o jsonpath="{.items[0].metadata.name}" -l "component=scheduler,app=airflow" | xargs -I {} kubectl cp $COPY_THIS_DAG {}:/usr/local/airflow/dags -n airflow
kubectl get pods -n airflow -o jsonpath="{.items[0].metadata.name}" -l "component=web,app=airflow" | xargs -I {} kubectl cp $COPY_THIS_DAG {}:/usr/local/airflow/dags -n airflow



COPY_THIS_DAG='k8s_sample_dag.py'
kubectl get pods -n airflow -o jsonpath="{.items[0].metadata.name}" -l "component=scheduler,app=airflow" | xargs -I {} kubectl cp $COPY_THIS_DAG {}:/usr/local/airflow/dags -n airflow

kubectl get pods -n airflow -o jsonpath="{.items[0].metadata.name}" -l "component=web,app=airflow" | xargs -I {} kubectl cp $COPY_THIS_DAG {}:/usr/local/airflow/dags -n airflow
```

## For clear airflow pods
```
helm delete  --purge "airflow"

# clean error pods
kubectl get pods -n airflow | tail -n +2 | awk '{print $1}' | xargs kubectl delete pod -n airflow
```


## Install Kubernetes Dashboard
```

# https://github.com/helm/charts/tree/master/stable/kubernetes-dashboard

# Install dashboard
helm install stable/kubernetes-dashboard --name kubedash --namespace kube-system --set rbac.clusterAdminRole=true

# Install heapster
helm install --name heapster stable/heapster --namespace kube-system

# Checks status
kubectl get pods -n kube-system

# Get token
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep -i "kubernetes-dashboard-token-" |awk '{print $1;}') 

# Expose port locally
export POD_NAME=$(kubectl get pods -n kube-system -l "app=kubernetes-dashboard,release=kubedash" -o jsonpath="{.items[0].metadata.name}")
echo https://127.0.0.1:8443/
kubectl -n kube-system port-forward $POD_NAME 8443:8443

# Delete dashboard
helm delete kubedash --purge
```


