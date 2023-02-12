# Домашнее задание к занятию "`10.1. Keepalived/vrrp`" - `Барановский Станислав`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

## Задание 1

Разверните топологию из лекции и выполните установку и настройку сервиса Keepalived.
```
vrrp_instance test {

state "name_mode"

interface "name_interface"

virtual_router_id "number id"

priority "number priority"

advert_int "number advert"

authentication {

auth_type "auth type"

auth_pass "password"

}

unicast_peer {

"ip address host"

}

virtual_ipaddress {

"ip address host" dev "interface" label "interface":vip

}

}

```
*Пришлите скриншот рабочей конфигурации и состояния сервиса для каждого нода.*
```
sudo apt update
apt install iptables-persistent keepalived
sudo iptables -I INPUT -p vrrp -d 224.0.0.18 -j ACCEPT
netfilter-persistent save
nano /etc/keepalived/keepalived.conf  # Содержимое файла keepalived.conf для нод master и backup ниже в блоке
sudo systemctl start keepalived
sudo systemctl enable keepalived
sudo systemctl status keepalived
ip add
```
Файл keepalived.conf для ноды MASTER
```
vrrp_instance test {
state MASTER
interface enp0s8
virtual_router_id 10
priority 110
advert_int 4

authentication {
auth_type AH
auth_pass password
}

unicast_peer {
192.168.56.3
}

virtual_ipaddress {
192.168.56.50 dev enp0s8 label enp0s8:vip
}

}
```
Файл keepalived.conf для ноды BACKUP
```
vrrp_instance test {
state BACKUP
interface enp0s8
virtual_router_id 10
priority 110
advert_int 4

authentication {
auth_type AH
auth_pass password
}

unicast_peer {
192.168.56.2
}

virtual_ipaddress {
192.168.56.50 dev enp0s8 label enp0s8:vip
}

}
```

![Скриншот рабочей конфигурации](https://github.com/StanislavBaranovskii/10-1-hw/blob/main/img/10-1-1.png "Скриншот рабочей конфигурации")

---

## Задание 2*

Проведите тестирование работы ноды, когда один из интерфейсов выключен. Для этого:

- добавьте ещё одну виртуальную машину и включите её в сеть;
- на машине установите Wireshark и запустите процесс прослеживания интерфейса;
- запустите процесс ping на виртуальный хост;
- выключите интерфейс на одной ноде (мастер), остановите Wireshark;
- найдите пакеты ICMP, в которых будет отображён процесс изменения MAC-адреса одной ноды на другой.

*Пришлите скриншот до и после выключения интерфейса из Wireshark.*
```
# На виртуальной машине 3 :
sudo touch /home/baranovskii/keepalived.pcap
sudo tshark -i enp0s8 -f icmp -w /home/baranovskii/keepalived.pcap
ping 192.168.56.50

# На ноде master :
sudo ip link set dev enp0s8 down  # На ноде master

# На виртуальной машине 3 :
tshark -T fields -r /home/baranovskii/keepalived.pcap -e icmp.data_time -e ip.src -e eth.src -e ip.dst -e eth.dst
```
![Скриншот из tshark](https://github.com/StanislavBaranovskii/10-1-hw-prometheus/blob/main/img/10-1-2.png "Скриншот из thark")

---
