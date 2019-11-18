## Краткая выжимка из документации Kubernetes


Для работы с kubernetes используется утилита kubectl. Можно, как использовать уже установленный kubectl, на мастере кластера, так и установить его на свой рабочий компьютер и настроить. Настройка сводиться к копированию файла `/etc/kubernetes/admin.conf` с мастера в папку `~/.kube/config` пользователя, от имени которого планируется работать.

Некоторые абстракции в kubernetes:
- Pod - минимальная единица в kubernetes. Это абстракция чуть выше одного контейнера. В Pod файле могут быть указаны общие характеристики для контейнеров, такие как сеть, метки, общие папки, доступные ресурсы, образ контейнера, количество запущенных образов. Если приложение требует последовательного запуска нескольких контейнеров они все описываются в одном файле Pod. Пример простого Pod файла [*ДОБАВИТЬ ССЫЛКУ*]

- Deployment — это абстракция Kubernetes, в котором описывается желаемое состояние Pod и не только. Deployment поддерживает такие поля, как политика перезапуска подов, стратегию обновления старых подов, селектор меток для подов (т.е. каким требованиям должен отвечать узел кластера, на котором будет развернут Pod. Метки ставятся отдельной командой `kubectl label pods pod_name new-label=awesome`)

- Service - используется для открытия доступа к приложению. В этой обструкции описывается, какой метод открытия доступа к приложению используется и какие порты при этом задействованы. Пример простого Service файла [*ДОБАВИТЬ ССЫЛКУ*]

## Пример создание запуска простого контейнера с web страницей

- создать Pod, Service и д.р. описанный в файле, можно командой `kubectl apply -f hello_world_pod.yaml`  (`kubectl apply -f hello_world_service.yml`)
- после создания Pod можно вывести информацию о них командой `kubectl get pod`
- или расширенную информацию `kubectl get pod -o wide`
пример вывода:
```
root@kmaster2:~# kubectl get pod -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP          NODE       NOMINATED NODE   READINESS GATES
hello-world-pod   1/1     Running   0          43s   10.36.0.1   kworker1   <none>           <none>
```
- для подробного описания pod - `kubectl describe pods hello-world-pod`
- вывести все метки всех подов - `kubectl get pods --show-labels`
- для подключения к контейнеру в Pod `kubectl exec -it pod_name -c container_name -- /bin/bash`, где  
pod_name - значение поля name блока metadata из файла с Pod
```
metadata:
  name: hello-world-pod
```
container_name - имя одного из (или единственного) контейнера в блоке spec из файла с Pod
```
spec:
  containers:
    - name: hello-world-cntr
```

- чтобы удалить Pod `kubectl delete pod hello-world-pod`

Команды для работы с node, service, deployment и д.р. аналогичны `kubectl get node -o wide`


## Пример конфигов

Простой Pod файл - https://kubernetes.io/docs/concepts/workloads/pods/
```
---
apiVersion: v1              # версия api, подробности https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/
kind: Pod                   # тип объекта, список типов можно посмотреть командой 'kubectl api-resources'
metadata:                   # метаданные, которые служат для опознания объекта
  name: hello-world-pod     # имя пода, оно отображается в первой колонке 'kubectl get pods'
  labels:                   # метки объекта
    app: hello-world-web    # эта метка помогает связать этот под с Service для проброса порта
spec:                       # блок требований, то чему должен соответствовать Pod. В этом блоке можно указать общие ресурсы, ограничения по ресурсам, сеть и д.р
  containers:               # описание контейнера
    - name: hello-world-cntr      # имя контейнера
      image: httpd:latest         # образ контейнера
      ports:                      
        - containerPort: 80       # порт, который контейнер будет слушать 
```

Простой файл service для Pod - https://kubernetes.io/docs/concepts/services-networking/service/
```
---
apiVersion: v1          # версия api
kind: Service           # тип объекта
metadata:               # метаданные
  name: hello-world-svc     # имя сервиса
spec:                       # блок требований
  type: NodePort            # один из типов Services: ClusterIP, NodePort, LoadBalancer, Ingress. https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types.
  ports:
    - port: 3050  # номер порта для доступа другим контейнерам
      targetPort: 80 # номер потра который слушает контейнер 
      nodePort: 31000  # номер порта, который будет проброшен за пределы сети кластера. Он выбирается в диапазоне 30000-32767. В этом примере доступ к контейнеру будет происходить по адресу http://kworker1:31000
  selector:                 # Селекторы полей позволяют выбирать ресурсы Kubernetes, исходя из значения одного или нескольких полей ресурсов https://kubernetes.io/ru/docs/concepts/overview/working-with-objects/field-selectors/
    app: hello-world-web    # перенаправлять трафик объекта с metadata.label у которого ключ и значение 'app: hello-world-web'
```