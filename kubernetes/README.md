# kubernates #
* it's is an open source container orchestration tool devloped by google.
* helps you manage containerized applications in different deployment environments.
* it automaticalyy deploys, scales, heals, and manages container across server.
```
Docker		= Conatiner Engine
Kubernetes 	= Container manager
```
## Core Architecture
```
                User
                  |
             Load Balancer
                  |
           Kubernetes Cluster
          /                  \
     Worker Node         Worker Node
          |                   |
      Containers          Containers
	
	Cluster = clollection of machines.
```

## the need for container orchestration tool ##
* trend form monolith to microservices
* increased usage of containers 
* Demand for a proper way of managing those hundreds of containers

## what features do orchestration tools offer ? ##
* High availability or no downtime
* scalability or high performance
* Disaster recovery -> backup and restore 

# Main K8s components #
## 1. Pod ##
* Smallest deployable unit.
```
Pod
 └── Container
```
**examples** 
```
apiversion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
  - name: nginx
    image: nginx
```
**create:**
``` 
$ kubectl apply -f pod.yaml
```
**check:**
```
$ kubectl get pods 
```
## 2. deployment ##
* Real world application use Deploymets, not pods.
**Deployment manages Pods automatically**
```
apiversion: apps/v1
kind: Deployment
metadata:
  name: ghostline
spec:
  replicas: 3
  
  selector:
    matchables:
      app: ghostline
  template:
    metadata:
      app: ghostline
  spec:
    containers:
    - name: ghostline
      image: ghostline: latest
```
* this means : 
```
Ghostline Deployment
	| 
    3 Pods
```
* if one dies:
```
Pod dies
   ↓
Kubernetes notices
   ↓
Creates new Pod
```
* Automatic self healing.
## 3. Service ##
* Pods have changing IPs.
*service give a stable address.
```
kind: Service
```
* Example:
```
Service
   |
   ├── Pod1
   ├── Pod2
   └── Pod3
```
* Traffic automatically load balanced.

## 4. ReplicaSet ##
* keeps desired number of pods alive.
```
replicas: 3
```
* Current
``` 
Pod1
Pod2
```
* kubernetes:
```
Need 3
Have 2
Create 1
```
* Usually manages by Deployment.
## 5. Namespace ##
* Used for seprations.
```
$ kubectl get namespaces
```
* Examples:
```
default
kube-system
production
staging
```
* Useful when Ghostline grows.

## 6. ConfigMap ##
* Store configuration seperately.
* Instead of:
```
DB_HOST="localhost"
```
 inside image 
* Use:
```
kind: ConfigMap
```
 Then inject into container.

## 7. Secret ##
* Store sensitive vlaues.
```
JWT_SECRET
API_KEY
AB_PASSWORD
```
Not plain text inside image.

**Essential commands**
* cluster Info
```
$ kubectl cluster-info
```
* View Nodes
``` 
$ kubectl get nodes 
```
* view pods 
```
$ kubectl get pods
```
* View Deployments
```
$ kubectl get deployments
```
* Describe Resource
```
$ kubectl describe pod nginx
```
* Logs 
``` 
$ kubectl logs pod-name
```
* Enter Container
```
$ kubectl exec -it pod-name -- bash
```
* Delete 
```
$ kubectl delete pod nginx
```
* Scaling
current: 
```
replicas: 3
```
Scale:
```
$ kubectl scale deployment ghostline --replicas=5
```
Result: 
```
Pod1
Pod2
Pod3
Pod4
Pod5
```
* Rolling Updates
Old version:
```
GhostLine v1
```
Deploy new image:
``` 
$ kubectl set image deployment/ghostline \
ghostline=ghostline:v2
```
Kubernetes:
```
Create new pod
Verify healthy
Remove old pod 
```
No Downtime
* Kubernets Networking 
Traffic Flow:
```
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pods
```
Remember:
```
Ingress = External Access
Service = Internal Load Balancer
Pod = Application
```
* kubernetes Storage 
Container are temporary.
For persistent data:
```
PersistentVolume
PersistentVolumeClaim
```
Examples:
* PostgresSQL
* MongoDB
* Uploaded files


