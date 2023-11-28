[pavel@redos ~]$ kind create cluster --config kind-config.yaml 
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼 
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Configuring the external load balancer ⚖️ 
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining more control-plane nodes 🎮 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋

[pavel@redos ~]$  k get nodes
NAME                  STATUS   ROLES           AGE   VERSION
kind-control-plane    Ready    control-plane   81m   v1.27.3
kind-control-plane2   Ready    control-plane   80m   v1.27.3
kind-worker           Ready    <none>          78m   v1.27.3
kind-worker2          Ready    <none>          78m   v1.27.3



Чтобы заработал скрипт , необходимо добавить selector и выбирать под по метке.

После добавления  метки скрипт начинает работать.

pavel@redos ~]$ k apply -f frontend-replicaset.yaml 
replicaset.apps/frontend created
[pavel@redos ~]$ k get pods -w
NAME             READY   STATUS              RESTARTS   AGE
frontend-gtbf7   0/1     ContainerCreating   0          16s
frontend-gtbf7   0/1     Running             0          27s
frontend-gtbf7   1/1     Running             0          43s


[pavel@redos ~]$ k get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   1         1         1       107m
[pavel@redos ~]$ k scale replicaset frontend --replicas=3
replicaset.apps/frontend scaled
[pavel@redos ~]$ k get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         1       108m


[pavel@redos ~]$ kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
NAME             READY   STATUS        RESTARTS   AGE
frontend-gtbf7   1/1     Terminating   0          110m
frontend-gwp42   1/1     Running       0          92s
frontend-rpmwp   1/1     Running       0          92s
frontend-gwp42   1/1     Terminating   0          92s
frontend-gtfmq   0/1     Pending       0          1s
frontend-gtfmq   0/1     Pending       0          2s
frontend-rpmwp   1/1     Terminating   0          94s
frontend-tzvl2   0/1     Pending       0          1s
frontend-gtfmq   0/1     ContainerCreating   0          4s
frontend-tzvl2   0/1     Pending             0          3s
frontend-gtbf7   0/1     Terminating         0          110m
frontend-tzvl2   0/1     ContainerCreating   0          5s
frontend-r7lpt   0/1     Pending             0          1s
frontend-r7lpt   0/1     Pending             0          2s
frontend-rpmwp   0/1     Terminating         0          101s
frontend-gtbf7   0/1     Terminating         0          110m
frontend-rpmwp   0/1     Terminating         0          102s
frontend-gtbf7   0/1     Terminating         0          110m
frontend-rpmwp   0/1     Terminating         0          103s
frontend-gwp42   0/1     Terminating         0          105s
frontend-tzvl2   0/1     Running             0          11s
frontend-gwp42   0/1     Terminating         0          105s
frontend-gwp42   0/1     Terminating         0          105s
frontend-gtfmq   0/1     Running             0          14s
frontend-r7lpt   0/1     ContainerCreating   0          7s
frontend-r7lpt   0/1     Running             0          8s

Проверяем количество работающих подов:

^C[pavel@redos ~]$  kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-gtfmq   1/1     Running   0          2m8s
frontend-r7lpt   1/1     Running   0          2m1s
frontend-tzvl2   1/1     Running   0          2m6s

Вновь применяем скрипт.

[pavel@redos ~]$ k apply -f frontend-replicaset.yaml 
replicaset.apps/frontend configured
[pavel@redos ~]$  kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-r7lpt   1/1     Running   0          3m14

Количство подов вернулось к исходному  количеству.

Увеличиваем количество подов путем изменения параметра replicas в исходной коде:

pavel@redos ~]$  kubectl get pods -l app=frontend -w
NAME             READY   STATUS              RESTARTS   AGE
frontend-mr8gg   0/1     Running             0          5s
frontend-r7lpt   1/1     Running             0          6m3s
frontend-t4jv4   0/1     ContainerCreating   0          5s
frontend-t4jv4   0/1     Running             0          6s

Теперь их стало три.

Проверяем, из каких образов запущены поды:
kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'



"image":"docker.io/hastartdocker/frontend:v1

Запускаем поды с новым образом (измененный тэг)  v.0.0.2

Смотрим:

[pavel@redos ~]$ k apply -f frontend-replicaset.yaml 
replicaset.apps/frontend configured
[pavel@redos ~]$ kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-mr8gg   1/1     Running   0          32m
frontend-r7lpt   1/1     Running   0          38m
frontend-t4jv4   1/1     Running   0          32m

Ничего не произошло. 


Проверяем образ, который указан для ReplicaSet: 

k get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'

Получаем: 
hastartdocker/frontend:v0.0.2

То есть  для реплики указан новый образ.

Проверяем, из какого образа запущены работающие поды.

ubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'



"image":"docker.io/hastartdocker/frontend:v1


То есть поды остались старыми.


Replicaset не умеет обновлять шаблон контейнеров, поэтому используется старый образ.



Работа с Deployment

