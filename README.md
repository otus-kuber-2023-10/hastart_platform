# hastart_platform
hastart Platform repository
1. Отклонировал ветку в hastart_platform
2. Установил minikub  и kubectl

pavel@redos hastart_platform]$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured



3. Ответ на вопрос.

Pod-ы из этой системной области контроллируются напрямую kubelet и пересоздаются
 им автоматически без участия kube-apiserver


4. Создал докерфайл, создал на его основе образ и запушил на  докерхаб

https://hub.docker.com/repository/docker/hastartdocker/webcentos/general

5.  Поднял на его основе pod

[pavel@redos ~]$ k get pod
NAME                       READY   STATUS    RESTARTS      AGE

webnew                     1/1     Running   2 (19h ago)   47h


6. Посмотрим на сам Pod

[pavel@redos web]$ k describe pod webnew
Name:             webnew
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Mon, 20 Nov 2023 11:21:16 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Init Containers:
  init-index-html:
    Container ID:  docker://3522c00d3caff08c06947bf3e23b67af78b2cfb79240ee3390f16f435666600d
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 22 Nov 2023 11:06:04 +0300
      Finished:     Wed, 22 Nov 2023 11:06:25 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Containers:
  webnew:
    Container ID:   docker://5e286fe7bb9e84219a74231630c97a66aa72e02ad8cd9f7c3fcaa0e44bcf9c5c
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 22 Nov 2023 11:06:32 +0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Nov 2023 11:03:43 +0300
      Finished:     Tue, 21 Nov 2023 15:26:18 +0300
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-tpvb6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  16m   kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          15m   kubelet  Container image "busybox:1.31.0" already present on machine
  Normal  Created         15m   kubelet  Created container init-index-html
  Normal  Started         15m   kubelet  Started container init-index-html
  Normal  Pulled          15m   kubelet  Container image "hastartdocker/webcentos:v3" already present on machine
  Normal  Created         15m   kubelet  Created container webnew
  Normal  Started         15m   kubelet  Started container webnew
6. Посмотрим на сам Pod

[pavel@redos web]$ k describe pod webnew
Name:             webnew
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Mon, 20 Nov 2023 11:21:16 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Init Containers:
  init-index-html:
    Container ID:  docker://3522c00d3caff08c06947bf3e23b67af78b2cfb79240ee3390f16f435666600d
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 22 Nov 2023 11:06:04 +0300
      Finished:     Wed, 22 Nov 2023 11:06:25 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Containers:
  webnew:
    Container ID:   docker://5e286fe7bb9e84219a74231630c97a66aa72e02ad8cd9f7c3fcaa0e44bcf9c5c
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 22 Nov 2023 11:06:32 +0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Nov 2023 11:03:43 +0300
      Finished:     Tue, 21 Nov 2023 15:26:18 +0300
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-tpvb6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  16m   kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          15m   kubelet  Container image "busybox:1.31.0" already present on machine
  Normal  Created         15m   kubelet  Created container init-index-html
  Normal  Started         15m   kubelet  Started container init-index-html
  Normal  Pulled          15m   kubelet  Container image "hastartdocker/webcentos:v3" already present on machine
  Normal  Created         15m   kubelet  Created container webnew
  Normal  Started         15m   kubelet  Started container webnew
6. Посмотрим на сам Pod

[pavel@redos web]$ k describe pod webnew
Name:             webnew
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Mon, 20 Nov 2023 11:21:16 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Init Containers:
  init-index-html:
    Container ID:  docker://3522c00d3caff08c06947bf3e23b67af78b2cfb79240ee3390f16f435666600d
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 22 Nov 2023 11:06:04 +0300
      Finished:     Wed, 22 Nov 2023 11:06:25 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Containers:
  webnew:
    Container ID:   docker://5e286fe7bb9e84219a74231630c97a66aa72e02ad8cd9f7c3fcaa0e44bcf9c5c
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 22 Nov 2023 11:06:32 +0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Nov 2023 11:03:43 +0300
      Finished:     Tue, 21 Nov 2023 15:26:18 +0300
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-tpvb6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  16m   kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          15m   kubelet  Container image "busybox:1.31.0" already present on machine
  Normal  Created         15m   kubelet  Created container init-index-html
  Normal  Started         15m   kubelet  Started container init-index-html
  Normal  Pulled          15m   kubelet  Container image "hastartdocker/webcentos:v3" already present on machine
  Normal  Created         15m   kubelet  Created container webnew
  Normal  Started         15m   kubelet  Started container webnew
6. Посмотрим на сам Pod

