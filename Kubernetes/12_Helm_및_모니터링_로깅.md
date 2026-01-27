https://github.com/helm/helm/releases/tag/v4.0.4
https://artifacthub.io/

```sh
vagrant@kube-control1:~/scheduler$ helm repo add bitnami https://charts.bitnami.com/bitnami
vagrant@kube-control1:~/scheduler$ helm repo list
NAME    URL
bitnami https://charts.bitnami.com/bitnami

vagrant@kube-control1:~/scheduler$ helm repo update 
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈

vagrant@kube-control1:~/scheduler$ helm install mydb bitnami/mysql 

vagrant@kube-control1:~/scheduler$ helm list 
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mydb    default         1               2026-01-14 05:55:05.741296257 +0000 UTC deployed        mysql-14.0.3    9.4.0

vagrant@kube-control1:~/scheduler$ helm history mydb 
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Wed Jan 14 05:55:05 2026        deployed        mysql-14.0.3    9.4.0           Install complete

vagrant@kube-control1:~/scheduler$ helm status mydb 
NAME: mydb
LAST DEPLOYED: Wed Jan 14 05:55:05 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mysql
CHART VERSION: 14.0.3
APP VERSION: 9.4.0
...

vagrant@kube-control1:~/scheduler$ kubectl get pod
NAME                             READY   STATUS    RESTARTS   AGE
myapp-ds-5n2lj                   1/1     Running   0          3h22m
myapp-ds-cxscp                   1/1     Running   0          3h22m
myapp-ds-snlv9                   1/1     Running   0          3h22m
myapp-rs-podaff-frontend-plmzn   1/1     Running   0          3h18m
myapp-rs-podaff-frontend-prbnv   1/1     Running   0          3h18m
myapp-rs-podaff-frontend-pwz7r   1/1     Running   0          3h8m
mydb-mysql-0                     0/1     Pending   0          5m40s

vagrant@kube-control1:~/scheduler$ kubectl get sts
NAME         READY   AGE
mydb-mysql   0/1     6m39s

vagrant@kube-control1:~/scheduler$ helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mydb    default         1               2026-01-14 05:55:05.741296257 +0000 UTC deployed        mysql-14.0.3    9.4.0

vagrant@kube-control1:~/scheduler$ kubectl describe sts mydb-mysql 
...
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  7m14s  statefulset-controller  create Claim data-mydb-mysql-0 Pod mydb-mysql-0 in StatefulSet mydb-mysql success
  Normal  SuccessfulCreate  7m14s  statefulset-controller  create Pod mydb-mysql-0 in StatefulSet mydb-mysql successful

vagrant@kube-control1:~/scheduler$ kubectl describe pod mydb-mysql-0 
...
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  7m56s  default-scheduler  0/4 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
  Warning  FailedScheduling  2m33s  default-scheduler  0/4 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.

vagrant@kube-control1:~/scheduler$ kubectl get pvc
NAME                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-mydb-mysql-0   Pending                                                     <unset>                 9m29s
deploy-pvc          Pending                                      nfs-client     <unset>                 5d23h

vagrant@kube-control1:~/scheduler$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      6d23h
mydb-mysql         1      11m

vagrant@kube-control1:~/scheduler$ kubectl get secrets 
NAME                         TYPE                 DATA   AGE
myapp-tls-secret             kubernetes.io/tls    2      6d23h
mydb-mysql                   Opaque               2      11m
sh.helm.release.v1.mydb.v1   helm.sh/release.v1   1      11m

vagrant@kube-control1:~/scheduler$ kubectl get svc
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes            ClusterIP   10.233.0.1      <none>        443/TCP    4h27m
mydb-mysql            ClusterIP   10.233.33.109   <none>        3306/TCP   11m
mydb-mysql-headless   ClusterIP   None            <none>        3306/TCP   11m

vagrant@kube-control1:~/scheduler$ kubectl get pod
NAME                             READY   STATUS    RESTARTS   AGE
myapp-ds-5n2lj                   1/1     Running   0          3h30m
myapp-ds-cxscp                   1/1     Running   0          3h30m
myapp-ds-snlv9                   1/1     Running   0          3h30m
myapp-rs-podaff-frontend-plmzn   1/1     Running   0          3h26m
myapp-rs-podaff-frontend-prbnv   1/1     Running   0          3h26m
myapp-rs-podaff-frontend-pwz7r   1/1     Running   0          3h15m
mydb-mysql-0                     0/1     Pending   0          13m

vagrant@kube-control1:~/scheduler$ kubectl get all -l app.kubernetes.io/managed-by=Helm
NAME               READY   STATUS    RESTARTS   AGE
pod/mydb-mysql-0   0/1     Pending   0          13m
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/mydb-mysql            ClusterIP   10.233.33.109   <none>        3306/TCP   13m
service/mydb-mysql-headless   ClusterIP   None            <none>        3306/TCP   13m
NAME                          READY   AGE
statefulset.apps/mydb-mysql   0/1     13m


vagrant@kube-control1:~/scheduler$ helm uninstall mydb
release "mydb" uninstalled

vagrant@kube-control1:~/scheduler$ kubectl get all -l app.kubernetes.io/managed-by=Helm
No resources found in default namespace.
```