Создадим ресурс этого типа на основе старого образа, который был использован под ReplicaSet frontend

[pavel@redos ~]$ k apply -f paymentservice-deployment.yaml 
deployment.apps/paymentservice created
[pavel@redos ~]$ k get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
paymentservice   0/2     2            0           9s
[pavel@redos ~]$ k get rs
NAME                        DESIRED   CURRENT   READY   AGE
paymentservice-84dddcc566   2         2         0       15s
[pavel@redos ~]$ k get pods
NAME                              READY   STATUS    RESTARTS   AGE
paymentservice-84dddcc566-6pzgx   0/1     Running   0          19s
paymentservice-84dddcc566-gm5vf   0/1     Running   0          18s

Мы видим, что после создания Deployment были созданы как ReplicaSet, так и поды в количестве, указанные параметром replicas. В
 данном случае - две реплики.

Обновим образ , подставив в скрипт yaml новую версию образа v 0.0.2

aymentservice-84dddcc566-6pzgx   1/1     Running   0          11m
paymentservice-84dddcc566-gm5vf   1/1     Running   0          11m
[pavel@redos ~]$ k apply -f paymentservice-deployment.yaml | k get pods -l app=paymentservice -w
NAME                              READY   STATUS              RESTARTS   AGE
paymentservice-78b9444969-w72bt   0/1     ContainerCreating   0          4s
paymentservice-84dddcc566-6pzgx   1/1     Running             0          11m
paymentservice-84dddcc566-gm5vf   1/1     Running             0          11m
paymentservice-78b9444969-w72bt   0/1     Running             0          8s
paymentservice-78b9444969-w72bt   1/1     Running             0          22s
paymentservice-84dddcc566-6pzgx   1/1     Terminating         0          11m
paymentservice-78b9444969-zg5ct   0/1     Pending             0          1s
paymentservice-78b9444969-zg5ct   0/1     Pending             0          2s
paymentservice-78b9444969-zg5ct   0/1     ContainerCreating   0          3s
paymentservice-84dddcc566-6pzgx   0/1     Terminating         0          11m
paymentservice-84dddcc566-6pzgx   0/1     Terminating         0          11m
paymentservice-84dddcc566-6pzgx   0/1     Terminating         0          11m
paymentservice-84dddcc566-6pzgx   0/1     Terminating         0          11m
paymentservice-78b9444969-zg5ct   0/1     Running             0          11s
paymentservice-78b9444969-zg5ct   1/1     Running             0          23s


paymentservice-84dddcc566-gm5vf   1/1     Terminating         0          12m

Мы видим, что новые поды создаются постепенно, они создаются наряду с уже существующими. Затем 
старые поды уничтожаются. В итоге мы имеем следующее:

[pavel@redos ~]$  k get pods -l app=paymentservice 
NAME                              READY   STATUS    RESTARTS   AGE
paymentservice-78b9444969-w72bt   1/1     Running   0          4m1s
paymentservice-78b9444969-zg5ct   1/1     Running   0          3m38s
[pavel@redos ~]$  k get rs
NAME                        DESIRED   CURRENT   READY   AGE
paymentservice-78b9444969   2         2         2       4m49s
paymentservice-84dddcc566   0         0         0       16m

Мы имеем два новых пода и два  ReplicaSet
Информация о том, из каких образов они были развернуты:

pavel@redos ~]$ k get deployments -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                SELECTOR
paymentservice   2/2     2            2           22m   server       hastartdocker/paymentservice:v0.0.2   app=paymentservice

Мы видим, что образ для paymentservice Deployment был обновлен на новую версию  0.0.2

Что же касается ReplicaSet,  то у него сохранился еще и старый  вариант v0.0.1

[pavel@redos ~]$ k get rs -o wide
NAME                        DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                SELECTOR
paymentservice-78b9444969   2         2         2       11m   server       hastartdocker/paymentservice:v0.0.2   app=paymentse
rvice,pod-template-hash=78b9444969
paymentservice-84dddcc566   0         0         0       23m   server       hastartdocker/paymentservice:v0.0.1   app=paymentse
rvice,pod-template-hash=84dddcc566

k rollout history deployment paymentservice
deployment.apps/paymentservice 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>


C[pavel@redos ~]k get rs -o wide
NAME                        DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                SELECTOR
paymentservice-78b9444969   2         2         2       20m   server       hastartdocker/paymentservice:v0.0.2   app=paymentse
rvice,pod-template-hash=78b9444969
paymentservice-84dddcc566   1         1         0       31m   server       hastartdocker/paymentservice:v0.0.1   app=paymentse
rvice,pod-template-hash=84dddcc566
[pavel@redos ~]$ 
[pavel@redos ~]$ k get rs
NAME                        DESIRED   CURRENT   READY   AGE
paymentservice-78b9444969   0         1         1       20m
paymentservice-84dddcc566   2         2         2       32m

Мы видим, что в результате отката используется старый тип диплоймента (оканчивается на 566)
