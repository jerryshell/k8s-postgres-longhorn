# K8s PostgreSQL Longhorn

[K8s](https://kubernetes.io/) + [PostgreSQL](https://www.postgresql.org/) + [Longhorn](https://longhorn.io/)

## Install Longhorn

```bash
apt install open-iscsi -y
```

```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.1/deploy/longhorn.yaml
```

## Git clone

```bash
git clone https://github.com/jerryshell/k8s-postgres-longhorn.git
cd k8s-postgres-longhorn
```

## ConfigMap

```bash
export POSTGRES_DB=postgres
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=your_password
cat k8s/configmap/configmap.yaml | envsubst | kubectl apply -f -
```

## PVC + Deployment + Service

```bash
kubectl apply -f k8s/
```

## Delete

```bash
kubectl delete --ignore-not-found=true -f k8s/ -f k8s/configmap/
```

## Connect to PostgreSQL via kubectl

```bash
kubectl exec -it postgres-UUID -- psql -h localhost -U postgres --password -p 5432 postgres
```

## Backup and Restore PostgreSQL Database

Backup

```bash
kubectl exec -it postgres-UUID -- pg_dump -U postgres -d postgres > db_backup.sql
```

Restore

```bash
kubectl cp db_backup.sql postgres-UUID:/tmp/db_backup.sql
kubectl exec -it postgres-UUID -- /bin/bash
psql -U postgres -d postgres -f /tmp/db_backup.sql
```

## LICENSE

[GNU Affero General Public License v3.0](https://choosealicense.com/licenses/agpl-3.0/)
