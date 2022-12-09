```sh
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc 
alias k=kubectl
complete -F __start_kubectl k

alias kn='kubectl config set-context --current --namespace '

export gen="--dry-run=client -o yaml"
# ex: k run nginx --image nginx $gen

# Get K8S resources
alias kp="kubectl get pods -o wide"
alias kd="kubectl get deployment -o wide"
alias ks="kubectl get svc -o wide"
alias kno="kubectl get nodes -o wide"

# Describe K8S resources 
alias kdp="kubectl describe pod"
alias kdd="kubectl describe deployment"
alias kds="kubectl describe service"

vi ~/.vimrc
set ts=2 sw=2
. ~/.vimrc

# See also https://github.com/kubernetes/examples


Q1: 8%
Context
As an administrator of a small development team, you have been asked to set up a Kubernetes cluster to test the viability of a new application.
Task
You must use kubeadm to perform this task. Any kubeadm invocations will require the use of the --ignore-preflight-errors=all option.

Configure the node ik8s-master-0 as a master node.
Join the node ik8s-node-0 to the cluster.
You must use the kubeadm configuration file located at /etc/kubeadm.conf when initializing your cluster.

The cluster will be considered complete once both nodes are in a Ready state.

Docker is already installed on both nodes and apt has been configured so that you can install the required tools.

Q2: 3%

Create a file: /opt/KUCC00302/kucc00302.txt that lists all pods that implement service bar in namespace default.
The format of the file should be one pod name per line.

k get svc bar -o wide
k get po -l environment=production -o custom-columns=:metadata.name > /opt/KUCC00302/kucc00302.txt

Q3: 2%
From the pod label name=cpu-utilizer, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /opt/KUTR00102/KUTR00102.txt (which already exists).

k top pod -l name=cpu-utilizer -A > /opt/KUTR00102/KUTR00102.txt

Q4: 3%
Create a deployment spec file that will:

Launch 2 replicas of the redis Image with the label pipeline_stage=staging
deployment name: kual00201
Save a copy of this spec file to /opt/KUAL00201/spec_deploy.yaml (or /opt/KUAL00201/spec_deploy.json).

When you are done, clean up (delete) any new Kubernetes API object that you produced during this task.

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: kual00201
  name: kual00201
spec:
  replicas: 2
  selector:
    matchLabels:
      app: kual00201
      pipeline_stage: staging
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: kual00201
        pipeline_stage: staging
    spec:
      containers:
      - image: redis
        name: redis
        resources: {}
status: {}

Q5: Scale the deployment presentation to 6 pods.
kubectl scale --replicas=6 deploy/presentation 

Q6: 7%

Create a snapshot of the etcd instance running at https://127.0.0.1:2379, saving the snapshot to the file path /var/lib/backup/etcd-snapshot.db.
The following TLS certificates/key are supplied for connecting to the server with etcdctl:
CA certificate: /opt/KUCM00302/ca.crt
Client certificate: /opt/KUCM00302/etcd-client.crt
Client key: /opt/KUCM00302/etcd-client.key

ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 \
--cacert="/opt/KUCM00302/ca.crt" \
--cert="/opt/KUCM00302/etcd-client.crt" \
--key="/opt/KUCM00302/etcd-client.key" \
snapshot save /var/lib/backup/etcd-snapshot.db

sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status /var/lib/backup/etcd-snapshot.db

Q7: 9%
Create a Kubernetes secret as follows:
Name: super-secret
username: bob
Create a pod named pod-secrets-via-file, using the redis Image, which mounts a secret named super-secret at /secrets.
Create a second pod named pod-secrets-via-env, using the redis Image, which exports username as TOPSECRET.

k create secret generic super-secret --from-literal='username=bob' $gen
k run pod  pod-secrets-via-file --image=redis --restart=Never $gen > pod-secrets-via-file.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-file
spec:
  containers:
    - name: pod-secrets-via-file
      image: redis
      volumeMounts:
        # name must match the volume name below
        - name: super-secret
          mountPath: /secrets
  # The secret data is exposed to Containers in the Pod through a Volume.
  volumes:
    - name: super-secret
      secret:
        secretName: super-secret

# verify
k exec -it pod-secrets-via-file -- ls /secrets
k exec -it pod-secrets-via-file -- cat /secrets/username \n

apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-env
spec:
  containers:
  - name: pod-secrets-via-env
    image: redis
    env:
    - name: TOPSECRET
      valueFrom:
        secretKeyRef:
          name: super-secret
          key: username

k exec -it pod-secrets-via-env -- /bin/sh -c 'echo $TOPSECRET'

Q8: 4%
Create a pod as follows:

Name: non-persistent-redis
container Image: redis
Volume with name: cache-control
Mount path: /data/redis
The pod should launch in the staging namespace and the volume must not be persistent

apiVersion: v1
kind: Pod
metadata:
  name: non-persistent-redis
spec:
  containers:
    - name: non-persistent-redis
      image: redis
      volumeMounts:
        - name: cache-control
          mountPath: /data/redis
  volumes:
    - name: cache-control
       emptyDir: {}

kp -n staging
k apply -f non-persistent-redis.yaml -n staging

Q9: 3%
Create a pod as follows:
Name: mongo
Using Image: mongo
In a new Kubernetes namespace named: my-application

k run mongo --image mongo --restart=Never $gen > mongo_pod.yaml
k create ns my-application
k get ns
k apply -f mongo_pod.yaml -n my-application
kp -n my-application
kdp mongo -n my-application

Q10: 4%
kubectl config use-context wk8s
A Kubernetes worker node, named wk8s-node-0 is in state NotReady.
Investigate why this is the case, and perform any appropriate steps to bring the node to a Ready state, ensuring that any changes are made permanent.
You can ssh to the failed node using:

ssh wk8s-node-0
You can assume elevated privileges on the node with the following command:
sudo -i

k get nodes
ssh wk8s-node-0
sudo systemctl status kubelet.service
sudo systemctl restart  kubelet.service
sudo systemctl status  kubelet.service

# Durable setting, Always start Kubelet: enable service
systemctl enable kubelet.service 

k get nodes

Q11: 4%
Task
Create and configure the service front-end-service so it's accessible through NodePort and routes to the existing pod named front-end

kp front-end
kdp front-end
k expose pod/front-end --name=front-end-service --type=NodePort  $gen > front-end-service.yaml

Q12: 2%
Schedule a pod as follows:
Name: nginx-kusc00101
Image: nginx
Node selector: disk=spinning

k run nginx-kusc00101 --image nginx --restart=Never $gen > nginx-kusc00101.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-kusc00101
  name: nginx-kusc00101
spec:
  containers:
  - image: nginx
    name: nginx-kusc00101
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  nodeSelector:
    disk: spinning
status: {}


kp -o wide
kdp nginx-kusc00101 

k get node --show-labels
NAME           STATUS   ROLES    AGE   VERSION   LABELS
k8s-master-0   Ready    master   31d   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master-0,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node-0     Ready    <none>   31d   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disk=spinning,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node-0,kubernetes.io/os=linux
k8s-node-1     Ready    <none>   31d   v1.18.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disk=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node-1,kubernetes.io/os=linux

Q13: 4%
Set the node named ek8s-node-1 as unavailable and reschedule all the pods running on it.

# kubectl cordon ek8s-node-1
# kubectl drain ek8s-node-1 --dry-run
kubectl drain ek8s-node-1 --ignore-daemonsets --delete-local-data --force=true
kubectl uncordon ek8s-node-1

Reconfigure the existing deployment front-end and add a port specification named http exposing port 80/tcp of the existing container nginx.
Create a new service named front-end-svc exposing the container port http.
Configure the new service to also expose the individual Pods via a NodePort on the nodes on which they are scheduled.
k create deployment nginx-dns --image=nginx $gen > nginx-dns-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-dns
  name: nginx-dns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-dns
      pipeline_stage: staging
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-dns
        pipeline_stage: staging
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

# add below
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
		  
Q14: 7%
Create a deployment as follows:
Name: nginx-dns
Exposed via a service nginx-dns
Ensure that the service & pod are accessible via their respective DNS records
The container(s) within any pod(s) running as a part of this deployment should use the nginx Image
Next, use the utility nslookup to look up the DNS records of the service & pod and write the output to /opt/KUNW00601/service.dns and /opt/KUNW00601/pod.dns respectively.



# get Pod IP : 10.244.2.14
kdp nginx-dns-7fff7989f4-fw4bg 
k exec nginx-dns-7fff7989f4-fw4bg -it -- cat /etc/resolv.conf
k exec nginx-dns-7fff7989f4-fw4bg  -it -- nslookup nginx-dns-7fff7989f4-fw4bg

k expose deployment/nginx-dns --name=nginx-dns --type=ClusterIP --port=80 $gen > nginx-dns-svc.yaml

kubectl run dns-test -it --rm --image busybox --restart=Never -- nslookup 10.244.2.14 > /opt/KUNW00601/pod.dns

cat /opt/KUNW00601/pod.dns

kubectl run dns-test -it --rm --image busybox --restart=Never -- nslookup 10.107.181.46  >  /opt/KUNW00601/service.dns

cat /opt/KUNW00601/service.dns

kubectl exec dns-test -it --rm --image busybox  --restart=Never  -- nslookup nginx-dns.default

Q15: 3% 
Ensure a single instance of pod nginx is running on each node of the Kubernetes cluster where nginx also represents the Image name which has to be used. 
Do not override any taints currently in place.

Use DaemonSet to complete this task and use ds-kusc00201 as DaemonSet name.

k run ds-kusc00201 --image=nginx --restart=Always $gen > ds-kusc00201-ds.yaml 

apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  labels:
    app: ngninx
  name: ds-kusc00201
spec:
  selector:
    matchLabels:
      app: ngninx
 containers:
    image: nginx
    name: ds-kusc00201
    resources: {}
status: {}

Q16: 4%
Create a pod named kucc8 with a single app container for each of the following images running inside (there may be between 1 and 4 images specified): nginx + redis.

k run kucc8 --image=nginx --restart=Never $gen > kucc8-pod.yaml 

Q17: 7%
Perform the following tasks:
Add an init container to hungry-bear (which has been defined in spec file /opt/KUCC00108/pod-spec-KUCC00108.yaml)
The init container should create an empty file named /workdir/gentle.txt
If /workdir/gentle.txt is not detected, the pod should exit
Once the spec file has been updated with the init container definition, the pod should be created

apiVersion: v1
kind: Pod
metadata:
  name: hungry-bear
spec:
  volumes:
    - name: workdir
      emptyDir:
  containers:
  - name: checker
    image: alpine
    command: ["/bin/sh", "-c", "if [ ! -e /workdir/gentle.txt ]; then exit 1; fi"]
    volumeMounts:
    - name: workdir
      mountPath: /workdir
  initContainers:
    - name: install
      image: busybox
      command: ["/bin/sh", "-c", "touch /workdir/gentle.txt"]
      volumeMounts:
      - name: workdir
        mountPath: "/workdir"
  volumes:
  - name: workdir
    emptyDir: {}

Q18: 4%
k create deployment nginx-app --image=nginx:1.11.10-alpine $gen > nginx-1.11.10-alpine-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-app
  name: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-app
    spec:
      containers:
      - image: nginx:1.11.10-alpine
        name: nginx
        resources: {}
status: {}

# perform rolling update
kubectl set image deployment/nginx-app nginx=nginx:1.12.0-alpine --record
kubectl rollout status  deployment/nginx-app
kubectl rollout history deployment/nginx-app 
# Rollback
kubectl rollout undo deployment/nginx-app 
kubectl rollout status  deployment/nginx-app


Q19: 2%
How many nodes have the NoSchedule ?
kubectl describe node | grep -i taint


Q20: 4%
Configure the kubelet systemd-managed service, on the node labelled with name=wk8s-node-1, to launch a pod containing a single container of Image nginx named webtool automatically. Any spec files required should be placed in the /etc/kubernetes/manifests directory on the node.

Q21: 3%
List all persistent volumes sorted by name, saving the full kubectl output to /opt/KUCC00102/pv_list. Use kubectl's own functionality for sorting the output, and do not manipulate it any further

k get pv --sort-by='.metadata.name'

Q22: 3%
Create a persistent volume with name app-config, of capacity 2Gi and access mode ReadWriteOnce. The type of volume is hostPath and its location is /srv/app-config.

k create pv app-config $gen > app-config-pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/srv/app-config"
	
	
CKA real exam 05/05/2021

Q1: 4%
Context
You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.
Task
Create a new ClusterRole named deployment-clusterrole, which only allows to create the following resource types:

Deployment
StatefulSet
DaemonSet

Create a new ServiceAccount named cicd-token in the existing namespace app-team1.

Bind the new ClusterRole deployment-clusterrole to the new ServiceAccount cicd-token, limited to the namespace app-team1.


k api-resources --namespaced=true

daemonsets                            ds               apps                            true         DaemonSet
deployments                           deploy           apps                            true         Deployment
replicasets                           rs               apps                            true         ReplicaSet
statefulsets                          sts              apps


Q2 4%
Set the node named ek8s-node-1 as unavailable and reschedule all the pods running on it.

Q3 7%
Given an existing Kubernetes cluster running version 1.20.0, upgrade all of the Kubernetes control plane and node components on the master node only to version 1.20.1.

Be sure to drain the master node before upgrading it and uncordon it after the upgrade.


Q4 7%
First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /srv/data/etcd-snapshot.db
CA certificate: /opt/KUIN00601/ca.crt
Client certificate: /opt/KUIN00601/etcd-client.crt
Client key: /opt/KUIN00601/etcd-client.key

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key snapshot save /srv/data/etcd-snapshot.db

ETCDCTL_API=3 etcdctl snapshot restore /srv/data/etcd-snapshot.db --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key


Q5 7%

Create a new NetworkPolicy named allow-port-from-namespace in the existing namespace big-corp.

Ensure that the new NetworkPolicy allows Pods in namespace corp-net to connect to port 80 of Pods in namespace big-corp.

Further ensure that the new NetworkPolicy:

does not allow access to Pods, which don't listen on port 80
does not allow access from Pods, which are not in namespace corp-net


Q6 7%
Reconfigure the existing deployment front-end and add a port specification named http exposing port 80/tcp of the existing container nginx.

Create a new service named front-end-svc exposing the container port http.

Configure the new service to also expose the individual Pods via a NodePort on the nodes on which they are scheduled.

Q7: 7%

Create a new nginx Ingress resource as follows:
Name: ping
Namespace: ing-internal
Exposing service hello on path /hello using service port 5678
The availability of service hello can be checked using the following command, which should return hello:

curl -kL <INTERNAL_IP>/hello

k create ing ping --namespace=ing-internal --rule="/hello=hello:5678" $gen
ks --namespace=ing-internal

Q8: 4%
Scale the deployment loadbalancer to 6 pods.
k scale deployment loadbalancer --replicas=6

Q9: 4%
Schedule a pod as follows:
Name: nginx-kusc00401
Image: nginx
nodeSelector: disk=spinning

k run pod nginx-kusc00401 --image=nginx $gen > nginx-kusc00401.yaml

==> edit yaml to add Node Selector

  nodeSelector:
    disk: spinning


Q10 : 4%
Check to see how many nodes are ready (not including nodes tainted NoSchedule) and write the number to /opt/KUSC00402/kusc00402.txt.

Q11: 4%
Create a pod named kucc8 with a single app container for each of the following images running inside (there may be between 1 and 4 images specified): nginx + redis + memcached.

Q12: 4%
Create a persistent volume with name app-config, of capacity 1Gi and access mode ReadWriteOnce. The type of volume is hostPath and its location is /srv/app-config.

Q13: 7%
Create a new PersistentVolumeClaim:
Name: pv-volume
Class: csi-hostpath-sc
Capacity: 10Mi
Create a new Pod which mounts the PersistentVolumeClaim as a volume:

Name: web-server
Image: nginx
Mount path: /usr/share/nginx/html
Configure the new Pod to have ReadWriteOnce access on the volume.

Finally, using kubectl edit or kubectl patch expand the PersistentVolumeClaim to a capacity of 70Mi and record that change.

k run po web-server --image=nginx $gen > web-server.yaml

Q14: 5%
Monitor the logs of pod bar and:
Extract log lines corresponding to error file-not-found
Write them to /opt/KUTR00101/bar

k logs bar | grep -i "file-not-found" > /opt/KUTR00101/bar
k logs bar | grep -i "file-not-found" > /opt/KUTR00101/bar

Q15: 7%
An existing Pod needs to be integrated into the Kubernetes built-in logging architecture (e.g. kubectl logs). Adding a streaming sidecar container is a good and common way to accomplish this requirement.
Task
Add a sidecar container named sidecar, using the busybox Image, to the existing Pod big-corp-app. The new sidecar container has to run the following command:

/bin/sh -c "tail -n+1 -f /var/log/big-corp-app.log"
Use a Volume, mounted at /var/log, to make the log file big-corp-app.log available to the sidecar container

Q16: 5%
From the pod label name=cpu-utilizer, find pods running high CPU workloads and write the name of the pod consuming most CPU to the file /opt/KUTR00401/KUTR00401.txt (which already exists).

k top pods -l name=cpu-utilizer

Q17: 13%
A Kubernetes worker node, named wk8s-node-0 is in state NotReady.
Investigate why this is the case, and perform any appropriate steps to bring the node to a Ready state, ensuring that any changes are made permanent.
You can ssh to the failed node using:

ssh wk8s-node-0
You can assume elevated privileges on the node with the following command:
sudo -i

kubectl config use-context wk8s
ssh wk8s-node-0

ssh wk8s-node-0
sudo -i service kubelet status
sudo -i service kubelet restart
# Durable setting, Always start Kubelet: enable service
sudo -i systemctl enable kubelet.service 



https://www.katacoda.com/javajon/courses/kubernetes-networking/connectivity

***********************************************************************************

07/05/2021
```



	