## Kubernetes commands ## 
**Cluster Commands**
```
$ kubectl cluster-info	-> show kubernetes control plane endpoints.
$ kubectl get notes	-> Shows available worker/control nodes.
$ kubectl describe node minikube	-> Detaild node info (eg: cpu,memory,running pods,etc.)
```
**Pod Commands**
```
$ kubectl get pods	-> list all pods 
$ kubectl get pods -w	-> watch pods live, updates continously
$ kubectl get pods -o wide	-> show more details Displays= podIp, node, internal networking
$ kubectl describe pod pod-name		-> describes events, images Errors,etc.
$ kubectl logs POD-NAME		-> view logs.
$ kubectl log -f POD-NAME	-> follows logs live. similar to tail -f.
$ kubectl exec -it POD-NAME -- bash/sh		-> open shell inside container useful for debugging.
$ kubectl delete pod POD-NAME 		-> DELETE pod if managed by deployment.
```
**Deployment Commands**
```
$ kubectl get deployments 	-> view deployments.
$ kubectl describe deployment nginx 	-> detailed deployment info.
$ kubectl create deployment nginx --image=nginx		-> create deployment
$ kubectl scale deployments nginx --replicas=5 		-> scale deployments (eg: 5 pods running.) trigger rolling update.
$ kubectl rollout restart deployment nginx	-> restart deployments.
$ kubectl rollout status deployment nginx 	-> check rollout status.
$ kubectl rollout undo deployment nginx		-> rollback deployments, useful after bad updates.
```
**Service Commands**
```
$ kubectl get services/svc 	-> view services
$ kubectl describe svc SERVICE-NAME	-> describe service
$ kubectl delete svc SERVICE-NAME	-> delete service
```
**Namespace Commands**
```
$ kubectl get namespaces	-> list namespaces
$ kubectl create namespaces NAME	-> Create namespaces
$ kubectl get pods -n NAME 	-> Use namespaces 
$ kubectl delete namespace NAME 	-> delete namespaces.
```
**YAML File Operations**
```
$ kubectl apply -f FILENAME.yaml -> apply configurations creates or updates resources.
$ kubectl apply -f . 	-> apply entire directory (eg: applies all YAML file)
$ kubectl delete -f FILENAME.yaml 	-> delete resource from file.
$ kubectl apply --dry-run=client -f FILENAME.taml 	-> validate without creating.
```
**Resource inspection**
```
$ kubectl get all 	-> get everything. (eg: pods, services, etc.)
$ kubectl get deployment NAME -o yaml -> output YAML.
$ kubectl get deployment NAME -o json -> output json.
```
**ConfigMap Commands**
```
$ kubectl create configmap app-config \
  --from-literal=ENV=production 	-> create configmap
$ kubectl get configmaps 	-> view configmaps 
$ kubectl describe configmap APP-CONFIG 	-> describe configmap 
```
**Secret Commands**
```
$ kubectl get secrets	-> view secrets.
$ kubectl create secret generic DB-SECRET \
  --from-literal=password=SUPERSECRET		-> CREATE secret.
$ kubectl describe secret DB-SECRET 	-> DESCRIBE secret
$ kubectl get secret DB-SECRET \
  -o jsonpath='{.data.password}' | base64 -d 	-> decode secret.
```

**Debugging Commands**
```
$ kubectl get events 	-> show events, shows recent failures.
$ kubectl get events --sort-by=.metadata.creationTimestamp 	-> short event by time.
$ kubectl top pods 	-> show resource usage eg: CPU or RAM, requires matrics server.
$ kubectl top nodes 	-> node resource usage.
```

**Port Forwarding**
```
$ kubcectl port-forward PODNAME 8080:80		-> access-pod locally
```

