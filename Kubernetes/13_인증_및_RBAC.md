```sh
vagrant@kube-control1:~$ kubectl get sa
NAME                     SECRETS   AGE
default                  0         22d
nfs-client-provisioner   0         9d

vagrant@kube-control1:~$ ls volume/nfs-subdir-external-provisioner/deploy/
class.yaml  deployment.yaml  kustomization.yaml  objects  rbac.yaml  test-claim.yaml  test-pod.yaml

vagrant@kube-control1:~$ \vim volume/nfs-subdir-external-provisioner/deploy/rbac.yaml

vagrant@kube-control1:~$ kubectl get po
NAME                                      READY   STATUS              RESTARTS   AGE
nfs-client-provisioner-6f9c66997d-9prlq   0/1     ContainerCreating   0          23h

vagrant@kube-control1:~$ kubectl describe pod nfs-client-provisioner-6f9c66997d-9prlq 

vagrant@kube-control1:~$ kubectl create -f controller/myapp-rs.yaml

vagrant@kube-control1:~$ kubectl get pod
NAME                                      READY   STATUS              RESTARTS   AGE
myapp-rs-jp8s4                            1/1     Running             0          4s
myapp-rs-l2v8j                            1/1     Running             0          4s
myapp-rs-snxqp                            1/1     Running             0          4s
nfs-client-provisioner-6f9c66997d-9prlq   0/1     ContainerCreating   0          23h

vagrant@kube-control1:~$ kubectl describe pod myapp-rs-jp8s4 | grep -i account
Service Account:  default
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-twpq9 (ro)

vagrant@kube-control1:~$ kubectl describe pod | grep Account
Service Account:  default
Service Account:  default
Service Account:  default
Service Account:  nfs-client-provisioner

vagrant@kube-control1:~$ kubectl explain pod.spec | grep Account
  automountServiceAccountToken  <boolean>
    AutomountServiceAccountToken indicates whether a service account token
  serviceAccount        <string>
    DeprecatedServiceAccount is a deprecated alias for ServiceAccountName.
    Deprecated: Use serviceAccountName instead.
  serviceAccountName    <string>
    ServiceAccountName is the name of the ServiceAccount to use to run this pod.

vagrant@kube-control1:~$ kubectl exec myapp-rs-jp8s4 -- ls /var/run/secrets/kubernetes.io/serviceaccount

vagrant@kube-control1:~$ kubectl create sa test-user

vagrant@kube-control1:~$ kubectl get sa
NAME                     SECRETS   AGE
default                  0         22d
nfs-client-provisioner   0         9d
test-user                0         9s
```

```sh
vagrant@kube-control1:~/rbac$ vim sa-user1.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-user1

vagrant@kube-control1:~/rbac$ kubectl create -f sa-user1.yaml

vagrant@kube-control1:~/rbac$ kubectl get sa
NAME                     SECRETS   AGE
default                  0         22d
nfs-client-provisioner   0         9d
sa-user1                 0         2s
test-user                0         2m25s

vagrant@kube-control1:~/rbac$ vim role-read-pod.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pod
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]

vagrant@kube-control1:~/rbac$ kubectl create -f role-read-pod.yaml

vagrant@kube-control1:~/rbac$ kubectl get role read-pod 
NAME       CREATED AT
read-pod   2026-01-15T06:19:35Z

vagrant@kube-control1:~/rbac$ kubectl describe role read-pod 
Name:         read-pod
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get list]
```

