1. Сначала  создаем Statefulness 
[pavel@redos hastart_platform]$ k apply -f minio-statefuleness.yaml 
statefulset.apps/minio created
 или
[pavel@redos hastart_platform]$ k apply -f https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Kuberenetes-volumes/minio-statefulset.yaml
statefulset.apps/minio configured
2. Затем создаем сервис для работы с этим объектом.

[pavel@redos hastart_platform]$ k apply -f minio-headless-service.yaml 
service/minio created
[pavel@redos hastart_platform]$ k get all
NAME          READY   STATUS    RESTARTS   AGE
pod/minio-0   1/1     Running   0          14m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP    63m
service/minio        ClusterIP   None         <none>        9000/TCP   5m51s

NAME                     READY   AGE
statefulset.apps/minio   1/1     14m


[pavel@redos hastart_platform]$ k get statefulsets
NAME    READY   AGE
minio   1/1     25m
[pavel@redos hastart_platform]$ k get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          25m
[pavel@redos hastart_platform]$ k get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-bdaba60d-d34f-4844-b8e7-3860aca43c47   10Gi       RWO            standard       69m
[pavel@redos hastart_platform]$ k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-bdaba60d-d34f-4844-b8e7-3860aca43c47   10Gi       RWO            Delete           Bound    default/data-minio-0


3. Теперь поработаем с объектами PV и PVC.


На основании предложенных скриптов создаем объекта PV и PVC.

Затем создаем под  my-pod и убеждаемся, что под получил свой PV

pavel@redos kubernetes-volumes]$  k exec -it my-pod -- /bin/bash
root@my-pod:/# df -h
Filesystem                 Size  Used Avail Use% Mounted on
overlay                     69G   26G   40G  39% /
tmpfs                       64M     0   64M   0% /dev
/dev/mapper/ro_redos-root   69G   26G   40G  39% /app/data
shm                         64M     0   64M   0% /dev/shm
tmpfs                      3.8G   12K  3.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                      1.9G     0  1.9G   0% /proc/asound
tmpfs                      1.9G     0  1.9G   0% /proc/acpi
tmpfs                      1.9G     0  1.9G   0% /proc/scsi
tmpfs                      1.9G     0  1.9G   0% /sys/firmware


4. Теперь проверяем  работу томов. Создаем запись.

root@my-pod:/# echo "Hello, Kubernetes Volumes!" > /app/data/data.txt

5. Удаляем под и создаем новый под my-pod-2 с тем же томом.


[pavel@redos kubernetes-volumes]$ k  apply -f my-pod-2.yaml 
pod/my-pod-2 created
[pavel@redos kubernetes-volumes]$  k exec -it my-pod-2 -- /bin/bash
root@my-pod-2:/# df -h
Filesystem                 Size  Used Avail Use% Mounted on
overlay                     69G   26G   40G  39% /
tmpfs                       64M     0   64M   0% /dev
/dev/mapper/ro_redos-root   69G   26G   40G  39% /app/data
shm                         64M     0   64M   0% /dev/shm
tmpfs                      3.8G   12K  3.8G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                      1.9G     0  1.9G   0% /proc/asound
tmpfs                      1.9G     0  1.9G   0% /proc/acpi
tmpfs                      1.9G     0  1.9G   0% /proc/scsi
tmpfs                      1.9G     0  1.9G   0% /sys/firmware
root@my-pod-2:/# ls /app/data/
data.txt
root@my-pod-2:/# cat  /app/data/data.txt 
Hello, Kubernetes Volumes!


Все данные сохранились! 