**Minikube Commands**
```
$ minikube start 	-> start cluster
$ minikube stop 	-> stop cluster
$ minikube delete 	-> delete cluster
$ minikube dashboard 	-> open dashboard
$ minikube ip 		-> get cluster IP
```
## Imperative Demo ##
```
$ kubectl create deployment nginx --image=nginx  -> Create deployment.
$ kubectl get deployments 			-> check
$ kubectl get pods 				-> check
$ kubectl scale deployments nginx --replicals=3	-> scale 
$ kubectl get pods -w 				->  watch
$ kubectl delete pod POD_NAME			-> delete one pod
	pod deleted 
	    |
	new pod automatically created 
$ kubectl logs POD_NAME 			-> view logs.
$ kubectl exec -it POD_NAME -- bash 		-> enter container.
$ kubectl delete deployments nginx		-> delete deployment.
```
 
## Config Map ##
-> A ConfiMap is a kubernetes object used to store non-sensitive configuration data.
-> No need to hardcode configuration.
```
Eamples: 
* Hostnames
* URLs
* Feature flags 
* Ports
* environment names
* Logging settings
eg: 
DB_HOST=postgres
DB_PORT=5432
APP-ENV=production
```
* NEVER STORE:
```
Passwords
JWT Secrets
API Keys
Private Keys
Tokens 
* use secrets instead
```
**Creating ConfigMap Using YAML**
Example: 
```
apiVersionn: v1
kind: ConfigMap
metadata:
  name: ghostline-config
data:
  APP_ENV: "production"
  DB_HOST: "postgres"
  DB_PORT: "5432"
  LOG_LEVEL: "debug"

* save 
config-map.yaml

* Apply 
$ kubectl apply -f config-map.yaml
```
**verify it exists**
```
$ kubectl get configmaps 
```
**Import Entire ConfigMap**
instead of:
```
env:
- name: APP_ENV
* for every variable 
USE:
envFrom:
- configMapRef:
    name: config-map.yaml
* All values imported automaticlly.
```
**ConfigMap As Mounted Files**
* extremely important 
ConfigMap:
```
data:
  config.yaml: |
    server:
      port: 8080
    logging:
      level: debug
```
Deployment:
```
volumeMounts:
- name: config-volume
  mountPath: /app/config
volumes:
- name: config-volume
  configMap:
    name: config-map
```
Inside container:
```
/app/config/config.yaml
* exists automatically
```

Visualize:
``` 
ConfigMap
   |
   |
Mounted
   |
   |
/app/config/config.yaml
```
Useful for:
* nginx configs 
* Application configs
* JSON files
* YAML files

**Editing ConfigMap**
Current:
```
$ kubectl edit configmap SERVICE-CONFIG
```
Opens editor:
```
data:
  APP_ENV: production
```
Change:
```
data:
  APP_ENV: staging
```
Save.
Kubernetes updates object.

**Patch ConfigMap**
```
$ kubectl patch configmap service-config \
  -p '{"data":{"APP_ENV":"staging"}}'
```
**Delete ConfigMap**
```
$ kubectl delete configmap service-config
```

**Important Real-world Behavior**
* if configMap changed then pods doesnot updates automatically, environment variables not changed
```
ConfigMap updated 
	|
pods atill uses old value 
```
Need Restart:
```
$ kubectl rollout restart deployment service_name
```
-> Mounted files : kubernetes refreshes mounted files periodically.

**ConfigMap Lifecycle**
```
Create ConfigMap
	|
	V
Store Configuration
	|
	V
Deployment Refrences It
	|
	V
Pod Receives Variable
	|
	V
Application Uses Variables
	|
	V
Config changes
	|
	V
Restart Deployment
```

