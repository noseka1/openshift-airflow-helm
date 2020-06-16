# Deploying Apache Airflow to OpenShift using the standard Helm chart

The standard [Airflow Helm chart](https://github.com/helm/charts/tree/master/stable/airflow) is used.

Clone this git repo:

```
$ git clone https://github.com/noseka1/openshift-airflow-helm
$ cd openshift-airflow-helm
```

Initialize Helm:

```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm repo update
```

Fetch the upstream Airflow Helm chart:

```
$ helm pull stable/airflow \
    --version 7.1.5 \
    --untar \
    --untardir .
```

Add templates from this git repo:

```
$ cp templates/* airflow/templates/
```

Create new project:

```
$ oc new-project airflow
```

Install Airflow:

```
$ helm install --values values.yaml airflow ./airflow
```