[pavel@redos web]$ k describe pod webnew
Name:             webnew
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Mon, 20 Nov 2023 11:21:16 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Init Containers:
  init-index-html:
    Container ID:  docker://3522c00d3caff08c06947bf3e23b67af78b2cfb79240ee3390f16f435666600d
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 22 Nov 2023 11:06:04 +0300
      Finished:     Wed, 22 Nov 2023 11:06:25 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Containers:
  webnew:
    Container ID:   docker://5e286fe7bb9e84219a74231630c97a66aa72e02ad8cd9f7c3fcaa0e44bcf9c5c
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 22 Nov 2023 11:06:32 +0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Nov 2023 11:03:43 +0300
      Finished:     Tue, 21 Nov 2023 15:26:18 +0300
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-tpvb6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  16m   kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          15m   kubelet  Container image "busybox:1.31.0" already present on machine
  Normal  Created         15m   kubelet  Created container init-index-html
  Normal  Started         15m   kubelet  Started container init-index-html
  Normal  Pulled          15m   kubelet  Container image "hastartdocker/webcentos:v3" already present on machine
  Normal  Created         15m   kubelet  Created container webnew
  Normal  Started         15m   kubelet  Started container webnew
6. Посмотрим на сам Pod

[pavel@redos web]$ k describe pod webnew
Name:             webnew
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Mon, 20 Nov 2023 11:21:16 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Init Containers:
  init-index-html:
    Container ID:  docker://3522c00d3caff08c06947bf3e23b67af78b2cfb79240ee3390f16f435666600d
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 22 Nov 2023 11:06:04 +0300
      Finished:     Wed, 22 Nov 2023 11:06:25 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Containers:
  webnew:
    Container ID:   docker://5e286fe7bb9e84219a74231630c97a66aa72e02ad8cd9f7c3fcaa0e44bcf9c5c
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 22 Nov 2023 11:06:32 +0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Nov 2023 11:03:43 +0300
      Finished:     Tue, 21 Nov 2023 15:26:18 +0300
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-tpvb6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  16m   kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          15m   kubelet  Container image "busybox:1.31.0" already present on machine
  Normal  Created         15m   kubelet  Created container init-index-html
  Normal  Started         15m   kubelet  Started container init-index-html
  Normal  Pulled          15m   kubelet  Container image "hastartdocker/webcentos:v3" already present on machine
  Normal  Created         15m   kubelet  Created container webnew
  Normal  Started         15m   kubelet  Started container webnew
6. Посмотрим на сам Pod

[pavel@redos web]$ k describe pod webnew
Name:             webnew
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Mon, 20 Nov 2023 11:21:16 +0300
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.26
IPs:
  IP:  10.244.0.26
Init Containers:
  init-index-html:
    Container ID:  docker://3522c00d3caff08c06947bf3e23b67af78b2cfb79240ee3390f16f435666600d
    Image:         busybox:1.31.0
    Image ID:      docker-pullable://busybox@sha256:fe301db49df08c384001ed752dff6d52b4305a73a7f608f21528048e8a08b51e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      wget -O- https://tinyurl.com/otus-k8s-intro | sh
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 22 Nov 2023 11:06:04 +0300
      Finished:     Wed, 22 Nov 2023 11:06:25 +0300
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Containers:
  webnew:
    Container ID:   docker://5e286fe7bb9e84219a74231630c97a66aa72e02ad8cd9f7c3fcaa0e44bcf9c5c
    Image:          hastartdocker/webcentos:v3
    Image ID:       docker-pullable://hastartdocker/webcentos@sha256:f496ba548e06b29246904dd7290a9a1c1f7bcb97237c04720e9d8aa3ce58612d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 22 Nov 2023 11:06:32 +0300
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 21 Nov 2023 11:03:43 +0300
      Finished:     Tue, 21 Nov 2023 15:26:18 +0300
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /app from app (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tpvb6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  app:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-tpvb6:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From     Message
  ----    ------          ----  ----     -------
  Normal  SandboxChanged  16m   kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          15m   kubelet  Container image "busybox:1.31.0" already present on machine
  Normal  Created         15m   kubelet  Created container init-index-html
  Normal  Started         15m   kubelet  Started container init-index-html
  Normal  Pulled          15m   kubelet  Container image "hastartdocker/webcentos:v3" already present on machine
  Normal  Created         15m   kubelet  Created container webnew
  Normal  Started         15m   kubelet  Started container webnew



7  Открываем доступ  к поду по порту 8000

pavel@redos web]$ kubectl port-forward --address 0.0.0.0 pod/webnew  8000:8000
Forwarding from 0.0.0.0:8000 -> 8000

Handling connection for 8000




8. Проверил работу.

На экране появляется рыбка с цифрой 42.


9. Установил frontend из предложенного докерфайла. При запуске дает ошибку Error.  Посмотрел логи, ошибки были связаны с переменной PRODUCT_CATALOG_SERVICE_ADDR

  С помощью опции dry-dun сгенерировал ямлик и внес секцию env с данной переменной.  В итоге pod  работает:

[pavel@redos web]$ k get pod
NAME                       READY   STATUS    RESTARTS      AGE
frontend-6dd84fd98-qxkg8   1/1     Running   1 (20h ago)   23h
webnew                     1/1     Running   2 (20h ago)   2d
 
