# k8s-lbwreg
k8s configs for lbwreg

# Machine level stuff
NB: the debian stretch apt repository (kubernetes-stretch) doesn't contain
useful packages. So leave it on xenial.

## Run this on each machine:
```shell
apt install -y apt-transport-https curl less
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
curl -s https://download.docker.com/linux/debian/gpg | apt-key add -
echo "deb [arch=amd64] https://download.docker.com/linux/debian stretch stable" | tee -a /etc/apt/sources.list.d/docker.list
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" |  tee -a /etc/apt/sources.list.d/kubernetes.list  
apt-get update
apt install -y \
  kubelet \
  kubeadm \
  kubernetes-cni \
  docker-ce
```

### On master machine:
`kubeadm init`

(Copy the join command)

Setup cluster networking (using weaveworks, for no particular reason other than cargo culting):

`curl -SL "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=172.16.6.64/27" | kubectl apply -f -`

### On client machines:
run the join command given by the master.

`kubeadm join host:port --token TOKEN --discovery-token-ca-cert-hash hash`


# cluster level work:

## Admin interfaces
If you want to use cabin for admin:
```shell
kubectl create serviceaccount "cabin-access"
kubectl create clusterrolebinding "cabin-access" --clusterrole=cluster-admin --user "system:serviceaccount:default:cabin-access"
kubectl describe serviceaccount "cabin-access"
# the above command will display a token name... use that in the next step
kubectl describe secret "cabin-access-token-XXXXX"
# use the token in cabin.
```

## MariaDB
### Setup persistent storage for the database
kubectl create -f pv-iscsi-db.yml
kubectl create -f pvc-iscsi-db.yml

### Setup db password as a secret
kubectl create secret generic mariadb-pass --from-literal=password=<PASSWORD>

## Django
### Setup persistent storage for the static content (and user provided files)
kubectl create -f pv-iscsi-static.yml
kubectl create -f pvc-iscsi-static.yml

### Setup ingress with SSL
