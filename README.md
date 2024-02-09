# demo2024 КИРИЛЛ ГЕЙ
# №1
Выполните базовую настройку всех устройств:

a. Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian 

b. Присвоить имена в соответствии с топологией

c. Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.

d. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.

e. Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.


          
 
 # Топология 
 
   ![image](https://github.com/Ganibal-24/demo2024/assets/148868527/f418c7cc-4fa7-475e-83b2-05b348a75161)

# Таблица 1

| Имя устройства | Интерфейс   | IPv4/IPv6        |    Маска/Префикс    | Шлюз         |
| -----------    | ----------- |------------      |   ---------------   |------        |
| ISP            | ens192      | 10.12.10.6       |  255.255.255.0/24   | 10.12.10.254 |
|                | ens256      | 192.168.0.162    |  255.255.255.252/30 |              |
|                | ens224      | 192.168.0.166    |  255.255.255.252/30 |              |
| HQ-R           | ens192      | 192.168.0.161    |  255.255.255.252/30 | 192.168.0.162|
|                | ens224      | 192.168.0.1      |  255.255.255.128/25 |              |
| BR-R           | ens192      | 192.168.0.165    |  255.255.255.252/30 | 192.168.0.166|
|                | ens224      | 192.168.0.129    |  255.255.255.224/27 |              |
| HQ-SRV         | ens192      | 192.168.0.126    |  255.255.255.128/25 | 192.168.0.1  |
| BR-SRV         | ens192      | 192.168.0.158    |  255.255.255.224/27 | 192.168.0.129|

Захожу в файл конфигурации:
```
nano /etc/network/interfaces
```
Ввожу IP-адреса, маску подсети и шлюз по умолчанию по следующему образцу:
```
auto ens192
iface ens192 inet static
        address 10.12.10.6
        netmask 255.255.255.0.
        gateway 10.12.10.254

auto ens256
iface ens256 inet static
        address 192.168.0.162
        netmask 255.255.255.252
        
auto ens224
iface ens224 inet static
        address 192.168.0.166
        netmask 255.255.255.252
```
Далее сохраняю:
```
ctrl + s
```
Далее выхожу из файла конфигурации:
```
ctrl + x
```
И перезагрузил виртуальную машину:
```
systemctl reboot
```
Точно также настроил остальные виртуальные машины.

# №2
Настройте внутреннюю динамическую маршрутизацию по средствам FRR. Выберите и обоснуйте выбор протокола динамической маршрутизации из расчёта, что в дальнейшем сеть будет масштабироваться.
# Ход выполнения работы:
Установил пакет FRR:
```
apt-get install frr
```
Проверил состояние:
```
systemctl status frr
```
Вошёл в файл:
```
nano /etc/frr/daemons
```
Изменил значение:
```
ospfd=yes
```
Перезагрузил службу FRR:
```
systemctl restart frr
```
Вошёл в настройку маршрутизации на ISP:
```
vtysh
```
Вхожу в конфигурацию терминала, затем запускаю процесс и добавляю сети:
```
conf t
router ospf
 network 192.168.1.160/30 area 0
 network 192.168.2.164/30 area 0
```
Просматриваю соседей:
```
show ip ospf neighbor
```
Далее нужно прописать sysctl -w net.ipv4.ip_forward=1. Затем раскомментировать сроку net.ipv4.ip_forward в файле /etc/sysctl.conf. Сохранить настройку в vtysh. Проверить настройку можно с помощью команды sysctl -a | grep forward.

Такие же действия повторяю с HQ-R и BR-R.
# №3
Настройте автоматическое распределение IP-адресов на роутере HQ-R.
# Ход выполнения работы:
Установка DHCP
```
apt-get install isc-dhcp-server
```
Вошёл в файл
```
nano /etc/default/isc-dhcp-server
```
Указал интерфейс который смотрит на HQ-SRV
```
INTERFACESv4="ens224"
Ctrl + s - сохранил изменения
Ctrl + x - вышел с файла
```
Далее следует настройка раздачи адресов, а для этого захожу в файл:
```
nano /etc/dhcp/dhcpd.conf
```
Прописываю конфиг
```
subnet 192.168.0.0 netmask 255.255.255.128 {
range 192.168.0.10 192.168.0.125;
option domain-name-servers 8.8.8.8, 8.8.4.4;
option routers 192.168.0.1;
}
Ctrl + s - сохранил изменения
Ctrl + x - вышел с файла
```
Применил настройку 
```
systemctl restart isc-dhcp-server.service
```
Проверить DHCP можно при помощи HQ-SRV, включив на нём DHCP.
# №4
Настройте локальные учётные записи на всех устройствах в соответствии с таблицей
| Учётная запись | Пароль   | Примечание        |   
| -----------    | -------- |------------       | 
| admin          | toortoor | HQ-SRV, HQ-R      | 
| branchadmin    | toortoor | BR-SRV, BR-R      | 
| networkadmin   | toortoor | HQ-R, BR-R, BR-SRV|
# Ход выполнения работы:
Добавляю пользователя:
```
adduser <имя пользователя>
usermod -aG root <имя пользователя>
```
Прописываю пароль и подтверждаю его. 

Захожу в файл /etc/passwd и проверяю, что созданный пользователей появился в файле.

Пример:
```
<имя пользовтеля>:x:0:501::/home/admin:/bin/bash
```
# №5
Настройте туннель между маршрутизаторами HQ-R И BR-R.
# Ход выполнения работы:
Установка графического интерфейса nmtui:
```
apt install network-manager
```
Заходим в интерфейс:
```
nmtui
```
![image](https://github.com/Ganibal-24/demo2024/assets/148868527/aa3f89b4-258d-416d-b640-841dc113e882)

Добавить IP tunnel

![image](https://github.com/Ganibal-24/demo2024/assets/148868527/7119312a-f41a-46c5-bf1e-7d04fe7f29ac)

![image](https://github.com/Ganibal-24/demo2024/assets/148868527/102808ba-b1f2-4e68-a05c-7b8afa776640)

Для HQ-R:
```
nmcli connection modify HQ-R ip-tunnel.ttl 64
ip r add 192.168.0.128/27 dev gre1
```
Для BR-R:
```
nmcli connection modify BR-R ip-tunnel.ttl 64
ip r add 192.168.0.0/25 dev gre1
```
# №6
Настройте NAT на всех маршрутизаторах 
# Ход выполнения работы:
Устанавливаем пакет:
```
apt-get -y install firewalld
```
Задаём автозагрузку:
```
systemctl enable --now firewalld
```
Прописываем интерфейс, который смотрит во внешнюю сеть:
```
firewall-cmd --permanent --zone=public --add-interface=ens192
```
Прописываем интерфейс, который смотрит во внутреннюю сеть:
```
firewall-cmd --permanent --zone=trusted --add-interface=ens224
firewall-cmd --permanent --zone=trusted --add-interface=ens256
```
Включаем NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохраняем правила:
```
firewall-cmd --reload
```
# №7
Измерьте пропускную способность сети между двумя узлами HQ-R-ISP
# Ход выполнения работы:
Установка:
```
apt-get -y install iperf3 
```
Делаем ISP в роли сервера:
```
iperf3 -s
```
Если порт не открыт, то:
```
iptables -A INPUT -p tcp --dport 5201 -j ACCEPT
```
На HQ-R пишем:
```
iperf3 -c 192.168.0.162 -f M
```
# №8
Составьте backup скрипты для сохранения конфигурации сетевых устройств, а именно HQ-R и BR-R. Продемонстрируйте их работу.
# Ход выполнения работы:
Создал каталог, где в будущем будет храниться конфигурация:
```
mkdir /etc/frr/backup
```
Установка службы:
```
apt-get install rsync
```
Вход в файл:
```
crontab -e
```
Вписываем скрипт:
```
0 0 * * * rsync /etc/frr/frr.conf /etc/frr/backup
```
Где 0 0 * * * это минута час день месяц день недели.

В данном случае нам не нужны параметры дня, месяца и дня недели, поэтому мы прописываем только минуты и часы.

/etc/frr/frr.conf это файл, который мы хотим сохранить, а /etc/frr/backup это наш каталог, в котором всё и будет храниться.

Когда настанет время, которое вы указали, нужно будет зайти в каталог и убедиться, что скрипт сработал:
```
ls /etc/frr/backup
```
