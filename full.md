---===---
01. K8s Core Concepts/useful-links.md

##### K8s Components
* K8s official documentation: https://kubernetes.io/docs/concepts/
* Enrypting Secret Data at Rest: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

##### K8s Architecture
* K8s Architecture Components: https://kubernetes.io/docs/concepts/overview/components/

##### kubectl and K8s configuration 
* Kubectl - K8s CLI: https://kubernetes.io/docs/reference/kubectl/overview/
* Managing Objects Imperative (Kubectl): https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/
* Managing Objects Declarative (Config File): https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/
* Imperative vs Declarative: https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/

---===---
02. Install Cluster/commands.md

### Provision Infrastructure 

##### move private key to .ssh folder and restrict access
    mv ~/Downloads/k8s-node.pem ~/.ssh
    chmod 400 ~/.ssh/k8s-node.pem

##### ssh into ec2 instance with its public ip
    ssh -i ~/.ssh/k8s-node.pem ubuntu@35.180.130.108


### Configure Infrastructure
    sudo swapoff -a

##### set host names of nodes
    sudo vim /etc/hosts

##### get priavate ips of each node and add this to each server 
    45.14.48.178 master
    45.14.48.179 worker1
    45.14.48.180 worker2

##### we can now use these names instead of typing the IPs, when nodes talk to each other. After that, assign a hostname to each of these servers.

##### on master server
    sudo hostnamectl set-hostname master 

##### on worker1 server
    sudo hostnamectl set-hostname worker1 

##### on worker2 server
    sudo hostnamectl set-hostname worker2


### Initialize K8s cluster
    sudo kubeadm init

### Check kubelet process running 
    service kubelet status
    systemctl status kubelet

### Check extended logs of kubelet service
    journalctl -u kubelet

### Access cluster as admin
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

### Kubectl commands

##### get node information
    kubectl get node

##### get pods in kube-system namespace
    kubectl get pod -n kube-system

##### get pods from all namespaces
    kubectl get pod -A

##### get wide output
    kubectl get pod -n kube-system -o wide


### Install pod network plugin

##### download and install the manifest
    kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

[Link to the Weave-net installation guide](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation)    

##### check weave net status
    kubectl exec -n kube-system weave-net-1jkl6 -c weave -- /home/weave/weave --local status

### Join worker nodes

##### create and execute script
    vim install-containerd.sh
    chmod u+x install-containerd.sh
    ./install-containerd.sh

##### on master
    kubeadm token create --help
    kubeadm token create --print-join-command

##### copy the output command and execute on worker node as ROOT
    sudo kubeadm join 172.31.43.99:6443 --token 9bds1l.3g9ypte9gf69b5ft --discovery-token-ca-cert-hash sha256:xxxx

##### start a test pod
    kubectl run test --image=nginx


---===---
02. Install Cluster/install-containerd.sh

#!/bin/bash

# Install and configure prerequisites
## load the necessary modules for Containerd
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

# Install containerd
sudo apt-get update
sudo apt-get -y install containerd

# Configure containerd with defaults and restart with this config
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
---===---
02. Install Cluster/install-k8s-components.sh

# Install packages needed to use the Kubernetes apt repository
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Download the Google Cloud public signing key
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add the Kubernetes apt repository
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubelet, kubeadm & kubectl, and pin their versions
sudo apt-get update
# check available kubeadm versions (when manually executing)
apt-cache madison kubeadm
# Install version 1.21.0 for all components
sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
sudo apt-mark hold kubelet kubeadm kubectl
## apt-mark hold prevents package from being automatically upgraded or removed---===---
02. Install Cluster/useful-links.md

##### provision infrastructure on aws 
* Troubleshooting SSH Connection on AWS: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html

##### k8s installation
* System Requirements: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
* Static Pods: https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
* Certificates: https://kubernetes.io/docs/setup/best-practices/certificates/
* Kubeadm: https://kubernetes.io/docs/reference/setup-tools/kubeadm/

##### configure Infrastructure
* Pre-Requisites: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin
* AWS Security Group Docs: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html

##### container runtime 
* Installing container runtime: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime

##### kubeadm 
* Kubeadm init command details: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
* kubeadm init workflow: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#init-workflow
* Addons: https://kubernetes.io/docs/concepts/overview/components/#addons

##### kubeconfig
* Kubeconfig File: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

##### kube-system namespace
* Namespaces Official Docs: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

##### networking in K8s - CNI 
* CoreDNS: https://coredns.io/plugins/kubernetes/
* CNI Specification: https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration
* Cluster Networking - Official Docs: https://kubernetes.io/docs/concepts/cluster-administration/networking/
* Weave Net: https://www.weave.works/docs/net/latest/kubernetes/kube-addon/

