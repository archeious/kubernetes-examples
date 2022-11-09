# Setting up a WordPress site backed by MySQL

## Setup the Mysql Database
1) setup a secret in `mysql-secret.yaml`.
    1) Generate a strong base64 encoded password.  I usually use somethings like `echo $(strings /dev/urandom | grep -o '[[:alnum:]!@#$%^&*()]' | head -n 32 | tr -d '\n'; echo) | base64` 
2) setup persistent storage. `mysql-persistent-storage.yaml` In my case I am using a Ceph backed rook storage class.
    1) The default retention policy for my volumes is delete. I needed to patch the claim to keep the volume around if things die. `kubectl patch pv pvc-9935e4fc-a464-4547-a8f6-90864c91b78b -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'`
3) create a replicaset for the mysql container. `mysql-replicaset.yaml`
4) create the WordPress database.  You will need to attach to the container and run the mysql command.
    1) `kubectl get pods -n kubernetes-examples`  Get the pod name.
    2) `kubectl exec -it -n kubernetes-examples $PODNAME -- bash`
    3) My `mysql -u root -p` and enter the password generated in step 1.1
    4) `create database wordpress;`
    5) exit the MySql CLI and shell

## Setup up WordPress
1) `wp-persistent-volume.yaml` Create a persistence volume claim to store the WordPress files.
2) Create a WordPress deployment `wp-deployment.yaml`.
    - Note that we are using custom namespaces so the host value will need to include the namespace
    - e.g., `mysql-service.kubernetes-example.svc.cluster.local` instead of `mysql-service.default.svc.cluster.local`