```sh
vagrant@kube-control1:~/scheduler$ helm show values bitnami/mysql
...

vagrant@kube-control1:~/scheduler$ helm show values bitnami/mysql | grep -A4 ^auth:
auth:
  ## @param auth.rootPassword Password for the `root` user. Ignored if existing secret is provided
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mysql#setting-the-root-password-on-first-run
  ##
  rootPassword: ""
```

```sh
vagrant@kube-control1:~$ vim custom.yaml

auth:
  rootPassword: "P@ssw0rd"
global:
  storageClass: "nfs-client"

vagrant@kube-control1:~$ ls -ld /srv/nfs-volume/
drwxr-xr-x 2 root root 4096 Jan  6 02:25 /srv/nfs-volume/

vagrant@kube-control1:~$ sudo apt install nfs-kernel-server -y 

vagrant@kube-control1:~$ tail -n1 /etc/exports 
/srv/nfs-volume *(rw,sync,no_subtree_check,no_root_squash)

vagrant@kube-control1:~$ sudo exportfs -r

vagrant@kube-control1:~$ ssh vagrant@kube-node1 apt install -y nfs-common

vagrant@kube-control1:~$ cd volume/nfs-subdir-external-provisioner/deploy/
vagrant@kube-control1:~/volume/nfs-subdir-external-provisioner/deploy$ grep -A2 192 deployment.yaml 

vagrant@kube-control1:~/volume/nfs-subdir-external-provisioner/deploy$ kubectl create -f rbac.yaml -f class.yaml -f deployment.yaml

vagrant@kube-control1:~/volume/nfs-subdir-external-provisioner/deploy$ cd -
/home/vagrant
vagrant@kube-control1:~$ kubectl get sc
```

```sh
vagrant@kube-control1:~$ kubectl create ns mysql-ns

vagrant@kube-control1:~$ helm install -n mysql-ns mydb bitnami/mysql -f custom.yaml

vagrant@kube-control1:~$ kubectl get sts,pod -n mysql-ns 
vagrant@kube-control1:~$ kubectl describe pod -n mysql-ns 
vagrant@kube-control1:~$ kubectl get sts,pod -n mysql-ns 
vagrant@kube-control1:~$ kubectl get pvc -n mysql-ns 

vagrant@kube-control1:~$ kubectl get sc -A
vagrant@kube-control1:~$ kubectl get pvc -A

vagrant@kube-control1:~$ kubectl get pv
vagrant@kube-control1:~$ kubectl get sts,pod -n mysql-ns 
vagrant@kube-control1:~$ kubectl describe pod -n mysql-ns 
vagrant@kube-control1:~$ kubectl get sts,pod -n mysql-ns 
vagrant@kube-control1:~$ kubectl get secrets -n mysql-ns 
vagrant@kube-control1:~$ kubectl get secrets -n mysql-ns mydb-mysql -o yaml

vagrant@kube-control1:~$ echo UEBzc3cwcmQ= | base64 --decode 
P@ssw0rd

vagrant@kube-control1:~$ helm upgrade mydb bitnami/mysql -f custom.yaml --set primary.service.ports.mysql=13306 -n mysql-ns 

vagrant@kube-control1:~$ helm status -n mysql-ns mydb 
vagrant@kube-control1:~$ helm history -n mysql-ns mydb 

vagrant@kube-control1:~$ helm rollback -n mysql-ns mydb 1
Rollback was a success! Happy Helming!

vagrant@kube-control1:~$ helm list -n mysql-ns 
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
mydb    mysql-ns        3               2026-01-14 07:13:27.749776675 +0000 UTC deployed        mysql-14.0.3    9.4.0 

vagrant@kube-control1:~$ helm uninstall -n mysql-ns mydb 
release "mydb" uninstalled
```

