# Setup LBW registration services/pods.

on each client, configure whatever is needed for persistent storage
- for nfs:
  `apt install nfs-common rpcbind portmap`
- for iscsi:
  `open-iscsi`

## MariaDB
### Setup persistent storage for the database
kubectl create -f pv-nfs-db.yml
kubectl create -f pvc-nfs-db.yml

### Setup db root password as a secret
kubectl create -f mariadb-root-pass.yml

### Setup db user/passwordas a secret
kubectl create -f mariadb-credentials.yml

## Django
### Setup persistent storage for the static content (and user provided files)
kubectl create -f pv-nfs-static.yml
kubectl create -f pvc-nfs-static.yml

### Setup mail details
edit lbwreg_emails.yml
kubectl apply -f lbwreg_emails.yml 

### Setup instance details
edit lbwreg_details.yml
kubectl apply -f lbwreg_details.yml

### Setup ingress with SSL
