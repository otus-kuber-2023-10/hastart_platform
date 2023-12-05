1. Создаем на основе образа webcentos новый под согласно предложенной инструкции.
    Добавляем в него проверку.

2. Смотрим, что в нем.

k describe pod/web
Name:             web
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Wed, 29 Nov 2023 10:09:46 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.31
IPs:
  IP:  10.244.0.31
Init Containers:
  init-index-html:
    Container ID:  docker://bcdc976938dd3df8a502166bf5b202d47a9ba7e43bdf66b1f73835406ea1dfb6
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 29 Nov 2023 10:09:51 +0300
      Finished:     Wed, 29 Nov 2023 10:09:55 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-smwvr (ro)
Containers:
  web:
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    Restart Count:  0
    Readiness:      http-get http://:80/index.html delay=0s timeout=1s period=10s #success=1 #failure=3
 ........................................................

  Warning  Unhealthy  40s (x19 over 3m9s)  kubelet            Readiness probe failed: Get "http://10.244.0.31:80/index.html": dial tcp 10.244.0.31:80: connect: connection refused

.........................................................................

   Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 


Итак , проверка завершилась неудачно.

3. Добавляем новый вариант проверки livenessProbe с проверкой порта 8000

  После чего сновы выполняем describe

  Получаем такой вывод:

  Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 

  То есть проверка прошла удачно.

4. Вопрос о проверки ситуации с ps aux | grep pid_name

   Эта проверка не имеет реального значения, поскольку вывод такой команды всегда имеет вывод со значением 0. То есть она всегда завершается успешно.

    Но если нам нужно проверить, работают ли команды в данном поде вообще, например, работает ли команда ps, то тогда можно такую проверку добавить.

5. Проверим работу deployment. На основе образа webcentos , который был использован в предыдушем примере,  собираем Deployment  с количество реплик = 1  и старым портом 80. Запускаем:
      k apply -f web-deployment-bad.yaml

  Смотрим  k describe deployment/web

 В выводе находим строчки:

  Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated

 Смотрим состояние пода
 k describe pod/web-948bc7c4f-cbvvr :

Warning  Unhealthy  52s (x71 over 10m)  kubelet            Readiness probe failed: Get "http://10.244.0.36:80/index.html": dial tcp 10.244.0.36:80: connect: connection refuse

  Теперь увеличиваем количество реплик до трех и меняем порт проверки с 80 на 8000:

 Количество подов увеличилось до трех

[pavel@redos kubernetes-networks]$  k get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-6855d765df-9v8vs   1/1     Running   0          37s
web-6855d765df-c8mzm   1/1     Running   0          54s
web-6855d765df-jhxvv   1/1     Running   0          23s

Смотрим состояние деплоя

Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable

То есть все хорошо. Смотрим состояния подов:

k describe pod/web-6855d765df-9v8vs

Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 

То есть поды работают штатно, все проверки завершились успешно.


Проверка работы параметров  RollingUpdate

При установке параметров  MaxSurge 100 %  и  maxUnavaille 0  наблюдал
 увеличение количества подов с трех до 6. То есть на 100%. Если установить параметр  maxUnavailable  1, то количество подов ожидаемо уменьшится на 1. Их будет 5.


Создаем сервис согласно предложенному ямлу. Получаем

[pavel@redos kubernetes-networks]$ k get services
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.96.0.1      <none>        443/TCP   16d
web-svc-cip   ClusterIP   10.98.186.82   <none>        80/TCP    45s

Смотрим на куберспай

[pavel@redos hastart_platform]$ kubespy trace service web-svc-cip
[ADDED v1/Service]  default/web-svc-cip
    ✅ Successfully created Endpoints object 'web-svc-cip' to direct traffic to Pods
    ✅ Successfully allocated a cluster-internal IP: web-svc-cip

[ADDED v1/Endpoints]  default/web-svc-cip
    ❌ Does not direct traffic to any Pods

Заходим в minikube по ssh, пытаемся сделать curl на Cluster IP

Пишет connection refused

Почему?

Потому что не были запущены поды деплоймента.

Запускаем деплоймент из примеры выше.

Kuberspy показывает, что сервис обслуживает наши поды

