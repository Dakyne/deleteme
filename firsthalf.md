---===---
11. Taints & Tolerations, NodeAffinity/pod-nodeaffinity.yaml

apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  containers:
  - name: myapp
    image: nginx:1.20
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: type 
            operator: In
            values:
            - cpu
            
---===---
11. Taints & Tolerations, NodeAffinity/pod-podaffinity.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 5
  template:
    metadata:
      labels:
        app: myapp
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - myapp
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - etcd
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: myapp-container
        image: nginx:1.20
      nodeSelector: 
        type: master---===---
11. Taints & Tolerations, NodeAffinity/pod-with-node-name.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.20
  nodeName: worker1---===---
11. Taints & Tolerations, NodeAffinity/pod-with-node-selector.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.20
  nodeSelector:
    type: cpu---===---
11. Taints & Tolerations, NodeAffinity/pod-with-tolerations.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-toleration
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx:1.20
  tolerations:
  - effect: NoExecute
    operator: Exists
  nodeName: master
---===---
11. Taints & Tolerations, NodeAffinity/useful-links.md

##### assigning pods to nodes
* Assigning Pods to Nodes: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

##### nodeAffinity
* Node Affinity: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

##### interPodAffinity and antiAffinity
* InterPod Affinity & AntiAffinity: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity

##### taints & tolerations
* Taints and Tolerations: https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/


---===---
12. Readiness & Liveness Probes/pod-health-probes.yaml

apiVersion: v1
kind: Pod
metadata:
 name: myapp-health-probes
spec:
  containers:
  - image: nginx:1.20
    name: myapp-container
    ports:
    - containerPort: 80
    readinessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 15
      

---===---
12. Readiness & Liveness Probes/useful-links.md

##### health probes
* Configure Readiness and Liveness Probes: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
---===---
13. Rolling Updates/commands.md

##### rollout commands
    kubectl rollout history deployment/{depl-name}
    kubectl rollout undo deployment/{depl-name}
    kubectl rollout status deployment/{depl-name}
---===---
13. Rolling Updates/useful-links.md

##### replicaset
* ReplicaSet: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

##### deployment upgrade strategies
* Deployment Strategies: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy
* Rolling Update: https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

---===---
14. Etcd Backup & Restore/commands.md

### Install ectdctl
    sudo apt  install etcd-client

### Backup 

##### snapshot backup with authentication
    ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key

##### check snapshot status
    ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshotdb


### Restore

##### create restore point from the backup
    ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup

##### the restored files are located at the new folder /var/lib/etcd-backup, so now configure etcd to use that directory:
    vim /etc/kubernetes/manifests/etcd.yaml
---===---
14. Etcd Backup & Restore/useful-links.md

##### etcd backup
Back up etcd store: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

##### etcd restore
Restore etcd backup: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#restoring-an-etcd-cluster
---===---
15. K8s Rest API/commands.md

### Access API through proxy
    kubectl proxy --port=8081 &
    curl http://localhost:8081/api/

### Access without kubectl proxy

##### create serviceaccount for myscript usage
    kubectl create serviceaccount myscript

##### create role with Deployment, Pod, Service permissions
    kubectl apply -f myscript-role.yml

##### add Binding for serviceaccount
    kubectl create rolebinding script-role-binding --role=script-role --serviceaccount=default:myscript

##### get config info from kubectl
    kubectl config view

##### set cluster location var
    APISERVER=https://172.31.44.88:6443

##### set token var from default token
    kubectl get serviceaccount myscript -o yaml
    kubectl get secret xxxxx -o yaml

    TOKEN=$(echo "token" | base64 --decode | tr -d "\n")


##### if we don't have the ca cert for curl, we can accept insecure, without providing curl client with ca certificate 
    curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure

##### if we don't want insecure connection, we can specify ca cert for curl providing curl with k8s ca certificate
    curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt


### Get data

##### main endpoint /api
    curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt

##### list all deployments
    curl -X GET $APISERVER/apis/apps/v1/namespaces/default/deployments --header "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt

##### list all services
    curl -X GET $APISERVER/api/v1/namespaces/default/services --header "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt

##### get a specific service or deployment
    curl -X GET $APISERVER/api/v1/namespaces/default/services/nginx-service --header "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt

##### get all pod names
    curl -X GET $APISERVER/api/v1/namespaces/default/pods/pod-name/logs --header "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt
---===---
15. K8s Rest API/myscript-role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: script-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "delete"] 
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "delete"]---===---
15. K8s Rest API/useful-links.md