##### join worker nodes
* Kubeadm Join Command: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/
* Troubleshooting Weave Connection: https://www.weave.works/docs/net/latest/tasks/ipam/troubleshooting-ipam/



---===---
03. Deploy Application/commands.md

### Kubectl commands

##### apply manifests
    kubectl apply -f nginx-deployment.yaml
    kubectl apply -f nginx-service.yaml

##### labels
    kubectl get svc
    kubectl describe svc {svc-name}
    kubectl get ep

    kubectl get svc --show-labels
    kubectl get svc -l app=nginx 

    kubectl get pod --show-labels
    kubectl get pod -l app=nginx
    kubectl logs -l app=nginx

    kubectl get pod -n kube-system --show-labels
    kubectl logs -n kube-system -l="name=weave-net" -c weave

    kubectl get node —show-labels


##### scaling deployments
    kubectl scale --help
    kubectl scale deployment {depl-name} --replicas 4
    kubectl scale deployment {depl-name} --replicas 3

    kubectl scale deployment {depl-name} --replicas 5 --record
    kubectl rollout history deployment {depl-name}


##### create pods
    kubectl run test-nginx-service --image=busybox
    kubectl exec -it {pod-name} -- bash
---===---
03. Deploy Application/nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20




---===---
03. Deploy Application/nginx-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
    svc: test-nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
---===---
03. Deploy Application/useful-links.md

##### labels
* Recommended Labels: https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/

##### DNS in K8s
* DNS for Services and Pods: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
* Debugging DNS Resolution: https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
* CoreDNS in K8s: https://kubernetes.io/docs/tasks/administer-cluster/coredns/
* CoreDNS: https://coredns.io/

##### service IP address
* Kube API Server: https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
* Kubeadm Init Defaults: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-config/

##### working with kubectl
* Kubectl Cheatsheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/ 



---===---
04. External Access/ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: your-load-balancer-domain-name
    http:
      paths:
      - backend:
          service:
            name: nginx-service
            port: 
              number: 8080
        path: /my-app
        pathType: Exact
---===---
04. External Access/load-balancer-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
      nodePort: 30000
---===---
04. External Access/node-port-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
      nodePort: 30000
---===---
04. External Access/README.md

#### If you get an error on creating ingress component related to "nginx-controller-admission" webhook, than manually delete the ValidationWebhook and try again. To delete the ValidationWebhook:
    kubectl get ValidatingWebhookConfiguration 
    kubectl delete ValidatingWebhookConfiguration {name}

[Link to a more detailed description of the issue](https://pet2cattle.com/2021/02/service-ingress-nginx-controller-admission-not-found)
---===---
04. External Access/useful-links.md

##### service types
* NodePort Service: https://kubernetes.io/docs/concepts/services-networking/service/#nodeport
* Load Balancer Service: https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer

##### ingress
* Ingress: https://kubernetes.io/docs/concepts/services-networking/ingress/
* List of Ingress Controller: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
* Bare Metal Ingress Controller: https://kubernetes.github.io/ingress-nginx/deploy/baremetal/

##### helm
* Install Helm: https://helm.sh/docs/intro/install/  

##### install nginx-ingress-controller with helm
* Nginx Ingress Controller: https://kubernetes.github.io/ingress-nginx/
---===---
05. Users and Permissions/cicd-binding.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cicd-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cicd-role
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: default
---===---
05. Users and Permissions/cicd-role.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cicd-role
rules:
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - create
  - update
  - list
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
  - update
  - list
---===---
05. Users and Permissions/commands.md

### Create client certificate

##### create 2048-bit RSA key
    openssl genrsa -out dev-tom.key 2048

##### create user Certificate Signing Request
    openssl req -new -key dev-tom.key -subj "/CN=tom" -out dev-tom.csr 

##### get the base64 encoded value of the csr file
    cat dev-tom.csr | base64 | tr -d "\n"

##### create CSR in k8s
    kubectl apply -f dev-tom-csr.yaml

##### review CSR
    kubectl get csr

##### approve CSR
    kubectl certificate approve dev-tom

##### get dev-tom's signed certificate
    kubectl get csr dev-tom -o yaml

##### save decoded cert to dev-tom.crt file

##### connect to cluster as dev-tom user
    kubectl --server={api-server-address} \
    --certificate-authority=/etc/kubernetes/pki/ca.crt \
    --client-certificate=dev-tom.crt \
    --client-key=dev-tom.key \
    get pods

##### create cluster role & binding
    kubectl create clusterrole dev-cr --dry-run=client -o yaml > dev-cr.yaml
    kubectl create clusterrolebinding dev-crb --dry-run=client -o yaml > dev-crb.yaml

##### check user permissions as dev-tom
    kubectl auth can-i get pod

##### check user permissions as admin
    kubectl auth can-i get pod —-as {user-name}


### Create Service Account with Permissions
    kubectl create serviceaccount jenkins-sa --dry-run=client -o yaml > jenkins-sa.yaml

    kubectl create role cicd-role

    kubectl create clusterrolebinding cicd-binding \
    --clusterrole=cicd-role \
    --serviceaccount=default:jenkins

### Access with service account token

    kubectl options

    kubectl --server $server \
    --certificate-authority /etc/kubernetes/pki/ca.crt \
    --token $token \
    --user jenkins \
    get pods
---===---
05. Users and Permissions/dev-crb.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dev-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: dev-cr
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: tom
---===---
05. Users and Permissions/dev-cr.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: dev-cr
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs: ["*"]
- apiGroups:
  - apps
  resources:
  - deployments
  - statefulSets
  verbs:
  - get
  - list
  - create
---===---
05. Users and Permissions/dev-tom-csr.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev-tom
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  # expirationSeconds: 8640000
  usages:
  - client auth---===---
05. Users and Permissions/jenkins-sa.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
---===---
05. Users and Permissions/useful-links.md

##### RBAC
* Authentication: https://kubernetes.io/docs/reference/access-authn-authz/authentication/
* Authorization: https://kubernetes.io/docs/reference/access-authn-authz/authorization/
* RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

##### certificates API
* Manage TLS Certificates: https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
* Certificate Signing Request: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

##### API groups
* Resource Types and corresponding apiGroup: https://kubernetes.io/docs/reference/kubectl/overview/#resource-types
* API Groups: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#-strong-api-groups-strong

##### service account
* Configure Service Account: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
* Managing Service Accounts: https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/




---===---
06. Debugging & Troubleshooting/busybox-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox-container
    image: busybox
    args: ["echo", "hello"]
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 5; done"]
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "sleep 100"]---===---
06. Debugging & Troubleshooting/commands.md

