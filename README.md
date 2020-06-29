# Deploying Apache Airflow to OpenShift using the standard Helm chart

This repo deploys Apache Airflow to OpenShift. It is based on the upstream [Airflow Helm chart](https://github.com/helm/charts/tree/master/stable/airflow) and adds the following features:

1. Configured KubernetesExecutor
2. Airflow pods run with SCC anyuid
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

Some of the Airflow pods will restart. This is caused by the PostgreSQL database not being ready yet. Eventually, the PostgreSQL comes up and the pods will stop restarting.

```
$ oc get pod
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-postgresql-0                 1/1     Running   0          90s
airflow-scheduler-6878f776bc-lwmcg   1/1     Running   2          90s
airflow-web-6c7d584ddd-22dd9         1/1     Running   1          90s
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

Remove the database volume:

```
$ oc delete pvc data-airflow-postgresql-0
```

Delete the entire project:

```
$ oc delete project airflow
```

## Configuring git-sync

This section configures Airflow to pull the DAGs from a git repository. You must edit the `values.yaml` file. An example configuration looks as follows:

```yaml
airflow:
  config:
    AIRFLOW__KUBERNETES__DAGS_IN_IMAGE: "False"
    AIRFLOW__KUBERNETES__GIT_REPO: https://github.com/noseka1/airflow-example-bookstore
    AIRFLOW__KUBERNETES__GIT_BRANCH: master
    AIRFLOW__KUBERNETES__GIT_DAGS_FOLDER_MOUNT_POINT: /opt/airflow/dags
    AIRFLOW__KUBERNETES__DAGS_VOLUME_SUBPATH: repo
    
dags:
  git:
    url: https://github.com/noseka1/airflow-example-bookstore
    gitSync:
      enabled: true
```

```
