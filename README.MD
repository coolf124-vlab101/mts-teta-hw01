## Проект домашнего задания для МТС Тета модуль 2
###  Требования/ Requirements
На АРМ должны быть установлены:
 - ssh
 - ansible
 - git
 - kubectl
 - helm3
### Схема приложения [Ссылка на развернутое Приложение API из Домашного задания по модулю 2 (работает пока не выключено)](http://9f5f69f6-aa8c-4658-abf1-f6deeb4ebbba.mts-gslb.ru/WeatherForecast)
![схема приложения](https://github.com/coolf124-vlab101/mts-teta-hw01/blob/main/mts-teta-hw-01.drawio.png?raw=true)
### Список хостов
```
balancer-01 10.0.10.2 +publicIP
db-01 10.0.10.3
db-01 10.0.10.4
etcd-01 10.0.10.5
etcd-02 10.0.10.6
etcd-03 10.0.10.7
```
### 1. Подготовка
**1.1 Создать машины через интерфейс Hub.cloud.ms**  
**1.1 Склонировать проект `git clone git@github.com:coolf124-vlab101/mts-teta-hw01.git`**  
**1.3 Разрешить конфигурацию ansible из другой директории**  
```
export ANSIBLE_CONFIG="/shared/mts-teta/hw-01/ansible/ansible.cfg"
```
**1.4 Поправить ~/.ssh/config**  
```
Host bastion-balancer-01
  User appuser
  Hostname 77.105.185.127 #Заменить на публичный IP
  StrictHostKeyChecking=no
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
**1.5 Проставить IP адрес в строку подключения файла k8s/deployment.yml**  

### 2. Первичная настройка ВМ
```sh
cd ansible
#Проверить что машины доступны
ansible all -m ping
#проверить выполнение
ansible-playbook deploy_pgcluster.yml --check
#боевой запуск
ansible-playbook deploy_pgcluster.yml
#создать в базе таблицы для прогнозы погоды и создать тестовые записи в  таблицы для прогнозы погоды
ansible-playbook populater-postgresql.yml
```
### 3. Первичная настройка Приложения в кубе
**3.1 Скопировать конфиг**  
```sh
export KUBECONFIG=~/.kube/config:/shared/mts-teta/student_a.burdin@digitalms.ru.yaml
kubectl config view --flatten
kubectl config use-context student13@grand-lion-d99896
```
**3.2 Запускаем контейнерное приложение**  
```sh
helm update api-01 k8s/api/
```
**3.3 Создать тестовую зону 9f5f69f6-aa8c-4658-abf1-f6deeb4ebbba.mts-gslb.ru в техническом домене mts и правило GSLB**  

### 4. Проверяем что работает
```sh
kubectl get pods #Проверяем что контейнеры запущены и работает
kubectl get ingresses #Проверяем что Ingress работает и узнаем его IP
kubectl get ingresses #Проверяем что Ingress работает и узнаем его IP
#Добавляем в /etc/hosts запись для Ingress IP mts-teta-hw01-api если требуется. Переделал на публичную DNS запись 9f5f69f6-aa8c-4658-abf1-f6deeb4ebbba.mts-gslb.ru
curl http://9f5f69f6-aa8c-4658-abf1-f6deeb4ebbba.mts-gslb.ru/WeatherForecast #проверям что приложение работает - если таблицы пусты, то вернет пустой ответ, но без ошибки
#Должно вернуться что типа {"id":1,"name":"Moscow","forecast":[{"id":1,"cityId":1,"dateTime":0,"temperature":30,"summary":"test"}]}
```
