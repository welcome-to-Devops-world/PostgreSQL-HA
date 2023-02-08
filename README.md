# PostgreSQL

## Introduction

This Helm chart has been developed based on  chart but including some changes to guarantee high availability such as:

. A new deployment, service have been added to deploy  to act as proxy for PostgreSQL backend. It helps to reduce connection overhead, acts as a load balancer for PostgreSQL, and ensures database node failover.
. Replacing bitnami/postgresql with bitnami/postgresql-repmgr which includes and configures . Repmgr ensures standby nodes assume the primary role when the primary node is unhealthy.

itnami charts can be used with  for deployment and management of Helm Charts in clusters.

## Prerequisites
```
Kubernetes 1.19+
Helm 3.2.0+
```
## Installing the Chart
Install the PostgreSQL HA helm chart with a release name my-release:
```
$ helm repo add my-repo https://charts.bitnami.com/bitnami
$ helm install my-release my-repo/postgresql-ha
```
## Uninstalling the Chart
To uninstall/delete the my-release deployment:
```
$ helm delete --purge my-release
```
Additionally, if persistence.resourcePolicy is set to keep, you should manually delete the PVCs.

##Initialize a fresh instance
The [Bitnami PostgreSQL with Repmgr](https://github.com/bitnami/containers/tree/main/bitnami/postgresql-repmgr) image allows you to use your custom scripts to initialize a fresh instance. You can specify custom scripts using the initdbScripts parameter as dict so they can be consumed as a ConfigMap.

In addition to this option, you can also set an external ConfigMap with all the initialization scripts. This is done by setting the initdbScriptsCM parameter. Note that this will override the two previous options. If your initialization scripts contain sensitive information such as credentials or passwords, you can use the initdbScriptsSecret parameter.

The above parameters (initdbScripts, initdbScriptsCM, and initdbScriptsSecret) are supported in both StatefulSet by prepending postgresql or pgpool to the parameter, depending on the use case (see above parameters table).

The allowed extensions are .sh, .sql and .sql.gz in the postgresql container while only .sh in the case of the pgpool one.


## Use of global variables
In more complex scenarios, we may have the following tree of dependencies
```

                     +--------------+
                     |              |
        +------------+   Chart 1    +-----------+
        |            |              |           |
        |            --------+------+           |
        |                    |                  |
        |                    |                  |
        |                    |                  |
        |                    |                  |
        v                    v                  v
+-------+------+    +--------+------+  +--------+------+
|              |    |               |  |               |
| PostgreSQL HA |  | Sub-chart 1 |  | Sub-chart 2 |
|---------------|--|-------------|--|-------------|
+--------------+    +---------------+  +---------------+
```
The three charts below depend on the parent chart Chart 1. However, subcharts 1 and 2 may need to connect to PostgreSQL HA as well. In order to do so, subcharts 1 and 2 need to know the PostgreSQL HA credentials, so one option for deploying could be deploy Chart 1 with the following parameters:
```
postgresql.postgresqlPassword=testtest
subchart1.postgresql.postgresqlPassword=testtest
subchart2.postgresql.postgresqlPassword=testtest
postgresql.postgresqlDatabase=db1
subchart1.postgresql.postgresqlDatabase=db1
subchart2.postgresql.postgresqlDatabase=db1
```
If the number of dependent sub-charts increases, installing the chart with parameters can become increasingly difficult. An alternative would be to set the credentials using global variables as follows:
```
global.postgresql.postgresqlPassword=testtest
global.postgresql.postgresqlDatabase=db1
```
This way, the credentials will be available in all of the subcharts.

## Upgrading

It's necessary to specify the existing passwords while performing a upgrade to ensure the secrets are not updated with invalid randomly generated passwords. Remember to specify the existing values of the postgresql.password and postgresql.repmgrPassword parameters when upgrading the chart:
```
$ helm upgrade my-release my-repo/postgresql-ha \
    --set postgresql.password=[POSTGRES_PASSWORD] \
    --set postgresql.repmgrPassword=[REPMGR_PASSWORD]
```
### On kubernetes
```
helm install postgresql-ha bitnami/postgresql-ha --set postgresql.password=postgresql --set postgresql.repmgrPassword=repmgr -n landrocker-dev
```

| Note: you need to substitute the placeholders [POSTGRES_PASSWORD], and [REPMGR_PASSWORD] with the values obtained from instructions in the installation notes.

| Note: As general rule, it is always wise to do a backup before the upgrading procedures.