```sh
vagrant@kube-control1:~/rbac$ vim rbing-sa-user1.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-user1-read-pod
  namespace: default
subjects:
  - kind: ServiceAccount
    name: sa-user1
    apiGroup: ""
    namespace: default
roleRef:
  kind: Role
  name: read-pod
  apiGroup: rbac.authorization.k8s.io

vagrant@kube-control1:~/rbac$ kubectl create -f rbing-sa-user1.yaml

vagrant@kube-control1:~/rbac$ kubectl get rolebindings.rbac.authorization.k8s.io 
NAME                                    ROLE                                         AGE
leader-locking-nfs-client-provisioner   Role/leader-locking-nfs-client-provisioner   9d
sa-user1-read-pod                       Role/read-pod                                24s

vagrant@kube-control1:~/rbac$ kubectl describe rolebindings.rbac.authorization.k8s.io sa-user1-read-pod 
Name:         sa-user1-read-pod
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  read-pod
Subjects:
  Kind            Name      Namespace
  ----            ----      ---------
  ServiceAccount  sa-user1  default


vagrant@kube-control1:~/rbac$ vim crole-read-pod.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-pod
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]

vagrant@kube-control1:~/rbac$ vim crbind-sa-user2.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sa-user2-read-pod
subjects:
  - kind: ServiceAccount
    name: sa-user2
    apiGroup: ""
    namespace: default
roleRef:
  kind: ClusterRole
  name: read-pod
  apiGroup: rbac.authorization.k8s.io

vagrant@kube-control1:~/rbac$ kubectl get role
NAME                                    CREATED AT
leader-locking-nfs-client-provisioner   2026-01-06T05:38:25Z
read-pod                                2026-01-15T06:19:35Z

vagrant@kube-control1:~/rbac$ kubectl create -f crole-read-pod.yaml 
vagrant@kube-control1:~/rbac$ kubectl get clusterrole | grep read
read-pod                                                               2026-01-15T06:49:50Z


vagrant@kube-control1:~/rbac$ kubectl create -f crbind-sa-user2.yaml 

vagrant@kube-control1:~/rbac$ kubectl get clusterrolebindings.rbac.authorization.k8s.io | grep sa
sa-user2-read-pod                                               ClusterRole/read-pod                                                               5s

vagrant@kube-control1:~/rbac$ kubectl get sa
NAME                     SECRETS   AGE
default                  0         23d
nfs-client-provisioner   0         9d
sa-user1                 0         46m
test-user                0         49m

vagrant@kube-control1:~/rbac$ kubectl describe clusterrolebindings.rbac.authorization.k8s.io sa-user2-read-pod 
Name:         sa-user2-read-pod
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  read-pod
Subjects:
  Kind            Name      Namespace
  ----            ----      ---------
  ServiceAccount  sa-user2  default

vagrant@kube-control1:~/rbac$ kubectl create sa sa-user2
```

```sh
vagrant@kube-control1:~/rbac$ kubectl auth whoami 
ATTRIBUTE   VALUE
Username    kubernetes-admin
Groups      [kubeadm:cluster-admins system:authenticated]

vagrant@kube-control1:~/rbac$ sudo vim /etc/kubernetes/admin.conf

vagrant@kube-control1:~/rbac$ ls ~/.kube/config 


vagrant@kube-control1:~/rbac$ kubectl get role -n metallb-system 
NAME         CREATED AT
controller   2025-12-23T06:30:41Z
pod-lister   2025-12-23T06:30:41Z

vagrant@kube-control1:~/rbac$ kubectl get sa -n metallb-system 
NAME         SECRETS   AGE
controller   0         23d
default      0         23d
speaker      0         23d

vagrant@kube-control1:~/rbac$ kubectl get po -n metallb-system 
NAME                          READY   STATUS    RESTARTS       AGE
controller-7f649565d4-46582   1/1     Running   1 (6h1m ago)   29h
speaker-gcfl4                 1/1     Running   1 (6h3m ago)   3d6h
speaker-lvvrq                 1/1     Running   1 (6h2m ago)   29h
speaker-mjxtr                 1/1     Running   2 (6h1m ago)   29h
speaker-wtpxr                 1/1     Running   3 (6h ago)     3d6h

vagrant@kube-control1:~/rbac$ kubectl describe pod -n metallb-system controller-7f649565d4-46582 | grep Account
Service Account:      controller
```