### Storage ###
-> Container are ephemeral if server crash data will lost.
Example:
```
applicaton saves /uploads/profile.jpg
* everything works 
```
* then container crashes 
* kubernetes creates new container so data willbe lost
```
container
    |
temporary filesyatem 
* when container dies filesystem dies too
```
**kubernetes Solution**
* Volumes 
-> A volume exists independently from the container.
```
container
   |
 Volume
```
-> Container can die. Volume remains 
**What is Volume**
-> A Volume is storage attached to a Pod.
```
Pod
 ├── Container A
 ├── Container B
 └── Volume
```
* Both containers can access same files.

**Basic Volume Example**
```
volumes: 
- name: my-volume
```
Container mounts it:
```
volumeMounts:
- name: my-volume
  mountPath: /data

Result:
Inside Container
/data

acts like storage.
```
**Volume Types**
-> most important types:
```
emptyDir
hostPath
PersistentVolume
PersistentVolumeClaim
```
* emptyDir 
-> simplest volume created when pod starts, Destroyed when pod dies.
```
apiVersion: v1
kind: pod 
metadata:
  name: test
spec:
  containers:
  - name: app
    image: nginx

  volumeMounts:
  - name: cache
    mountPath: /cache

  volumes:
  - name: cache
    emptyDir: {}
```
```
Behavior:
Pod Starts
    |
Volume Created
    |
Data Stored
    |
Pod Deleted
    |
volume Deleted
```
-> used for cache, temporary files, processing data, scratch space NOT for databases / uploads.
* hostPath 
-> This mounts a real directory from the node.
```
Example node:
worker Node
/gome/storage 
```
Mount into Container:
```
volumes:
- name: uploads
  
  hostPath:
    path: /home/storage


*Container:

volumeMounts:
- name: uploads
  mountPath: /uploads
```
```
Node Filesystem
/home/storage
	|
	|
    Container
/uploads
*same directory
```
create example:
```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo

spec:
  container:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /uploads
      name: storage
  volumes:
  - name: storage
    hostPath:
      path: /data/uploads
      type: DirectoryOrCreate
```
Node: /data/uploads
container: /uploads -> same storage.

DirectoryOrCreate -> if directory doesn't exist kubernetes creates it.
Pros:
-> easy, fast, good for learning
Cons: 
```
Imagine Cluster:
* Node 1
* Node 2
* Node 3

Data stored on Node1
Pod moves to Node 2 -> file disappear.
Because storage stayed on node1.

Therefore:
Devlopment okey
Testing okey
Production Not
 * Usually avoid hostpath in production.
```
* Persistent Volumes 
-> now we enter real-world production.
->> it solves problem pod move around >node fails > need storage independent of pods.
kubernetees solution:
```
PersistentVolume (PV)

Real Storage
	|
Persistent volume
	|
	pods
```
-> PV is a storage resource inside kubernetes.

```
Examples: 
Local disk
NFS
AWS EBS
Google Disk
Ceph
NAS

* kubernetes treates all as storage.
```
Example PV
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: sapp-pv
spec:
  capacity:
    sstorage: 10Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/sapp


$ kubectl apply -f pv.yaml  -> apply

Meaning:
* 10 GB storage Available in cluster.
Not attched to any pods yet.>>>

* Persistent Volume 10 Gb  waiting

```
* Persistent Volume Claim(PVC)
-> Now application asks for storage.
```
PV = Storage Resource
PVC = Storage Request


Example:
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sapp-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

$ kubectl apply -f pvc.yaml
```
kubernetes finds
```
PVC wants 5GB
PV has 10GB
match. and bind.

PVC
 |
Bound
 |
 PV

check:
$ kubectl get pvc

output:
NAME		STATUS
SAPP-PVC 	Bound
```

**Using PVC in Deployments**
```
apiVersion: app/v1
kind: Deployment

metadata:
  name: ghostline
spec:
  replicas: 1
  
  selector:
    matchLabels:
    app: sapp
  template:
    metadata:
    labeles:
      app: sapp
  
    containers:
    - name: sapp
      image: sapp
      volumeMounts:
      - mountPath: /uploads
        name: uploads
    volumes:
    - names: uploads
      persistentVolumeClaim:
        claimName: sapp-pvc