```sh
vagrant@kube-control1:~$ kubectl get ns
NAME              STATUS   AGE
default           Active   22d
development       Active   21d
ingress-nginx     Active   22d
kube-node-lease   Active   22d
kube-public       Active   22d
kube-system       Active   22d
metallb-system    Active   22d
my-ns             Active   2d21h
mysql-ns          Active   19h
practice-ns       Active   5d19h

vagrant@kube-control1:~$ kubectl create ns monitor

vagrant@kube-control1:~$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

vagrant@kube-control1:~$ helm repo list
NAME                    URL
bitnami                 https://charts.bitnami.com/bitnami
prometheus-community    https://prometheus-community.github.io/helm-charts

vagrant@kube-control1:~$ helm  repo update
vagrant@kube-control1:~$ vim prometheus-grafana.yaml

grafana:
  service:
    type: LoadBalancer

vagrant@kube-control1:~$ helm install prom prometheus-community/kube-prometheus-stack -f prometheus-grafana.yaml -n monitor 

vagrant@kube-control1:~$ kubectl get po -n monitor 
NAME                                                     READY   STATUS              RESTARTS   AGE
alertmanager-prom-kube-prometheus-stack-alertmanager-0   0/2     Init:0/1            0          17s
prom-grafana-8cfb86876-k8lxj                             0/3     ContainerCreating   0          43s
prom-kube-prometheus-stack-operator-746c466954-vftb2     1/1     Running             0          43s
prom-kube-state-metrics-56fbcdd8bf-5x97m                 0/1     Running             0          43s
prom-prometheus-node-exporter-2bxd6                      1/1     Running             0          43s
prom-prometheus-node-exporter-2wxnn                      1/1     Running             0          43s
prom-prometheus-node-exporter-5s2l8                      1/1     Running             0          43s
prom-prometheus-node-exporter-hn5qq                      1/1     Running             0          43s
prometheus-prom-kube-prometheus-stack-prometheus-0       0/2     Init:0/1            0          17s

vagrant@kube-control1:~$ kubectl get ds -n monitor
NAME                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
prom-prometheus-node-exporter   4         4         4       4            4           kubernetes.io/os=linux   2m25s

vagrant@kube-control1:~$ kubectl get ds -n monitor 
NAME                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
prom-prometheus-node-exporter   4         4         4       4            4           kubernetes.io/os=linux   4m9s

vagrant@kube-control1:~$ kubectl get deploy -n monitor 
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
prom-grafana                          1/1     1            1           4m12s
prom-kube-prometheus-stack-operator   1/1     1            1           4m12s
prom-kube-state-metrics               1/1     1            1           4m12s

vagrant@kube-control1:~$ kubectl get sts -n monitor 
NAME                                                   READY   AGE
alertmanager-prom-kube-prometheus-stack-alertmanager   1/1     3m49s
prometheus-prom-kube-prometheus-stack-prometheus       1/1     3m49s

vagrant@kube-control1:~$ kubectl get svc -n monitor 
NAME                                      TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
alertmanager-operated                     ClusterIP      None            <none>           9093/TCP,9094/TCP,9094/UDP   4m31s
prom-grafana                              LoadBalancer   10.233.33.135   192.168.56.201   80:32519/TCP                 4m58s
prom-kube-prometheus-stack-alertmanager   ClusterIP      10.233.56.63    <none>           9093/TCP,8080/TCP            4m58s
prom-kube-prometheus-stack-operator       ClusterIP      10.233.53.246   <none>           443/TCP                      4m58s
prom-kube-prometheus-stack-prometheus     ClusterIP      10.233.42.62    <none>           9090/TCP,8080/TCP            4m58s
prom-kube-state-metrics                   ClusterIP      10.233.12.247   <none>           8080/TCP                     4m58s
prom-prometheus-node-exporter             ClusterIP      10.233.43.157   <none>           9100/TCP                     4m58s
prometheus-operated                       ClusterIP      None            <none>           9090/TCP                     4m31s

vagrant@kube-control1:~$ kubectl get cm -n monitor 
NAME                                                           DATA   AGE
kube-root-ca.crt                                               1      15m
prom-grafana                                                   1      9m10s
prom-grafana-config-dashboards                                 1      9m10s
...

vagrant@kube-control1:~$ kubectl get secrets -n monitor 
NAME                                                                                  TYPE                 DATA   AGE
alertmanager-prom-kube-prometheus-stack-alertmanager                                  Opaque               1      9m53s
alertmanager-prom-kube-prometheus-stack-alertmanager-cluster-tls-config               Opaque               1      9m26s
...

vagrant@kube-control1:~$ helm list -n monitor 
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
prom    monitor         1               2026-01-15 02:26:17.902784303 +0000 UTC deployed        kube-prometheus-stack-80.14.3   v0.87.1

# http://192.168.56.201/login 접속
# ID: admin / PW: ***REMOVED***
# PW는 kubectl get secrets -n monitor prom-grafana -o yaml 해서 나온 admin-password 값을 디코딩해서 사용

vagrant@kube-control1:~$ kubectl get secrets -n monitor prom-grafana -o yaml
apiVersion: v1
data:
  admin-password: ***REMOVED***
  admin-user: YWRtaW4=
  ldap-toml: ""
  ...

vagrant@kube-control1:~$ echo ***REMOVED*** | base64 -d
***REMOVED***

vagrant@kube-control1:~$ helm uninstall prom -n monitor
```