```sh
vagrant@kube-control1:~/rbac$ vim kubectl-pods.yaml

apiVersion: v1
kind: Pod
metadata:
  name: kubectl-sa-user1
spec:
  containers:
    - name: kubectl
      image: bitnami/kubectl
      command: ['sleep']
      args: ['infinity']

vagrant@kube-control1:~/rbac$ kubectl create -f kubectl-pods.yaml

vagrant@kube-control1:~/rbac$ kubectl get pod
NAME                                      READY   STATUS              RESTARTS   AGE
kubectl-sa-user1                          1/1     Running             0          22s
myapp-rs-jp8s4                            1/1     Running             0          77m
myapp-rs-l2v8j                            1/1     Running             0          77m
myapp-rs-snxqp                            1/1     Running             0          77m
nfs-client-provisioner-6f9c66997d-9prlq   0/1     ContainerCreating   0          24h

vagrant@kube-control1:~/rbac$ kubectl describe pod kubectl-sa-user1 | grep Accou
Service Account:  default

vagrant@kube-control1:~/rbac$ 
kubectl exec kubectl-sa-user1 -- kubectl auth whoami
vagrant@kube-control1:~/rbac$ kubectl exec kubectl-sa-user1 -- kubectl auth can-i get pod
vagrant@kube-control1:~/rbac$ kubectl exec kubectl-sa-user1 -- kubectl get pod
```

```sh
vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test set-cluster production --server https://1.2.3.4
Cluster "production" set.

vagrant@kube-control1:~/rbac$ ls -l config-test
-rw------- 1 vagrant vagrant 159 Jan 16 01:52 config-test

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test set-cluster development --server https://5.6.7.8
Cluster "development" set.

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test get-clusters 
NAME
development
production

vagrant@kube-control1:~/rbac$ kubectl config get-clusters
NAME
cluster.local

vagrant@kube-control1:~/rbac$ grep cluster.local ~/.kube/config
  name: cluster.local
    cluster: cluster.local
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local

vagrant@kube-control1:~/rbac$ cat config-test 
apiVersion: v1
clusters:
- cluster:
    server: https://5.6.7.8
  name: development
- cluster:
    server: https://1.2.3.4
  name: production
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null

vagrant@kube-control1:~/rbac$ kubectl config view --kubeconfig config-test
apiVersion: v1
clusters:
- cluster:
    server: https://5.6.7.8
  name: development
- cluster:
    server: https://1.2.3.4
  name: production
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test set-cluster production --server https://1.2.3.5
Cluster "production" set.

vagrant@kube-control1:~/rbac$ kubectl config view --kubeconfig config-test
apiVersion: v1
clusters:
- cluster:
    server: https://5.6.7.8
  name: development
- cluster:
    server: https://1.2.3.5
  name: production
...

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test set-credentials user --token abc
User "user" set.

vagrant@kube-control1:~/rbac$ cat config-test
...
users:
- name: user
  user:
    token: abc


vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test set-context prod-admin --cluster production --user admin --namespace default
Context "prod-admin" created.

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test get-clusters 
NAME
development
production

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test get-users 
NAME
user

vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test get-contexts 
CURRENT   NAME         CLUSTER      AUTHINFO   NAMESPACE
          prod-admin   production   admin      default


vagrant@kube-control1:~/rbac$ kubectl config --kubeconfig config-test view 

apiVersion: v1
clusters:
- cluster:
    server: https://5.6.7.8
  name: development
- cluster:
    server: https://1.2.3.5
  name: production
contexts:
- context:
    cluster: production
    namespace: default
    user: admin
  name: prod-admin
current-context: ""
kind: Config
preferences: {}
users:
- name: user
  user:
    token: REDACTED
```
---
우리가 kubectl 명령어를 실행할 때,
  1. 지정된 명령 파일 확인 (기본은 ~/.kube/config)
    -> --kubeconfig 옵션 혹은 환경변수 설정을 해두면 다른 파일 사용 가능
  2. 파일 안에서 현재 활성화된 컨텍스트를 확인
    -> kubernetese-admin@cluster.local
  3. 그 활성화된 컨텍스트에 설정되어 있는 사용자/클러스터 확인
    -> 사용자: kubernetes-admin
    클러스터: cluster.local
  4. 지정된 사용자와 클러스터의 정보를 확인
    -> 사용자에 대한 인증서/토큰 확인
    클러스터에 대한 인증서 + 서버 주소 확인
  5. kube-apiserver (클러스터의 서버 주소)로 요청 및 인증
  6. 인증 후에는 해당 사용자의 권한을 확인 (role / clusterRole)
  7. 해당 역할에 설정되어 있는 권한 내용을 확인 (대상 / 동작)
  8. 명령어 실행 가능 여부가 결정
