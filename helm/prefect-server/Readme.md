# Prefect server helm chart

Very experimental version of helm chart to deploy Prefect server.

## Installation

1. Clone repository
2. From this directory:

```
$ helm install --namespace <namespace> <name> ./prefect-server [helm options]
```
Where `<namespace>` is namespace to install into, and `<name>` is name of instance.

## Options:

See comments in [values.yaml](values.yaml).

### External Database Setup
Review this section if an external database will be used.

Update following in [values.yaml](values.yaml):
- `global.postgresql.postgresqlExternalHost` - Set to DNS hostname of preconfigured PostgreSQL database
- `postgresqlEnabled: false` - Helm will not deploy PostgreSQL
- `postgresql.persistence.enabled: false` - Persistent volumes will not be deployed for PostgreSQL storage.
- `postgresql.global.existingSecret` - Set to `null` if specifying `postgresqlPassword`, else to an existing secret containing `postgresql-password`
- `postgresqlPassword` - Set to user password (can be passed as an environment variable to helm), only if an existing secret is not being used.

If using `postgresqlPassword` the installation can be performed as below:
```
$ export postgresqlPassword="POSTGRESQL_PREFECT_USER_PASSWORD"
$ helm install --namespace <namespace> <name> --set postgresqlPassword=$postgresqlPassword ./prefect-server [helm options]
```

PostgreSQL requirements:
1. PostgreSQL must be version 11 (11.8 has been tested).
2. The database must be accessible by the Kubernetes cluster.
3. The prefect user must have super privileges.

Sample script for Prefect setup in a PostgreSQL database:
```sql
DROP DATABASE IF EXISTS prefect_db;
DROP USER IF EXISTS prefect_username;
CREATE DATABASE prefect_db;
CREATE USER prefect_username WITH ENCRYPTED PASSWORD 'prefect_userpassword';
GRANT ALL PRIVILEGES ON DATABASE prefect_db TO prefect_username;
ALTER USER prefect_username WITH SUPERUSER;
# For RDS instances, use this to grant SUPER:
GRANT rds_superuser TO prefect_username;
```



### Database passwords
*Only applicable when deploying database via the helm chart into kubernetes.*

A password for both db admin user and application user should be supplied. The former is needed to install pgcrypto.

Passwords can either be supplied via:

* `postgresql.global.existingSecret` or `postgresql.global.existingSecret` to specify existing secret.

* Specify both `postgresql.postgresqlPostgresPassword` for
admin, and `postgresql.postgresqlPassword` for application password.

### Ingresses

If ingresses for apollo and ui are enabled, at least one `hosts` entry
for each must be provided.

