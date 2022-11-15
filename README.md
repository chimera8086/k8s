# k8s
[TOC]

Разработчик - Звягина В.О.  
Ansible playbook для базовой установки кластера kubernetes с высокой доступностью на ОС Debian.


|ПО|Версия|
|:------|:-------|
|kubeadm|1.24.7-00|
|kubelet|1.24.7-00|
|kubectl|1.24.7-00|
|etcd|3.5.3|
|haproxy|alpine3.16|
|Debian|11|
|CRI|containerd|
|Weaveworks|v2.8.1|

## Состав и схема кластера kubernetes

Кластер kubernetes из 2-х мастеров и 1 воркера (kmaster1, kmaster2, kworker1).

Etcd на 3-х отдельных машинах (etcd-1, etcd-2, etcd3).  

Доступ к api kubernetes по-общему ip - 10.10.20.30 (переменная kube_api) и порт 6443 (переменная kube_api_port). IP адрес находится на одной из 3-х машин (etcd-1, etcd-2, etcd3) и поднимается keepalived. На etcd-1, etcd-2, etcd3 установлен HAproxy, который перенаправляет запросы с порта 6443 на этот же порт к одному из мастеров кластера (kmaster1, kmaster2).  

В данной схеме есть некоторая избыточность в виде HAproxy, которая добавлянела в учебных целях.

## Описание playbook

**k8s_run.yml** - основной playbook. В нем происходит установка зависимых программ: etcd, haproxy, keepalived, docker, пакеты kubernetes, далее инициализация кластера, добавление мастеров и воркеров. С помощью тэгов происходит выбор действий: 
- **init** - инициализация кластера, добавление мастеров и воркеров из инвентарного файла.
- **master_add** - обавление мастеров. Появление новых мастеров в инвентарном затрагивает роли `haproxy` и `keepalived` и вызывает перезагрузку сервиса на машинах поочереди.
- **worker_add** - воркеров из инвентарного файла. Появление новых воркеров в инвентарном не затрагивает другие сервисы.

**tasks/hello_world.yml** - playbook для поднятия или удаления тестового Pod с Hello World с сервисом к нему. Через 1-2 минуты после прокатки, есть возожность открыть через браузер страницу веб сервера из докера. Дополнительная информация есть в файле README_Hello_world.md . С помощью тэгов происходит выбор действий: 
* **create** - поднятия тестового Pod
* **remove** - удаление тестового Pod


**reconfig.yml** - playbook для замены установленного сетевого модумя CNI на другой. В этом плейбуке есть 3 модуля: *calico*, *waeve*, *flannel*. Таск по удалению выбирается автоматически по результату команды `kubectl get clusterroles`. Если утановить какой-то другой модуль удаления не произойдет. Имя модуля для установки задается через переменную *install_cni*. Если ее не задать, то не будет установлено. Для перехода с *flannel* на *calico* написан отдельный таск, для live миграции, без остановки.

## Примеры запуска
Предполагается, что все хосты из inventory.yaml доступны по dns. Все команды на удаленных машинах выполняются от root. Прокатка ansible происходит из папки k8s.  
|Команда|Действие|
|:------|:-------|
|Инициализация кластера kubernetes | `./bin/ansible-playbook k8s_run.yml -i staging.yml` |
|Добавление мастеров в кластер |`./bin/ansible-playbook k8s_run.yml -i staging.yml --tags "master_add"` |
|Добавление воркеров в кластер |`./bin/ansible-playbook k8s_run.yml -i staging.yml --tags "worker_add"` |
|Поднятие тестового Pod |`./bin/ansible-playbook tasks/hello_world.yml -i staging.yml --tags "create"`|
|Удаления тестового Pod |`./bin/ansible-playbook tasks/hello_world.yml -i staging.yml --tags "remove"`|
|Замена одного текущего CNI на другой |`./bin/ansible-playbook tasks/reconfig.yml -i staging.yml --extra-vars install_cni=calico`|