### Debug pod

##### start busybox in interactive mode
    kubectl run debug-pod --image=busybox -it 

##### check service name can be resolved
    nslookup nginx-service.default.svc.cluster.local
    nslookup nginx-service

##### access service ip returned by nslookup
    ping service-ip 

### Execute commands in pod from master node

##### ping service
    kubectl exec -it pod-name -- sh -c "ping nginx-service"

##### print all envs
    kubectl exec -it pod-name -- sh -c "printenv"

##### print all running ports
    kubectl exec -it pod-name -- sh -c "netstat -lntp"


### Jsonpath output format
    kubectl get node -o json
    kubectl get pod -o json 

##### for single pod
    kubectl get pod -o jsonpath='{.items[0].metadata.name}'

##### print for all pods
    kubectl get pod -o jsonpath='{.items[*].metadata.name}'

##### multiple attributes
    kubectl get pod -o jsonpath="{.items[*]['metadata.name', 'status.podIP']}"
    kubectl get pod -o jsonpath="{.items[*]['metadata.name', 'status.podIP', 'status.startTime']}"

##### print multiple attributes on new lines
    kubectl get pod -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'
    kubectl get pod -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\t"}{.status.startTime}{"\n"}{end}'

### Custom columns output
    kubectl get pods -o custom-columns=POD_NAME:.metadata.name,POD_IP:.status.podIP,CREATED_AT:.status.startTime


### Debugging kubelet
    service kubelet status
    sudo vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
    sudo systemctl daemon-reload
    sudo systemctl restart kubelet
    service kubelet status
---===---
06. Debugging & Troubleshooting/useful-links.md

##### troubleshooting in k8s
* Troubleshoot Applications: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/
* Troubleshoot Clusters: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/

##### debuging pods - commands & args
* BusyBox Image: https://hub.docker.com/_/busybox
* Debug running Pods: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/
* Get Shell of running container: https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/
* Define Command & Arguments for Container: https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

##### kubectl output formats
* Output Options: https://kubernetes.io/docs/reference/kubectl/overview/#output-options
* Kubectl jsonpath: https://kubernetes.io/docs/reference/kubectl/jsonpath/
---===---
07. Multi-container Pods/expose-pod-info.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx:1.20
  - name: logging-sidecar
    image: busybox:1.28
    command: [ "sh", "-c"]
    args:
    - while true; do
        echo sync logs;
        echo -en '\n';
        printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
        printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
        sleep 20;
      done;
    env:
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: MY_POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: MY_POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: MY_POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: MY_POD_SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          fieldPath: spec.serviceAccountName---===---
07. Multi-container Pods/multi-container-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx:1.20
  - name: logging-sidecar
    image: busybox:1.28
    command: ['sh', '-c', "while true; do echo sync logs; sleep 20; done"]
  initContainers:
  - name: myservice-available
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb-service; do echo waiting for myservice; sleep 4; done"]

