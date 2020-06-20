# Deploying Apache Airflow to OpenShift using the standard Helm chart

This repo deploys Apache Airflow to OpenShift. It is based on the upstream [Airflow Helm chart](https://github.com/helm/charts/tree/master/stable/airflow) and adds the following features:

1. Configured KubernetesExecutor
2. SCCs are installed to allow Airflow containers to run as uid=50000
3. PostgreSQL configuration is fixed to work on OpenShift
4. Edge-terminated TLS route is created for accessing the Airflow dashboard

## Deploying Airflow

Initialize Helm:

```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm repo update
```

Clone this git repo:

```
$ git clone https://github.com/noseka1/openshift-airflow-helm
$ cd openshift-airflow-helm
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

Some of the airflow pods will fail to start. This is caused by Helm applying Kubernetes/OpenShift resources in an incorrect order. Helm creates the scc resource only after the pods are already deployed which leads to pods starting without the required privileges:

```
$ oc get pod
NAME                                 READY   STATUS             RESTARTS   AGE
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

Obtain the Airflow web hostname:
```
$ oc get route airflow-web --output jsonpath='{.spec.host}'
```

Then visit `https://<route_hostname>` with your browser.

## Uninstalling Airflow

Uninstall Airflow:

```
$ helm uninstall airflow
```
