## Проект домашнего задания для МТС Тета модуль 2
### Список хостов
```
balancer-01 10.0.10.2 :public
db-01 10.0.10.3
db-01 10.0.10.4
etcd-01 10.0.10.5
etcd-01 10.0.10.6
etcd-01 10.0.10.7
```
### Подготовка
#Создать машины через интерфейс Hub.cloud.ms
#Разрешить конфигурацию ansible из другой директории
```
export ANSIBLE_CONFIG="/shared/mts-teta/hw-01/ansible/ansible.cfg"
```
#Поправить ~/.ssh/config
```
Host bastion-balancer-01
  User appuser
  Hostname 77.105.185.127 #Заменить на публичный IP
  StrictHostKeyChecking=no
  UserKnownHostsFile=/dev/null
  IdentityFile /home/user/.ssh/appuser
Host 10.0.10.3
 ProxyJump appuser@bastion-balancer-01
Host 10.0.10.4
 ProxyJump appuser@bastion-balancer-01
Host 10.0.10.5
 ProxyJump appuser@bastion-balancer-01
Host 10.0.10.6
 ProxyJump appuser@bastion-balancer-01
Host 10.0.10.7
 ProxyJump appuser@bastion-balancer-01
```
#Проставить IP адреса

### Первичная настройка ВМ
```
cd ansible
#Проверить что машины доступны
ansible all -m ping
#проверить выполнение
ansible-playbook deploy_pgcluster.yml --check
#боевой запуск
ansible-playbook deploy_pgcluster.yml
#заполнить базу
ansible-playbook populater-postgresql.yml
```
### Первичная настройка Приложения в кубе
#Скопировать конфиг
```
export KUBECONFIG=~/.kube/config:/shared/mts-teta/student_a.burdin@digitalms.ru.yaml
kubectl config view --flatten
kubectl config use-context student13@grand-lion-d99896
```
#Запускаем контейнерное приложение
```
helm update api-01 k8s/api/
```

### Проверяем что работает
```
kubectl get pods #Проверяем что контейнеры запущены и работает
kubectl get ingresses #Проверяем что Ingress
curl http://mts-teta-hw01-api/WeatherForecast #проверям что приложение работает - если таблицы пусты, то вернет пустой ответ, но без ошибки
```