```sh
vagrant@kube-control1:~$ sudo ls /var/log
...

vagrant@kube-control1:~$ sudo ls /var/log/pods
kube-system_calico-node-4n9k9_b0c60607-4a73-4c3d-bd18-3885ddc809c9  
...

vagrant@kube-control1:~$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS        AGE
calico-kube-controllers-7899b68c68-lfsd9   1/1     Running   1 (3h32m ago)   26h
calico-node-4n9k9                          1/1     Running   5 (3h34m ago)   22d
calico-node-6n866                          1/1     Running   5 (3h32m ago)   22d
calico-node-gf2vk                          1/1     Running   5 (3h33m ago)   22d
calico-node-hrxbq                          1/1     Running   5 (3h32m ago)   22d
....
```

```sh
vagrant@kube-control1:~$ kubectl create ns logging
vagrant@kube-control1:~$ helm repo add bitnami https://chart.bitnami.com/bitnami
vagrant@kube-control1:~$ helm repo update

vagrant@kube-control1:~$ vim elastic.yaml

global:
  kibanaEnabled: true
master:
  masterOnly: false
  replicaCount: 1
  persistence:
    storageClass: "nfs-client"
data:
  replicaCount: 0
coordinating:
  replicaCount: 0
ingest:
  replicaCount: 0
kibana:
  service:
    type: LoadBalancer

vagrant@kube-control1:~$ helm install elasticsearch -f elastic.yaml bitnami/elasticsearch -n logging
NAME: elasticsearch
LAST DEPLOYED: Thu Jan 15 04:51:50 2026
NAMESPACE: logging
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: elasticsearch
CHART VERSION: 22.1.6
APP VERSION: 9.1.2
...

vagrant@kube-control1:~$ helm list -n logging
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
elasticsearch   logging         1               2026-01-15 04:51:50.668769501 +0000 UTC deployed        elasticsearch-22.1.6    9.1.2

vagrant@kube-control1:~$ kubectl get -n logging pod
NAME                                    READY   STATUS    RESTARTS   AGE
elasticsearch-kibana-59649b7f9f-27tvk   0/1     Pending   0          105s
elasticsearch-master-0                  0/1     Pending   0          105s

vagrant@kube-control1:~$ kubectl get -n logging svc
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)             AGE
elasticsearch             ClusterIP      10.233.18.177   <none>           9200/TCP,9300/TCP   2m53s
elasticsearch-kibana      LoadBalancer   10.233.37.204   192.168.56.201   5601:31609/TCP      2m53s
elasticsearch-master-hl   ClusterIP      None            <none>           9200/TCP,9300/TCP   2m53s

vagrant@kube-control1:~$ helm repo add fluent https://fluent.github.io/helm-charts
```

```sh
vagrant@kube-control1:~$ vim fluent-bit.yaml

config:
  output: |
    [OUTPUT]
      Name es
      Match *
      Host elasticsearch.logging.svc.cluster.local
      Port 9200
      # tls Off
      # tls.verify Off
      Retry_Limit False
      Logstash_Format On
      Logstash_Prefix fluent-bit
      Trace_Error On
      Replaces_Dots On
      Suppress_Type_Name On

vagrant@kube-control1:~$ helm upgrade --install fluent-bit -f fluent-bit.yaml fluent/fluent-bit -n logging
Release "fluent-bit" has been upgraded. Happy Helming!
NAME: fluent-bit
LAST DEPLOYED: Thu Jan 15 05:03:44 2026
NAMESPACE: logging
STATUS: deployed
REVISION: 2
NOTES:
Get Fluent Bit build information by running these commands:

export POD_NAME=$(kubectl get pods --namespace logging -l "app.kubernetes.io/name=fluent-bit,app.kubernetes.io/instance=fluent-bit" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace logging port-forward $POD_NAME 2020:2020
curl http://127.0.0.1:2020

vagrant@kube-control1:~$ helm list -n logging
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
elasticsearch   logging         1               2026-01-15 04:51:50.668769501 +0000 UTC deployed        elasticsearch-22.1.6    9.1.2
fluent-bit      logging         1               2026-01-15 04:59:00.875822153 +0000 UTC deployed        fluent-bit-0.54.1       4.1.1
```