##### K8s rest API
* Access Cluster using API: https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/
* API Overview: https://kubernetes.io/docs/reference/using-api/
* More about K8s API: https://kubernetes.io/docs/concepts/overview/kubernetes-api/
* K8s REST API Documentation: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#-strong-api-groups-strong-
* Bearer Token: https://kubernetes.io/docs/reference/access-authn-authz/authentication/#putting-a-bearer-token-in-a-request

##### programmatic access
* Programmatic Access: https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#programmatic-access-to-the-api  
* Client Libraries: https://kubernetes.io/docs/reference/using-api/client-libraries/
---===---
16. Upgrade K8s Cluster/commands.md

### Upgrade control plane node

##### check apt-get version
    sudo apt-get --version

##### for apt-get version gt 1.1
    sudo apt-get update
    sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.0-00

##### get upgrade preview
    sudo kubeadm upgrade plan

##### upgrade cluster 
    sudo kubeadm upgrade apply v1.22.0

##### drain node
    kubectl drain master --ignore-daemonsets

##### upgrade kubelet & kubectl 
    sudo apt-get update
    sudo apt-get install -y --allow-change-held-packages kubelet=1.22.0-00 kubectl=1.22.0-00

##### restart kubelet
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet

##### uncordon node
    kubectl uncordon master


### Upgrade worker node
    sudo apt-get update && \
    sudo apt-get install -y --allow-change-held-packages kubeadm=1.22.x-00

    sudo kubeadm upgrade node

    kubectl drain worker1 --ignore-daemonsets --force

    sudo apt-get update
    sudo apt-get install -y --allow-change-held-packages kubelet=1.22.x-00 kubectl=1.22.x-00

    sudo systemctl daemon-reload
    sudo systemctl restart kubelet

    kubectl uncordon worker1
---===---
16. Upgrade K8s Cluster/useful-links.md

##### cluster upgrade
* Cluster Upgrade: https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/
* Upgrading Kubeadm clusters: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
* Drain a Node: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
---===---
17. Contexts with Multiple Clusters/commands.md

##### all context commands
    kubectl config --help

##### show all contexts
    kubectl config get-contexts

##### show current context
    kubectl config current-context

##### switch to another context
    kubectl set-context context-name

##### change user or cluter name for any context
    kubectl config set-context --help
    kubectl config set-context context-name --user=user_name --cluster=cluster_name

##### change user or cluster for current context
    kubectl config set-context --current --user=user_name --cluster=cluster_name

##### change default namespace
    kubectl get pod
    kubectl config set-context --current --namespace=myapp
    kubectl get pod

---===---
17. Contexts with Multiple Clusters/kubeconfig-multiple-contexts.yaml

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: xxxxxx
    server: https://172.31.44.88:6443
  name: development
- cluster:
    certificate-authority-data: xxxxxx
    server: https://251.15.20.12:6443
  name: staging  
contexts:
- context:
    cluster: development
    user: dev-admin
  name: dev-admin@development
- context:
    cluster: development
    namespace: myapp
    user: my-script
  name: my-script@development
- context:
    cluster: staging
    user: staging-admin
  name: staging-admin@staging
current-context: dev-admin@development
kind: Config
preferences: {}
users:
- name: dev-admin
  user:
    client-certificate-data: xxxx
    client-key-data: xxxx
- name: staging-admin
  user:
    client-certificate-data: xxxx
    client-key-data: xxxx 
- name: my-script
  user:
    token: xxxxxxx---===---
17. Contexts with Multiple Clusters/useful-links.md

##### contexts
* Configure Access to multiple clusters: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
* Context: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#context
* kubectl config subcommand: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config
---===---
18. Renew K8s Certificates/commands.md

##### check certificate expirations dates of all k8s certificates with kubeadm
    kubeadm certs --help
    kubeadm certs check-expiration

##### check expiration date of a certificate with openssl
    openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

##### filter resulting cert for validity attribute plus 2 following lines
    openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep Validity -A2
---===---
18. Renew K8s Certificates/useful-links.md

##### k8s certificates 
* Certificate Management with kubeadm: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/
---===---
19. Network Policy/commands.md

### Set namespace to myapp
    kubectl config set-context --current --namespace=myapp
    kubectl get pod -o wide

### Before creating network policies

