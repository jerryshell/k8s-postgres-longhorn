# K8s PostgreSQL Longhorn

[K8s](https://kubernetes.io/) + [PostgreSQL](https://www.postgresql.org/) + [Longhorn](https://longhorn.io/)

## Install open-iscsi

```bash
sudo apt install open-iscsi
```

## Install Longhorn

```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.6.1/deploy/longhorn.yaml
```

## PostgreSQL ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-secret
  labels:
    app: postgres
data:
  POSTGRES_DB: postgres
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: YOUR_PASSWORD
```

## PostgreSQL PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: longhorn
  resources:
    requests:
      storage: 1Gi
```

## PostgreSQL Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16
          envFrom:
            - configMapRef:
                name: postgres-secret
          env:
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          ports:
            - name: db
              containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-pvc
      volumes:
        - name: postgres-pvc
          persistentVolumeClaim:
            claimName: postgres-pvc
```

## PostgreSQL Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  ports:
    - name: db
      port: 5432
      targetPort: db
  selector:
    app: postgres
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
