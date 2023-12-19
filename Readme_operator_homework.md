1.  Создадим Customer Resource Definition для последующего создания ресурса MySQL

В этом CRD необходимо указать следующие данные.
1. Имя CRD - mysqls.otus.homework
2. Тип приложения, который будет указан для создания CR - otus.homework
3. Сама схема, описывающая объект.

В схеме мы указываем такие поля объекта:

образ, база данных, пароль и размер. При этом обязательным полями будут только первые три.

Выполняем скрипт:

k apply -f crd.yaml 
customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created

Смотрим, что получилось.

[pavel@redos deploy]$ k get crd
NAME                           CREATED AT
addresspools.metallb.io        2023-12-01T09:07:11Z
bfdprofiles.metallb.io         2023-12-01T09:07:11Z
bgpadvertisements.metallb.io   2023-12-01T09:07:12Z
bgppeers.metallb.io            2023-12-01T09:07:12Z
communities.metallb.io         2023-12-01T09:07:12Z
ipaddresspools.metallb.io      2023-12-01T09:07:12Z
l2advertisements.metallb.io    2023-12-01T09:07:12Z
mysqls.otus.homework           2023-12-11T07:20:11Z

Наш CRD создан - mysqls.otus.homework

Далее создаем наш кастомный ресурс

k apply -f cr.yaml

Смотрим, что получилось

pavel@redos deploy]$ k describe mysqls.otus.homework mysql-instance
Name:         mysql-instance
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  otus.homework/v1
Kind:         MySQL
Metadata:
  Creation Timestamp:  2023-12-11T07:35:17Z
  Generation:          1
  Resource Version:    139112
  UID:                 a4e6e017-049d-4067-bbaa-306a3eb5bdad
Spec:
  Database:      otus-database
  Image:         mysql:5.7
  Password:      otuspassword
  storage_size:  1Gi

Создаем на питоне скрипт, который реализует саму функцию оператора, то есть запускает mysql

После чего выполняем его  (предварительно  устанавливаем kopf) :

[pavel@redos build]$ kopf run mysql.py 
/usr/local/lib/python3.8/site-packages/kopf/_core/reactor/running.py:179: FutureWarning: Absence of either namespaces or cluster-wide flag will become an error soon. For now, switching to the cluster-wide mode for backward compatibility.
  warnings.warn("Absence of either namespaces or cluster-wide flag will become an error soon."
[2023-12-12 11:44:02,337] kopf._core.engines.a [INFO    ] Initial authentication has been initiated.
[2023-12-12 11:44:02,379] kopf.activities.auth [INFO    ] Activity 'login_via_client' succeeded.
[2023-12-12 11:44:02,382] kopf._core.engines.a [INFO    ] Initial authentication has finished.
mysql.py:12: YAMLLoadWarning: calling yaml.load() without Loader=... is deprecated, as the default Loader is unsafe. Please read https://msg.pyyaml.org/load for full details.
  json_manifest = yaml.load(yaml_manifest)
[2023-12-12 11:44:05,917] kopf.objects         [INFO    ] [default/mysql-instance] Handler 'mysql_on_create' succeeded.
[2023-12-12 11:44:05,918] kopf.objects         [INFO    ] [default/mysql-instance] Creation is processed: 1 succeeded; 0 failed.

Проверим pvc

k get pvc
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-instance-pvc   Bound    pvc-216b5303-ad9e-4362-b5c7-ac70225ec8cc   1Gi        RWO            standard       14m

Он создан!


Теперь удалим все ресурсы, созданные контроллером

[pavel@redos build]$ kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted
[pavel@redos build]$ kubectl delete deployments.apps mysql-instance
deployment.apps "mysql-instance" deleted
[pavel@redos build]$ kubectl delete pvc mysql-instance-pvc
persistentvolumeclaim "mysql-instance-pvc" deleted
[pavel@redos build]$ kubectl delete pv mysql-instance-pv
persistentvolume "mysql-instance-pv" deleted
[pavel@redos build]$ kubectl delete svc mysql-instance
service "mysql-instance" deleted


Добавим  в скрипт питона функцию удаления объектов Customer Resource после удаления самого Custom Resource. Для этого контроллер должен иметь обработку этого  события:

@kopf.on.delete('otus.homework', 'v1', 'mysqls')
def delete_object_make_backup(body, **kwargs):
return {'message': "mysql and its children resources deleted"}


Вносим изменния в  скрипт.  Назовем  новый контроллер mysqlnew-operator.py

После чего запускаем новый скрипт. И удаляем 

kubectl delete mysqls.otus.homework mysql-instance
mysql.otus.homework "mysql-instance" deleted

Смотрим на сообщения от контроллера:

2023-12-14 11:16:41,299] kopf.objects         [INFO    ] [default/mysql-instance] Handler 'delete_object_make_backup' succeeded.
[2023-12-14 11:16:41,299] kopf.objects         [INFO    ] [default/mysql-instance] Deletion is processed: 1 succeeded; 0 failed.
[2023-12-14 11:16:41,584] kopf.objects         [WARNING ] [default/mysql-instance] Patching failed with inconsistencies: (('remove', ('status',), {'delete_object_make_backup': {'message': 'mysql and its children resources deleted'}}, None),)

Смотрим статус объектов

[pavel@redos deploy]$ k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                        STORAGECLASS   REASON   AGE
mysql-instance-pv                          1Gi        RWO            Retain           Available                                                        4m5s
pvc-16b4005e-ef01-4dad-869c-476e3d4b2e83   1Gi        RWO            Delete           Released    default/mysql-instance-pvc   standard                4m4s

Контроллер работает!