[pavel@redos hastart_platform]$ kubespy trace service web-svc-cip
[ADDED v1/Service]  default/web-svc-cip
    ✅ Successfully created Endpoints object 'web-svc-cip' to direct traffic to Pods
    ✅ Successfully allocated a cluster-internal IP: web-svc-cip

[ADDED v1/Endpoints]  default/web-svc-cip
    ✅ Directs traffic to the following live Pods:
       - [Ready] web-6855d765df-4c59k @ 10.244.0.56
       - [Ready] web-6855d765df-c7df7 @ 10.244.0.57

Теперь curl выводит index.html  ,  сервис РАБОТАЕТ! 


Теперь настроим сеть. Установим режим работы IPVS согласно предложенной инструкции

[pavel@redos ~]$ k --namespace kube-system edit configmap/kube-proxy
configmap/kube-proxy edited


После редактирования смотрим запущенные поды в kube-system

[pavel@redos ~]$ k --namespace kube-system get pods
NAME                               READY   STATUS    RESTARTS       AGE
coredns-5dd5756b68-vncbm           1/1     Running   9 (13h ago)    17d
etcd-minikube                      1/1     Running   9 (13h ago)    17d
kube-apiserver-minikube            1/1     Running   9 (13h ago)    17d
kube-controller-manager-minikube   1/1     Running   9 (13h ago)    17d
kube-proxy-k7jf4                   1/1     Running   9 (13h ago)    17d
kube-scheduler-minikube            1/1     Running   9 (13h ago)    17d
storage-provisioner                1/1     Running   24 (30m ago)   17d


Удаляем kube-proxy, который будет автоматически воссоздан

kubectl --namespace kube-system delete pod --selector='k8s-app=kube-proxy'

Мы видим, что под пересоздался. Но остались старые записи в таблицах iptables.

Очистим старые записи согласно предложенного алгоритма.


Теперь посмотрим на описание сервисов по команде 

k  get service -o yaml

Получим такую инфу о нашем сервисе:

ame: web-svc-cip
    namespace: default
    resourceVersion: "100795"
    uid: fba77885-96a4-4c4f-88f6-72247cba45d7
  spec:
    clusterIP: 10.107.91.55
    clusterIPs:
    - 10.107.91.55



Теперь этот адрес появился на интерфейсе  в  minikube


oot@minikube:~# ip addr show kube-ipvs0
10: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default 
    link/ether 4a:b0:ac:3c:7e:df brd ff:ff:ff:ff:ff:ff
    inet 10.99.233.195/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.96.0.1/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.96.0.10/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.98.71.118/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.108.164.111/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
    inet 10.107.91.55/32 scope global kube-ipvs0
       valid_lft forever preferred_lft forever


Создадим конфигурацию metallb, в котором укажем пул адресов

apiVersion: v1
kind: ConfigMap
metadata:
namespace: metallb-system
name: config
data:
config: |
 address-pools:
 - name: default
 protocol: layer2
 addresses:
 - "172.17.255.1-172.17.255.255"

После чего создадим LoadBalancer

apiVersion: v1
kind: Service
metadata:
  name: web-svc-lb
spec:
  selector:
    app: web
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
 


[pavel@redos kubernetes-networks]$ k describe  service/web-svc-lb
Name:                     web-svc-lb
Namespace:                default
Labels:                   <none>
Annotations:              metallb.universe.tf/ip-allocated-from-pool: first-pool
Selector:                 app=web
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.107.129.81
IPs:                      10.107.129.81
LoadBalancer Ingress:     172.17.255.1
Port:                     <unset>  80/TCP
TargetPort:               8000/TCP
NodePort:                 <unset>  30642/TCP
Endpoints:                10.244.0.95:8000,10.244.0.97:8000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason       Age   From                Message
  ----    ------       ----  ----                -------
  Normal  IPAllocated  46s   metallb-controller  Assigned IP ["172.17.255.1"]


Балансер заработал!


Теперь проверим режим работы наших сервисов

Если зайти на http://172.17.255.1/index.html,  то ничего не получится

Надо прокинуть адрес с помощью роутинга.

Узнаем наш IP виртуальной машины minikube:  minikube ip

Он дает адрес  192.168.58.2 

Теперь пропишем на хосте:

route add -net 172.17.255.0/24  gw 192.168.58.2

И теперь браузер открывает страницу http://172.17.255.1/index.html
