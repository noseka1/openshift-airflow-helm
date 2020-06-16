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

```
$ NAME                                 READY   STATUS             RESTARTS   AGE
airflow-postgresql-0                 1/1     Running            0          4m14s
airflow-scheduler-845cdf9958-zq4xw   0/1     CrashLoopBackOff   5          4m14s
airflow-web-656656f779-2nvwk         0/1     CrashLoopBackOff   5          4m14s
```

Restart the pods to fix the issue:

```
$ oc delete pod --all
```

```
$ oc get pod
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-postgresql-0                 1/1     Running   0          23s
airflow-scheduler-845cdf9958-wgcnv   1/1     Running   0          25s
airflow-web-656656f779-dk9gb         1/1     Running   0          24s
```

Uninstall Airflow:

```
$ helm uninstall airflow
```