---
# alternative command syntax
- name: logging-sidecar
  image: busybox:1.28
  command:
  - 'sh'
  - '-c' 
  - "while true; do echo sync logs; sleep 20; done"

# alternative with args
- name: logging-sidecar
  image: busybox:1.28
  command: [ "sh", "-c"]
  args:
  - while true; do
      echo sync logs;
      sleep 20;
    done;---===---
07. Multi-container Pods/useful-links.md

##### multi-container pods
* Init Containers: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
* Sidecar Container: https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers

##### exposing pod data to containers
* Exposing Pod Information: https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/
---===---
08. Data Persistence/deployment-with-emptydir.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: busybox:1.28
        command: ['sh', '-c'] 
        args: 
        - while true; do 
            echo "$(date) INFO some app data" >> /var/log/myapp.log;
            sleep 5;
          done
  
        volumeMounts:
        - name: log
          mountPath: /var/log  
 
      - name: log-sidecar
        image: busybox:1.28
        command: ['sh', '-c'] 
        args:
        - tail -f /var/log/myapp.log 

        volumeMounts:
        - name: log
          mountPath: /var/log
  
      volumes: 
      - name: log
        emptyDir: {}
---===---
08. Data Persistence/deployment-with-pvc.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-db
  labels:
    app: my-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
       
        volumeMounts:
        - name: db-data
          mountPath: "/var/lib/mysql"
          

      volumes:
      - name: db-data
        persistentVolumeClaim:
          claimName: mysql-data-pvc---===---
08. Data Persistence/pv-and-pvc.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi---===---
08. Data Persistence/useful-links.md

##### volumes
* Storage Official Docs: https://kubernetes.io/docs/concepts/storage/
* Volume types: https://kubernetes.io/docs/concepts/storage/volumes/#volume-types
* Example k8s manifests https://gitlab.com/nanuchi/bootcamp-kubernetes/-/tree/master/kubernetes-volumes

##### hostpath
* Hostpath Volume Type: https://kubernetes.io/docs/concepts/storage/volumes/#hostpath
* How hostpath is different from local volume type: https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/#how-is-it-different-from-a-hostpath-volume
* Configure Pod to use PV: https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
* Access Modes: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes

##### emptydir
* emptyDir Volume Type: https://kubernetes.io/docs/concepts/storage/volumes/#emptydir

---===---
09. Secret & ConfigMap/config-as-env-vars.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  db_host: mysql-service
---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: busybox:1.28
        command: ['sh', '-c', "printenv MYSQL_USER MYSQL_PASSWORD MYSQL_SERVER"]
        env:
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: password
        - name: MYSQL_SERVER 
          valueFrom: 
            configMapKeyRef:
              name: myapp-config
              key: db_host---===---
09. Secret & ConfigMap/config-as-volumes.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  mysql.conf: |
    [mysqld]
    port=3306
    socket=/tmp/mysql.sock
    key_buffer_size=16M
    max_allowed_packet=128M
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  secret.file: |
    c29tZXN1cGVyc2VjcmV0IGZpbGUgY29udGVudHMgbm9ib2R5IHNob3VsZCBzZWU=

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-db
  labels:
    app: my-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
      - name: my-db
        image: busybox:1.28
        command: ['sh', '-c', "cat /mysql/db-config; cat /mysql/db-secret"]

        volumeMounts:
        - name: db-config
          mountPath: /mysql/db-config
        - name: db-secret
          mountPath: /mysql/db-secret
          readOnly: true

      volumes:
        - name: db-config
          configMap:
            name: mysql-config
        - name: db-secret
          secret:
            secretName: mysql-secret---===---
09. Secret & ConfigMap/useful-links.md

##### configmap
* Complete ConfigMap docs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

##### secret
* Complete Secret docs: https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/
---===---
10. Resource Requests & Limits/commands.md

##### print all pods with resource requests and limits 

    kubectl get pod -o jsonpath="{range .items[*]}{.metadata.name}{.spec.containers[*].resources}{'\n'}"
---===---
10. Resource Requests & Limits/my-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: nginx:1.20
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      - name: logging-sidecar
        image: busybox:1.28
        command: ['sh', '-c', "while true; do echo sync logs; sleep 20; done"]
        resources:
          requests:
            memory: "32Mi"
            cpu: "125m"
          limits:
            memory: "64Mi"
            cpu: "250m"---===---
10. Resource Requests & Limits/useful-links.md

##### resources
* Managing Resources in K8s: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
* Resource Unit in K8s: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes
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
