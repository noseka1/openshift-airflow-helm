# Deploying Apache Airflow to OpenShift using the standard Helm chart

This repo deploys Apache Airflow to OpenShift. It is based on the upstream [Airflow Helm chart](https://github.com/helm/charts/tree/master/stable/airflow) and adds the following features:

1. Configured KubernetesExecutor
2. Airflow pods run with SCC anyuid
3. PostgreSQL configured to run on OpenShift
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
    --version 7.1.6 \
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

## Configuring git-sync for DAGs

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

Apply the configuration changes using Helm:

```
$ helm upgrade airflow -f values.yaml ./airflow
```

## Configuring remote logging to S3-compatible storage

This section configures Airflow to send logs to the S3 bucket hosted by Noobaa.

Follow the guidance in [Generating a connection URI](https://airflow.apache.org/docs/stable/howto/connection/index.html#generating-a-connection-uri) to generate a connection URI:

For example:
```python
import json
from airflow.models.connection import Connection

c = Connection(
    conn_id='MyS3Conn',
    conn_type='S3',
    host='',
    login='',
    password='',
    extra=json.dumps(
        dict(
            host='https://s3-openshift-storage.apps.cluster-04c1.sandbox321.opentlc.com',
            aws_access_key_id='YSSUAtO4mFqJFW2mGcnJ',
            aws_secret_access_key='F0TJj9O7BP2nrcwaKVWbL1EP4iWy4f3GoiQIt+eM'
        )
    ),
)

print(f"AIRFLOW_CONN_{c.conn_id.upper()}='{c.get_uri()}'")
```
The above Python script will print:

```
AIRFLOW_CONN_MYS3CONN='s3://?host=https%3A%2F%2Fs3-openshift-storage.apps.cluster-04c1.sandbox321.opentlc.com&aws_access_key_id=YSSUAtO4mFqJFW2mGcnJ&aws_secret_access_key=F0TJj9O7BP2nrcwaKVWbL1EP4iWy4f3GoiQIt%2BeM'
```

Edit the `values.yaml` file to configure the remote logging:

```yaml
airflow:
  config:
    AIRFLOW__CORE__REMOTE_LOGGING: "True"
    AIRFLOW__CORE__REMOTE_BASE_LOG_FOLDER: "s3://mybucket"
    AIRFLOW__CORE__REMOTE_LOG_CONN_ID: "MyS3Conn"
    AIRFLOW__CORE__ENCRYPT_S3_LOGS: "False"
    AIRFLOW_CONN_MYS3CONN: 's3://?host=https%3A%2F%2Fs3-openshift-storage.apps.cluster-04c1.sandbox321.opentlc.com&aws_access_key_id=YSSUAtO4mFqJFW2mGcnJ&aws_secret_access_key=F0TJj9O7BP2nrcwaKVWbL1EP4iWy4f3GoiQIt%2BeM'
```

Apply the configuration changes using Helm:

```
$ helm upgrade airflow -f values.yaml ./airflow
```