---

```sh
vagrant@kube-control1:~/rbac$ openssl genrsa -out myuser.key 2048
vagrant@kube-control1:~/rbac$ ls -l myuser.key 
-rw------- 1 vagrant vagrant 1700 Jan 16 02:21 myuser.key

vagrant@kube-control1:~/rbac$ cat myuser.key
***REMOVED_PRIVATE_KEY***

vagrant@kube-control1:~/rbac$ openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser"
vagrant@kube-control1:~/rbac$ ls -l myuser.csr 
-rw-rw-r-- 1 vagrant vagrant 887 Jan 16 02:22 myuser.csr

vagrant@kube-control1:~/rbac$ base64 myuser.csr -w 0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0diWGwxYzJWeU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXh0eFFEUkRBYlV3enh1RjJSY2h2N0ZyK0dydFhHN0huM0lZb2NjUEk3TzdsCjFTdW9TSHpxZHBqcVN1RTFkWEEwaEJybDBYNEN0TGVXTEZoRHdpOCtqWjNscEI1SERlWElXQ3kyenpFSUpaNFIKMGZleEVRRy9BTHBlMk5JRExwV0tIVlNpckFucVRYWDRndDZ0TzNqbm53ZDFlWjdhNkt4RDNjLzl5VHliNS9OSwpJblhLQVdYS3hTNkFtNkJoZ3YyYWFEdnJhQko3V1haV3MrOFNMY3NUYlp0THpuQWhjcVkzM3lVUnBWR0tFcFNaCmErT2dsWnZYUUdDVkJPQ3AzK0NUcWcvNE1qK3Bla2pvVk1EQ2IzSXJERXRhWXA2anZvZFlWTlhQZGJQLzhIbzEKSkFiRzFHUXZUUXhFYWZZUGhBTjVTb1Z1R3hxN2NJUUExR0xwZGVNcFZRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBQ1ZRbG13blpRb29lZXY5bEtmRElXVmpkM1k4NjdKejljZ1JBbHFvZXNpelhGWnUza2lvCng3ODdyTEtqc3hLVFkwQUZ5Wlh0Sys5UFlpNitlQ0JnUTJWYTRmYnVTSGg5c0hQak15NTRnVDkvdkpOMjR6WXgKSjRrRmJraERSY3NuZWNNa3JLWVRwR3BhWXNZVnlIRmdmQXZwZEVFZFlxZTlkczFCa0J5ZUJUSkdIdm82SWVLSwp6dG16MWkwS1FwQldKY2JCUlVIL1poazJ2MmE1UGNWVmpTVG9HWWlXYUprNVNkVEQxeHFBczVkL0YvY2tJejhUCnc3QXl1ekcrZXc5KzJraEhOQXo4cEt6NDk5VWRKcnVZRHBWZGxMV0lST0dLd1dpclE2RE1aVVpzU25IV3hTcGwKR2VHZ2ZpSGc1czNqLzBhTFNMaDRrclVoTkZwSzFFYmtjL2M9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=

vagrant@kube-control1:~/rbac$ base64 myuser.csr -w 0 > myuser-csr.yaml
vagrant@kube-control1:~/rbac$ vim myuser-csr.yaml

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0diWGwxYzJWeU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXh0eFFEUkRBYlV3enh1RjJSY2h2N0ZyK0dydFhHN0huM0lZb2NjUEk3TzdsCjFTdW9TSHpxZHBqcVN1RTFkWEEwaEJybDBYNEN0TGVXTEZoRHdpOCtqWjNscEI1SERlWElXQ3kyenpFSUpaNFIKMGZleEVRRy9BTHBlMk5JRExwV0tIVlNpckFucVRYWDRndDZ0TzNqbm53ZDFlWjdhNkt4RDNjLzl5VHliNS9OSwpJblhLQVdYS3hTNkFtNkJoZ3YyYWFEdnJhQko3V1haV3MrOFNMY3NUYlp0THpuQWhjcVkzM3lVUnBWR0tFcFNaCmErT2dsWnZYUUdDVkJPQ3AzK0NUcWcvNE1qK3Bla2pvVk1EQ2IzSXJERXRhWXA2anZvZFlWTlhQZGJQLzhIbzEKSkFiRzFHUXZUUXhFYWZZUGhBTjVTb1Z1R3hxN2NJUUExR0xwZGVNcFZRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBQ1ZRbG13blpRb29lZXY5bEtmRElXVmpkM1k4NjdKejljZ1JBbHFvZXNpelhGWnUza2lvCng3ODdyTEtqc3hLVFkwQUZ5Wlh0Sys5UFlpNitlQ0JnUTJWYTRmYnVTSGg5c0hQak15NTRnVDkvdkpOMjR6WXgKSjRrRmJraERSY3NuZWNNa3JLWVRwR3BhWXNZVnlIRmdmQXZwZEVFZFlxZTlkczFCa0J5ZUJUSkdIdm82SWVLSwp6dG16MWkwS1FwQldKY2JCUlVIL1poazJ2MmE1UGNWVmpTVG9HWWlXYUprNVNkVEQxeHFBczVkL0YvY2tJejhUCnc3QXl1ekcrZXc5KzJraEhOQXo4cEt6NDk5VWRKcnVZRHBWZGxMV0lST0dLd1dpclE2RE1aVVpzU25IV3hTcGwKR2VHZ2ZpSGc1czNqLzBhTFNMaDRrclVoTkZwSzFFYmtjL2M9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth

vagrant@kube-control1:~/rbac$ kubectl create -f myuser-csr.yaml 

vagrant@kube-control1:~/rbac$ kubectl get certificatesigningrequests.certificates.k8s.io 
NAME     AGE     SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
myuser   2m15s   kubernetes.io/kube-apiserver-client   kubernetes-admin   <none>              Pending

vagrant@kube-control1:~/rbac$ kubectl certificate approve myuser
certificatesigningrequest.certificates.k8s.io/myuser approved
vagrant@kube-control1:~/rbac$ kubectl get certificatesigningrequests.certificates.k8s.io 
NAME     AGE     SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
myuser   3m24s   kubernetes.io/kube-apiserver-client   kubernetes-admin   <none>              Approved,Issued

vagrant@kube-control1:~/rbac$ kubectl get certificatesigningrequests.certificates.k8s.io myuser -o jsonpath='{.status.certificate}' |base64 -d > myuser.crt

openssl x509 -in myuser.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3b:77:88:74:15:7e:d5:1e:35:f3:78:08:e1:a1:8b:07
        Signature Algorithm: sha256WithRSAEncryption
...
```