Flow:
sapp Pod
   |
  PVC
   |
  PV
   |
Physical
storage
```
**Access Modes**
-> important 
* ReadWriteOnce (RWO)
```
One Node
Read + write 
* used by most databases.
Example:
PostgreSQL
MySQL
MongoDB
```
* ReadOnlyMany (ROX)
```
* may pods read only
```
* ReadWriteMany (RWX)
```
Many Pods 
Read + Write


Used with:
NFS
Ceph
GlusterFS

* Useful for uploads
```
**Reclaim Policies**
-> control what happens when PVC deleted.
* Retain 
```
PVC Deleted
    |
Data Stays
-> safest
```
* Delete
```
PVC Deleted
     |
Storage Deleted
 -> Dangerous
```

## Ingress ##
* Every application needs its own ip which is expensive and messy.
 so with ingress 
```
Internet
      |
      |
Ingress
 ┌────┴────┐
 │         │
GhostLine  API
```
* One publuc IP can run many applications.

Example:
```
ghostline.com
	|
	V
Ghostline Service

api.ghostline.com
	|
	V
  API service
```
Example YAML:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ghostline-ingress

spec:
  rules:
  - host: ghopstline.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ghostline-service
            port:
              number: 80
```
Think:
```
Ingress = Reverse Proxy
```
Usually powered by NGINX Ingress Controller

## Secret ##
* A secret is a kubernetes object used to store confidential information.
Example: kind: Secret
* secret is not exactly encryption By Default it is only base64 encoded anyone can decode it.
* Real encryption usually comes from:
-> etcd encryption
-> cloud KMS
-> Vault
**Create Secret Using CLI**
```
$ kubectl create secret generic sapp-Secret \
  --from-literal=DB_PASSWORD=subpersecret \
  --from-literal=JWT_SECRET=myjwrkey

Verify:
$ kubectl get secrets
 OutPut:
  ghostline-secret
```

**Create Secret Using YAML**
* First encode 
```
echo -n "supersecret" | base64
```
Secret YAML:
```
apiVersion: v1
kind: Secret

metadata:
  name: ghostline-secret

type: Opaque

dtaa:
  DB_PASSWORD: c3VwZXJzZWNyZXQ=

Apply: 
$ kubectl apply -f secret.yaml
```
**Secret Types**
most common "Opaque" > generic secrets.

others:
```
kubernetes.io/tls
kubernetes.io/dockerconfigjson
```
Used for TLS certs and registry authentication.

**Using Secrets As Environments Variables**
most common method.
Secret:
```
DB_PASSWORD 
JWT_SECRET
```
Deployment:
```
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: ghostline-secret
      key: DB_PASSWORD
```
Application:
```
Go: 
password  := os.GetEnv("DB_PASSWORD")

Result: supersecret
```
**Import Entire Secret**
Instead of: env: many times.

USE:
```
envFrom:
- secretRef:
    name: ghostline-secret

Result:
DB_PASSWORD
JWT_SECRET
SMTP_PASSWORD
```
Automatically loaded.

**Mount Secret As Files**

* Very common for certificates.

secret: tls.cert, tls.key
Mount:
```
volumeMounts:
- name: tls-volume
  mountPath: /certs

VOLUME->>>>

volumes:
- name: tls-volume

  secret:
    secretName: ghostline-secret

CONTAINER->>>>>>>>
/certs/tls.cert
/certs/tls.key
```
appear automatically

**Secret Lifecycle**
```
Create Secret
	|
store Secret
	|
Deployment Refrences It
	|
Pod Receives Value
	|
Application Uses Value
```

**Commands**
```
create:
$ kubectl get secrets

List:
$ kubectl get serets

Describe:
$ kubectl describe secret

View YAML:
$kubectl get secret NAME -o yaml

Delete:
$ kubectl delete secret NAME
```