##### all of these works
    kubectl exec backend-7787844fc5-q2hkk -- sh -c 'nc -v db-ip-10.44.0.9 6379'
    kubectl exec frontend-7d7b95b577-6m7fp -- sh -c 'nc -v backend-ip-10.44.0.8 80'
    kubectl exec frontend-7d7b95b577-6m7fp -- sh -c 'nc -v db-ip-10.44.0.10 6379'
    kubectl exec database-7874cd5f45-jvr5z -- sh -c 'nc -v frontend-ip-10.44.0.6 3000'
    kubectl exec database-7874cd5f45-jvr5z -- sh -c 'nc -v backend-ip-10.44.0.8 80'

### After creating network policies

##### still works
    kubectl exec backend-7787844fc5-q2hkk -- sh -c 'nc -v db-ip-10.44.0.9 6379'
    kubectl exec frontend-7d7b95b577-6m7fp -- sh -c 'nc -v backend-ip-10.44.0.8 80'

##### don't work any more
    kubectl exec frontend-7d7b95b577-6m7fp -- sh -c 'nc -v db-ip-10.44.0.10 6379'
    kubectl exec database-7874cd5f45-jvr5z -- sh -c 'nc -v frontend-ip-10.44.0.6 3000'
    kubectl exec database-7874cd5f45-jvr5z -- sh -c 'nc -v backend-ip-10.44.0.8 80'
---===---
19. Network Policy/demo-database.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  namespace: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: redis
        image: redis:6-alpine
        ports:
        - containerPort: 6379
---===---
19. Network Policy/demo-frontend.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: node
        image: node:16-alpine
        command: ['sh', '-c', "sleep 3000"]
        ports:
        - containerPort: 3000
---===---
19. Network Policy/demo-np-database.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-database
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  - Egress
  ingress: 
  - from:
    - podSelector:
        matchLabels:
           app: backend
    ports:
    - protocol: TCP
      port: 6379

---===---
19. Network Policy/demo-np-frontend.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-frontend
  namespace: myapp
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels: 
          app: backend
    ports:
    - protocol: TCP
      port: 80
---===---
19. Network Policy/demp-backend.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
---===---
19. Network Policy/np-example-1.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-db
  namespace: default
spec:
  podSelector: 
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend---===---
19. Network Policy/np-example-2.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
  - from:                  # first condition
    - podSelector:
        matchLabels:
          app: backend
    ports:                 # second condition 
    - protocol: TCP
      port: 3306---===---
19. Network Policy/np-example-3.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
  - from:                  # first rule
    - podSelector:
        matchLabels:
          app: backend
    ports:                 
    - protocol: TCP
      port: 3306
  - from:                  # second rule
    - podSelector:
        matchLabels:
          app: phpmyadmin
    ports:                 
    - protocol: TCP
      port: 3306---===---
19. Network Policy/np-example-4.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
  - to:                  # first rule
    - podSelector:
        matchLabels:
          app: mysql
    ports:                 
    - protocol: TCP
      port: 3306
  - to:                  # second rule
    - podSelector:
        matchLabels:
          app: redis
    ports:                 
    - protocol: TCP
      port: 6379---===---
19. Network Policy/np-example-5.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: myapp # namespace for pod that gets the policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
  - to:                  # first rule
    - podSelector:
        matchLabels:
          app: mysql
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: database
    ports:                 
    - protocol: TCP
      port: 3306
  - to:                  # second rule
    - podSelector:
        matchLabels:
          app: redis
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: database
    ports:                 
    - protocol: TCP
      port: 6379---===---
19. Network Policy/useful-links.md

##### network policy
* Network Policies: https://kubernetes.io/docs/concepts/services-networking/network-policies/
---===---
20. CKA Exam Tips/commands.md

### Create shortcuts
    alias k=kubectl
    export do="--dry-run=client -o yaml"

### Imperative commands

##### create pod
    kubectl run mypod --image=nginx

##### create deployment 
    kubectl create deployment nginx-deployment --image=nginx --replicas=2

##### create service for your pod
    kubectl expose deployment nginx-deployment --type=NodePort --name=nginx-service

##### generate service config
    kubectl create service clusterip myservice --tcp=80:80 --dry-run=client -o yaml > myservice.yaml

##### with shortcut for dry output
    kubectl create service clusterip myservice --tcp=80:80 $do > myservice.yaml 
---===---
20. CKA Exam Tips/useful-links.md

##### exam tips
* Official Frequently Asked Questions: https://docs.linuxfoundation.org/tc-docs/certification/faq-cka-ckad-cks
* Official Tips: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad
* More Tips & Tricks: https://hackernoon.com/tips-and-tricks-to-pass-certified-kubernetes-application-cka-exam-px13349o
* Kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet
