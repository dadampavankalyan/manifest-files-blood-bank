# Corrected bankapp Kubernetes manifests

These manifests target the `dev` namespace and use:
- Application pods labeled `app: bankapp`
- MySQL pods labeled `app: app-database`
- Application Service `my-service`
- MySQL headless Service and StatefulSet `mysqldb`
- MySQL service DNS `mysqldb` on port `3306`

## Important before applying
1. Confirm `dadampavankalyan/k8sproject:dbimage` is a MySQL server image that accepts the standard `MYSQL_*` environment variables and stores data in `/var/lib/mysql`. If it is not, adjust the image/env/volume path to match how that image was built.
2. Update the example passwords in `secrets.yml`. For an existing persistent MySQL data directory, MySQL initialization variables generally do not reset existing database users/passwords. You may need to update the account inside MySQL.
3. Ensure the PHP `config.php` reads `DB_HOST`, `DB_PORT`, `DB_USERNAME`, `DB_PASSWORD`, and `DB_NAME` from environment variables. If it hardcodes credentials or uses a different host, update it too.
4. An Ingress resource only works if an NGINX Ingress Controller is installed. The Service is also exposed as NodePort for lab access.
5. Apply only after reviewing the files. StatefulSet storage is persistent; deleting PVCs can delete database data.

## Apply
```bash
kubectl apply -f 00-namespace.yml
kubectl apply -f secrets.yml -f cm.yml
kubectl apply -f db-svc.yml -f statefulset.yml
kubectl apply -f deploy.yml -f app-svc.yml
kubectl apply -f netpol.yml -f pdb.yml -f rq.yml -f hpa.yml -f ingress.yml
kubectl -n dev get pods,svc,pvc
kubectl -n dev get deploy,sts,hpa,ingress
```

## If the MySQL StatefulSet already exists
Changing StatefulSet identity or volumeClaimTemplates may be immutable. Back up any database data first. Do not delete existing PVCs just to make the YAML apply. Review the existing StatefulSet and plan a safe migration if needed.
