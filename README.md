# Домашнее задание к занятию  «Очереди RabbitMQ» - `Смирнов Максим`

### Задание 1. Установка RabbitMQ

*Используя Vagrant или VirtualBox, создайте виртуальную машину и установите RabbitMQ.Добавьте management plug-in и зайдите в веб-интерфейс.*

Создавалась машина в облаке. Команды для установки:

```
sudo apt update
sudo apt install rabbitmq-server -y

sudo systemctl enable rabbitmq-server
sudo systemctl start rabbitmq-server
sudo systemctl status rabbitmq-server

sudo rabbitmq-plugins enable rabbitmq_management

sudo rabbitmqctl add_user myuser mypassword
sudo rabbitmqctl set_user_tags myuser administrator
sudo rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"
```
`Работающий сервер в веб`

![rabbit-web.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/855f8a42a116c2569df0c3cf6ffaa132167fdb84/rabbit-web.jpg)

---

### Задание 2. Отправка и получение сообщений

*Используя приложенные скрипты, проведите тестовую отправку и получение сообщения. Для отправки сообщений необходимо запустить скрипт producer.py.*

Скрипты были модифицировано чтобы запускать их удаленно

`producer.py`
```
#!/usr/bin/env python
# coding=utf-8
import pika

credentials = pika.PlainCredentials('test', 'test')
parameters = pika.ConnectionParameters(
    host='89.169.182.128', 
    port=5672,
    virtual_host='/',
    credentials=credentials
)


connection = pika.BlockingConnection(parameters)
channel = connection.channel()

channel.queue_declare(queue='hello')

channel.basic_publish(
    exchange='', 
    routing_key='hello', 
    body='Hello Netology!'
)

print(" [x] Sent 'Hello Netology!'")

connection.close()
```

`consumer.py`
```
#!/usr/bin/env python
# coding=utf-8
import pika

credentials = pika.PlainCredentials('test', 'test')

parameters = pika.ConnectionParameters(
    host='51.250.22.176',     
    port=5672,              
    virtual_host='/',        
    credentials=credentials
)


connection = pika.BlockingConnection(parameters)
channel = connection.channel()

channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)


channel.basic_consume(
    queue='hello', 
    on_message_callback=callback, 
    auto_ack=True
)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

*Зайдите в веб-интерфейс, найдите очередь под названием hello и сделайте скриншот.*

`Сообщение из очереди hello`
![publisher-py.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/e4c7df8cd1119a888b16f7cbf76537c675ba5e4b/publisher-py.jpg)


*После чего запустите второй скрипт consumer.py и сделайте скриншот результата выполнения скрипта*

`Запуск consumer.py и результат`
![consumer-py.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/e4c7df8cd1119a888b16f7cbf76537c675ba5e4b/consumer-py.jpg)
*В качестве решения домашнего задания приложите оба скриншота, сделанных на этапе выполнения.* - см выше

---

### Задание 3. Подготовка HA кластера

*Используя Vagrant или VirtualBox, создайте вторую виртуальную машину и установите RabbitMQ. Добавьте в файл hosts название и IP-адрес каждой машины, чтобы машины могли видеть друг друга по имени.*

Содержимое hosts файла:
```
89.169.182.128 compute-vm-2-2-10-hdd-1768307187182
51.250.22.176 zabbix-serv
```

*Затем объедините две машины в кластер и создайте политику ha-all на все очереди. В качестве решения домашнего задания приложите скриншоты из веб-интерфейса с информацией о доступных нодах в кластере и включённой политикой.*

`Доступные ноды`
![cluster-nodes.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/f2e6b6a45c95b8697cf3af30fe6b0018c1d19f76/cluster-nodes.jpg)

`Активная политака ha-all`
![ha-all-on.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/f2e6b6a45c95b8697cf3af30fe6b0018c1d19f76/ha-all-on.jpg)

*Также приложите вывод команды `rabbitmqctl cluster_status` с двух нод:*

`Статус ноды 1`
![status-node1.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/8067ab1ee7e123665462f064fc07381cd0f6101e/status-node1.jpg)

`Статус ноды 2`
![status-node1.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/8067ab1ee7e123665462f064fc07381cd0f6101e/status-node2.jpg)

*Для закрепления материала снова запустите скрипт producer.py и приложите скриншот выполнения команды `rabbitmqadmin get queue` на каждой из нод*

`Вывод команды get queue на ноде 1`
![getqueue-node1.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/f2e6b6a45c95b8697cf3af30fe6b0018c1d19f76/getqueue-node1.jpg)

`Вывод команды get queue на ноде 2`
![getqueue-node1.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/f2e6b6a45c95b8697cf3af30fe6b0018c1d19f76/getqueue-node2.jpg)

*После чего попробуйте отключить одну из нод, желательно ту, к которой подключались из скрипта, затем поправьте параметры подключения в скрипте consumer.py на вторую ноду и запустите его.*
*Приложите скриншот результата работы второго скрипта.*

`Отключена первая нода`
![node1-stop.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/8067ab1ee7e123665462f064fc07381cd0f6101e/node1-stop.jpg)

`Работа скрипта приема сообщений с подключением ко второй ноде`
![consumer-node1stopped.jpg](https://github.com/turboturtle-90/KrolikMQ/blob/8067ab1ee7e123665462f064fc07381cd0f6101e/consumer-node1stopped.jpg)


