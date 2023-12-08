Task01

1. Создаем serviceaccount bob в namespace  kube-system
k -n kube-system create serviceaccount bob
2. Создаем yaml скрипт для привязки  bob_clusteradmin  роли cluster-admin к bob
3. Выполняем скрипт bob_clusteradmin.yaml .

Получаем

[pavel@redos kubernetes-security]$ k describe   clusterrolebindings.rbac.authorization.k8s.io/bob:admin
Name:         bob:admin
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind            Name  Namespace
  ----            ----  ---------
  ServiceAccount  bob   kube-system



4.Создаем serviceaccount dave
k -n kube-system create serviceaccount dave

5. Создаем роль no-rights, которая не будет иметь никаких прав скриптом  cr_no.yaml

k describe  clusterroles.rbac.authorization.k8s.io/no-rights
Name:         no-rights
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
             []                 []              []

6. Привязываем  эту роль к  dave

k apply -f dave_norights.yaml 
clusterrolebinding.rbac.authorization.k8s.io/dave:norights created


7. Сморим на binding

[pavel@redos kubernetes-security]$ k describe    clusterrolebindings.rbac.authorization.k8s.io/dave:norights
Name:         dave:norights
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  no-rights
Subjects:
  Kind            Name  Namespace
  ----            ----  ---------
  ServiceAccount  dave  kube-system



Task02

1. Создаем namespace prometheus
 k create namespace prometheus

2. Создаем account в нем

k -n prometheus create serviceaccount carol

3. Создаем роль getwatchlist скриптом carol-cr.yaml



k describe clusterrole.rbac.authorization.k8s.io/getwatchlist
Name:         getwatchlist
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [get watch list]

4. Создаем ClusterRoleBinding c помощью скрипта carol_getwatchlist.yaml
   

k describe clusterrolebinding.rbac.authorization.k8s.io/carol:getwatchlist
Name:         carol:getwatchlist
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  getwatchlist
Subjects:
  Kind            Name   Namespace
  ----            ----   ---------
  ServiceAccount  carol  prometheus


Task03

1. Создаем  аккаунт и неймспейс

pavel@redos kubernetes-security]$ k create namespace dev
namespace/dev created
[pavel@redos kubernetes-security]$ k -n dev create serviceaccount jane
serviceaccount/jane created

2. Привязываем  роль admin к  jane  скриптом  dane_admin.yaml

Получаем :

k -n dev describe rolebinding.rbac.authorization.k8s.io/dane:admin
Name:         dane:admin
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  admin
Subjects:
  Kind            Name  Namespace
  ----            ----  ---------
  ServiceAccount  dane  dev

3. Создаем  аккаунт  и  привязываем роль скриптом ken_view.yaml

k -n dev describe rolebinding.rbac.authorization.k8s.io/ken:view
Name:         ken:view
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  view
Subjects:
  Kind            Name  Namespace
  ----            ----  ---------
  ServiceAccount  ken   dev