```sh
vagrant@kube-control1:~/rbac$ vim role-admin-pod.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: admin-pod
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "create", "delete", "update", "watch"]


vagrant@kube-control1:~/rbac$ vim rbind-myuser.yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: myuser-admin-pod
  namespace: default
subjects:
  - kind: User
    name: myuser
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: admin-pod
  apiGroup: rbac.authorization.k8s.io

vagrant@kube-control1:~/rbac$  kubectl create -f role-admin-pod.yaml -f rbind-myuser.yaml

vagrant@kube-control1:~/rbac$ kubectl config set-credentials myuser --client-certificate myuser.crt --embed-certs 

vagrant@kube-control1:~/rbac$ kubectl config get-users 
NAME
kubernetes-admin
myuser

vagrant@kube-control1:~/rbac$ kubectl config current-context 
kubernetes-admin@cluster.local

vagrant@kube-control1:~/rbac$ kubectl config get-contexts 
CURRENT   NAME                             CLUSTER         AUTHINFO           NAMESPACE
*         kubernetes-admin@cluster.local   cluster.local   kubernetes-admin

vagrant@kube-control1:~/rbac$ kubectl config set-context myuser@cluster.local --user myuser --cluster cluster.local --namespace default

vagrant@kube-control1:~/rbac$ kubectl config get-contexts
CURRENT   NAME                             CLUSTER         AUTHINFO           NAMESPACE
          kubernetes-admin@cluster.local   cluster.local   kubernetes-admin
*         myuser@cluster.local             cluster.local   myuser             default
```

```sh


```