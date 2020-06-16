# Deploying Apache Airflow to OpenShift using the standard Helm chart

```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm repo update
```

```
$ helm pull stable/airflow \
    --version 7.1.5 \
    --untar \
    --untardir .
```
