

# Background

![K8s Deployment](/doc/k8sdeploy.png)

Provides the declarative YAML files used to deploy an end-to-end web voting application to a Kubernetes cluster. The deployment of the web voting application is used to demonstrate the following K8s resources:
* Nginx Ingress Controller
* Deployments
* Services
* Pods
* StatefulSet-mongo-DB
* Persistent Volumes
* Persistent Volume Claims
* Network Policies
* HPA
* Load Testing

# Mongo K8s Scaling

Cluster Details:

Nodes : 3

Master Node: (2-cpu)

Workers:

CPU : 8 (2-nodes)

RAM : 16GB (total)

Disk Space : 50GB

Kubernetes Version: v1.18.6



# Dynamically provision NFS persistent volumes in Kubernetes  to bind PVC:

sudo mkdir /srv/nfs/kubedata -p

sudo chown nfsnobody: /srv/nfs/kubedata

Installing the required packages on the NFS server:

sudo dnf install nfs-utils

start the nfs-server service:

sudo systemctl enable  nfs-server

sudo systemctl start  nfs-server

Create the file systems to export or share on the NFS server:

vi /etc/exports

Add:

/srv/nfs/kubedata *{rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure}

To export the above file system, run the exportfs command with the -a: 

sudo exportfs  -rav

sudo exportfs -v

To mount the nfs to other node:

mount -t nfs <ip>:/srv/nfs/kubedata /mnt

mount | grep kubedata

unmount /mnt


# Deploy rbac and sc:

cd database

Deploy these manifests using kubectl apply -f rbac.yaml && class.yaml 

Check the status:
kubectl get clusterrole,clusterrolebinding,role,rolebinding | grep nfs
 
Create the pv pod for nfs volume just change the ip of nfs server in manifest and apply.

kubectl apply -f deployment.yaml

To check Make changes to manifests of pvc like sc-name  and apply.

 kubectl apply -f 4-pvc-nfs.yaml
 watch kubectl get pv,pvc
 
It will craete pv automatically and when it  will delete claim will delete the pv also.

Verify  the  file on path:

ls /srv/nfs/kubedata

kubectl -n vote  get all

![K8s Deployment](/doc/Capture0.PNG)
  
  
  # ---mongo-statefulset deployment---
  
 Run the statefulset and service:
  
  kubectl apply -f  mongo-statefulset.yaml
  

kubectl exec -it mongo-0 --mongo
  
  #rs.initiate{}
  
  var cfg = rs.conf{}
  
Add memeber to replica set:
  
  cfg.members[0].host='mongo-0.mongo:27017'
  
  rs.reconfig{cfg}
  
Now added the first member to replica set
 and  similary add  all the memeber aswell:  
  rs.status{}
  
  add similary for all the node
  
  rs.add{"mongo-1.mongo:27017"}
  
   rs.status{}
   
Now mongo db replica up and running with 3 members
   
Access: For that will create a pod for mongodb shell and connect to mongodb rs

   kubectl run mongo --rm -it --image mongo  -- sh
   
connection string:

   mongo mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo  (as we are  using default port no need to add port)
   
   rs.status{}
   
   mongo mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo  --eval 'rs.status{}' | grep name

To create new replica for mongo:  
 kubectl scale sts mongo --replicas 4

   rs.add{"mongo-3.mongo:27017"}

Create ingress network policies to allow inbound traffic from other pods  and ingress controller.
cd netpol

kubectl apply -f .

cd ingress

kubectl apply -f nginx-ingress-controller.yaml

Now deploy the api and  web application  using   deployments and services manifets and as we are using headless service, you can access  both app using  ingress.

cd api

kubectl apply -f .

cd frontend

kubectl apply -f .


kubectl -n vote  get sts

![K8s Deployment](/doc/Capture1.PNG)


For Autoscaling the mongo-db statefulset:

Enable the metrics server and apply the hpa.yaml manifests.

kubectl apply -f hpa.yaml

To verify: 

kubectl exec -it pod-name --container container-name

Then use stress command to increase the cpu load and check:
stress --cpu 2 --timeout 60s

Testing MongoDB on Kubernetes with locust for load testing follow loadtest.txt

	 




