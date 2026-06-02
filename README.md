                                                #Методические рекомендации по работе в ОС семейства ALT Linux


                                                                         #Введение
Настоящие рекомендации содержат перечень типовых команд, применяемых при настройке сетевой подсистемы и базовых сервисов в операционных системах на платформе ALT Linux. Материал ориентирован на системных администраторов, выполняющих работы по организации сетевого взаимодействия, управлению маршрутизацией, настройке файловых систем и служб удалённого доступа.

# Основные разделы
1. Управление сетевыми интерфейсами
Конфигурация сетевых интерфейсов осуществляется путём создания файлов в каталоге /etc/net/ifaces/. Для каждого интерфейса создаётся отдельная поддиректория, где указываются тип интерфейса, метод назначения адреса (статический или динамический) и параметры маршрутизации.

Пример для статического адреса:

создание каталога интерфейса;

указание типа интерфейса TYPE=eth;

отключение автоматического управления через NM_CONTROLLED=no;

назначение IP-адреса и маски в файле ipv4address;

определение шлюза по умолчанию в файле ipv4route.

После изменения конфигурации сеть перезапускается командой systemctl restart network.

2. Настройка VLAN
Виртуальные локальные сети (VLAN) настраиваются на основе физического интерфейса. В каталоге /etc/net/ifaces/ создаётся поддиректория с именем физический_интерфейс.VLAN_ID, содержащая:

options с параметрами TYPE=vlan, HOST=физический_интерфейс, VID=номер_VLAN, DISABLED=no, BOOTPROTO=static;

ipv4address с IP-адресом и маской в формате CIDR.

3. Управление маршрутизацией и NAT
Включение пересылки пакетов выполняется параметром net.ipv4.ip_forward = 1 в файле /etc/net/sysctl.conf. Для организации NAT используется утилита nftables. Правила маскарадинга задаются в таблице ip nat с цепочкой postrouting.

4. Настройка DNS и синхронизации времени
Для работы с доменными именами в файле /etc/resolv.conf прописываются адреса DNS-серверов. Для предотвращения перезаписи используется команда chattr +i /etc/resolv.conf. Синхронизация времени настраивается с помощью службы chronyd: на сервере задаются разрешённые сети, на клиентах указывается адрес сервера.

5. Управление пользователями и SSH-доступом
Создание пользователей выполняется командой useradd с указанием UID при необходимости. Для предоставления прав администратора пользователь добавляется в группу wheel. В конфигурационном файле /etc/openssh/sshd_config задаются порт, ограничение числа попыток входа и баннер.

6. Работа с файловыми хранилищами (RAID, NFS)
Программный RAID настраивается с помощью mdadm. После создания массива и форматирования он монтируется в целевой каталог, запись о монтировании добавляется в /etc/fstab. Для организации сетевого файлового сервера NFS создаётся экспортируемый каталог, в файл /etc/exports добавляется правило с указанием разрешённой сети и параметров доступа. На клиенте выполняется монтирование удалённой папки командой mount.

# Заключение
Представленные команды и подходы являются базовыми для администрирования сетевых и серверных компонентов в среде ALT Linux. Последовательное применение описанных шагов позволяет развернуть работоспособную инфраструктуру с заданными сетевыми параметрами, обеспечить удалённый доступ и организовать общее файловое пространство.


МОДУЛЬ 1


1. Базовая настройка устройств

Таблица 1 — Таблица адресации
Устройства	Интерфейс	IP-адреса	Маска	VLAN	Подсеть	Шлюз
ISP	Enp7s1	DHCP	-	-	-	-
	Enp7s2 (К HQ-RTR)	172.16.1.1	/28	-	172.16.1.0/28	-
	Enp7s3 (К BR-RTR)	172.16.2.1	/28	-	172.16.2.0/28	-
HQ-RTR	Enp7s1 (к ISP)	172.16.1.2	/28	-	172.16.1.0/28	172.16.1.1
	Enp7s2.100 (к HQ-SRV)	192.168.100.1	/27	100	192.168.99.0/27	-
	Enp7s2.200 (к HQ-CLI)	192.168.200.1	/28	200	192.168.99.0/28	-
	Enp7s2.999 (управление)	192.168.99.1	/29	999	192.168.99.0/29	-
	Gre1 (IP тунель)	10.10.10.1	/30	-	10.10.10.0/30	-
BR-RTR	enp7s1 (к ISP)	172.16.2.2	/28	-	172.16.2.0/28	172.16.2.1
	enp7s2 (к BR-SRV)	192.168.1.1	/28	-	192.168.1.0/28	-
	gre1 (IP туннель)	10.10.10.2	/30	-	10.10.10.0/30	-
HQ-SRV	enp7s1.100 (к HQ-RTR)	192.168.100.2	/27	100	192.168.100.0/27	192.168.100.1
BR-SRV	enp7s1 (к BR-RTR)	192.168.1.2	/28	-	192.168.1.0/28	192.168.1.1
HQ-CLI	enp7s1.200 (к HQ-RTR)	192.168.200.2	/28	200	192.168.200.0/28	192.168.200.1

1.1 Настройка имени хоста (FQDN) и часового пояса на всех машинах

ISP
hostnamectl hostname ISP && exec bash
timedatectl set-timezone Europe/Tver

HQ-RTR
hostnamectl hostname hq-rtr.au-team.irpo && exec bash
timedatectl set-timezone Europe/ Tver

BR-RTR
hostnamectl hostname br-rtr.au-team.irpo && exec bash
timedatectl set-timezone Europe/ Tver

HQ-SRV
hostnamectl hostname HQ-SRV.au-team.irpo && exec bash
timedatectl set-timezone Europe/ Tver

BR-SRV
hostnamectl hostname BR-SRV.au-team.irpo && exec bash
timedatectl set-timezone Europe/ Tver

HQ-CLI
hostnamectl hostname HQ-CLI.au-team.irpo && exec bash
timedatectl set-timezone Europe/ Tver
Проверка: hostname → timedatectl

2. Настройка ISP

Настроить внешний интерфейс (DHCP), внутренние интерфейсы, NAT и маршрутизацию.

2.1 Настройка интерфейсов ISP

Внешний интерфейс (DHCP от вышестоящего роутера)
mkdir -p /etc/net/ifaces/enp7s1
cat > /etc/net/ifaces/enp7s1/options << EOF
BOOTPROTO=dhcp
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
EOF

Внутренние интерфейсы
mkdir -p /etc/net/ifaces/enp7s{2,3}
echo 'TYPE=eth' | tee /etc/net/ifaces/enp7s{2,3}/options

IP для HQ-RTR и BR-RTR
echo '172.16.1.1/28' > /etc/net/ifaces/enp7s2/ipv4address
echo '172.16.2.1/28' > /etc/net/ifaces/enp7s3/ipv4address

2.2 Включение маршрутизации и настройка NAT

sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/' /etc/net/sysctl.conf

apt-get update && apt-get install nftables -y

cat > /etc/nftables/nftables.nft << EOF
flush ruleset
table ip nat {
 chain postrouting {
 type nat hook postrouting priority srcnat;
 oifname "enp7s1" masquerade
 }
}
EOF

systemctl enable --now nftables
systemctl restart network
Проверка: ip a — ping 8.8.8.8

3. Создание локальных учётных записей

Создать пользователей net_admin (на роутерах) и sshuser (на серверах).

3.1 На HQ-RTR и BR-RTR (пользователь net_admin)

useradd net_admin
echo "net_admin:P@ssw0rd" | chpasswd
usermod -aG wheel net_admin
echo "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/net_admin
su -l net_admin
sudo id

3.2 На HQ-SRV и BR-SRV (пользователь sshuser)

useradd -u 2026 sshuser
echo "sshuser:P@ssw0rd" | chpasswd
usermod -aG wheel sshuser
echo "WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/sshuser
su -l sshuser
sudo id
Проверка: id net_admin / id sshuser



4. Коммутация (VLAN)

Настроить VLAN 100, 200, 999 на HQ-RTR и подключить к ним клиентов.

4.1 Настройка VLAN на HQ-RTR

echo $'100\n200\n999' | xargs -i bash -c 'echo -e "TYPE=vlan\nHOST=enp7s2\nVID={}" > /etc/net/ifaces/enp7s2.{}/options'

cat /etc/net/ifaces/vlan999/options 

echo '192.168.100.1/27' > /etc/net/ifaces/enp7s2.100/ipv4address
echo '192.168.200.1/28' > /etc/net/ifaces/enp7s2.200/ipv4address
echo '192.168.99.1/29' > /etc/net/ifaces/enp7s2.999/ipv4address
4.2 Настройка VLAN на HQ-SRV и HQ-CLI

HQ-SRV (VLAN 100)
mkdir -p /etc/net/ifaces/enp7s1.100
cat > /etc/net/ifaces/enp7s1.100/options << EOF
TYPE=vlan
HOST=enp7s1
VID=100
DISABLED=no
BOOTPROTO=static
EOF

echo '192.168.100.2/27' > /etc/net/ifaces/enp7s1.100/ipv4address
echo 'default via 192.168.100.1' > /etc/net/ifaces/enp7s1.100/ipv4route
echo 'nameserver 8.8.8.8' > /etc/net/ifaces/enp7s1.100/resolv.conf

systemctl restart network
ping zz.ru -c3

HQ-CLI (VLAN 200)
mkdir -p /etc/net/ifaces/enp7s1.200
cat > /etc/net/ifaces/enp7s1.200/options << EOF
TYPE=vlan
HOST=enp7s1
VID=200
DISABLED=no
BOOTPROTO=static
EOF

echo '192.168.200.2/28' > /etc/net/ifaces/enp7s1.200/ipv4address
echo 'default via 192.168.200.1' > /etc/net/ifaces/enp7s1.200/ipv4route
echo 'nameserver 192.168.100.2' > /etc/net/ifaces/enp7s1.200/resolv.conf

systemctl restart network

4.3 Настройка маршрутизации на HQ-RTR

sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/' /etc/net/sysctl.conf
systemctl restart network
Проверка: ip a — появление интерфейсов enp7s2.100/200/999

5. Настройка безопасного удалённого доступа (SSH)

Настроить SSH на порту 2024, ограничить доступ пользователем sshuser.

5.1 Настройка SSH на HQ-SRV и BR-SRV

echo "Authorized access only" > /etc/openssh/banner
echo -e "Port 2026\nMaxAuthTries 2\nAllowUsers sshuser\nBanner /etc/openssh/banner\n" >> /etc/openssh/sshd_config
systemctl restart sshd
ss -ltnp | grep sshd 

+HQ-SRV (ssh sshuser@127.0.0.1 -p 2026)
Проверка: ss -ltnp | grep 2026

6. Настройка IP-туннеля (GRE)

Организовать туннель между HQ-RTR и BR-RTR.

6.1 Настройка GRE на HQ-RTR

cat << EOF > /etc/net/ifaces/gre1/options
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.1.2
TUNREMOTE=172.16.2.2
TUNOPTIONS='ttl 64'

EOF

cat /etc/net/ifaces/gre1/options

echo "10.10.10.1/30" > /etc/net/ifaces/gre1/ipv4address

systemctl restart network

ip -br -c a
ping 10.10.10.2 -c 3
ping zz.ru -c 2
6.2 Настройка GRE на BR-RTR

cat << EOF > /etc/net/ifaces/gre1/options
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.2.2
TUNREMOTE=172.16.1.2
TUNTTL=64
TUNOPTIONS='ttl 64'
EOF

cat /etc/net/ifaces/gre1/options

echo "10.10.10.2/30" > /etc/net/ifaces/gre1/ipv4address

systemctl restart network
ip -br -c a
Проверка: ip a show gre1 — ping 10.10.10.2

7. Настройка динамической маршрутизации (OSPF)

Настроить OSPF через FRR для обмена маршрутами.

7.1 Установка FRR и настройка OSPF

apt-get update && apt-get install frr -y
sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons
systemctl enable --now frr

7.2 Конфигурация OSPF на HQ-RTR

sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons ; grep ospf /etc/frr/daemons

cat <<'EOF' > /etc/frr/frr.conf
interface gre
 no ip ospf passive
exit
!
interface gre1
 ip ospf area 0
 ip ospf authentication
 ip ospf authentication-key P@ssw0rd
 no ip ospf passive
exit
!
interface enp7s2.100
 ip ospf area 0
exit
!
interface enp7s2.200
 ip ospf area 0
exit
!
interface enp7s2.999
 ip ospf area 0
exit
!
router ospf
 passive-interface default
exit

EOF
systemctl restart frr
Проверка: vtysh -c "show ip ospf neighbor"

8. Настройка динамической трансляции адресов (NAT)

Настроить маскарадинг для доступа в интернет.

8.1 NAT на ISP

apt-get update && apt-get install nftables -y

cat << EOF > /etc/nftables/nftables.nft
#!/usr/sbin/nft -f
flush ruleset
table ip nat {
 chain postrouting {
 type nat hook postrouting priority srcnat;
 oifname "enp7s1"  masquerade
 }
}
EOF
cat /etc/nftables/nftables.nft 

systemctl enable --now nftables

8.2 NAT на HQ-RTR

cat << EOF > /etc/nftables/nftables.nft
#!/usr/sbin/nft -f
flush ruleset
table ip nat {
 chain postrouting {
 type nat hook postrouting priority srcnat
 oifname "enp7s1" masquerade
 }
}
EOF

systemctl enable --now nftables
Проверка: nft list ruleset

9. Настройка DHCP

Настроить DHCP-сервер на HQ-RTR для HQ-CLI.

9.1 Установка и настройка dnsmasq

sed -i 's/AUTO_LOCAL_RESOLVER=yes/AUTO_LOCAL_RESOLVER=no/' /etc/sysconfig/dnsmasq ; grep AUTO_LOCAL_RESOLVER /etc/sysconfig/dnsmasq

cat <<'EOF' > /etc/dnsmasq.conf
port=0
interface=enp7s2.200
listen-address=192.168.200.1
dhcp-authoritative
dhcp-range=interface:enp7s2.200,192.168.200.2,192.168.200.2,255.255.255.240,6h
dhcp-option=3,192.168.200.1
dhcp-option=6,192.168.100.2
leasefile-ro
EOF

systemctl enable --now frr dnsmasq ; ss -lun | grep 67

systemctl restart network
cat /etc/resolv.conf
ip r | grep ospf
Проверка: ss -lun | grep 67

10. Настройка DNS

Настроить BIND на HQ-SRV.

10.1 Установка BIND

apt-get update && apt-get install bind bind-utils -y
rndc-confgen -a -c /etc/bind/rndc.key

10.2 Конфигурация BIND

cat > /etc/bind/options.conf << 'EOF'
options {
 listen-on { 127.0.0.1; 192.168.100.2; };
 forwarders { 77.88.8.7; 77.88.8.3; };
 recursion yes;
 dnssec-validation no;
 directory "/etc/bind/zone";
};
zone "au-team.irpo" { type master; file "au-team.irpo"; };
zone "168.192.in-addr.arpa" { type master; file "168.192.in-addr.arpa"; };
EOF

mkdir -p /etc/bind/zone
cat > /etc/bind/zone/au-team.irpo << 'EOF'
$TTL 1D
@ IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (2025020600 12H 1H 1W 1H)
@ IN NS hq-srv.au-team.irpo.
hq-rtr IN A 192.168.100.1
hq-srv IN A 192.168.100.2
hq-cli IN A 192.168.200.2
br-rtr IN A 192.168.1.1
br-srv IN A 192.168.1.2
EOF

cat > /etc/bind/zone/168.192.in-addr.arpa << 'EOF'
$TTL 1D
@ IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (2025020600 12H 1H 1W 1H)
@ IN NS au-team.irpo.
1.100 IN PTR hq-rtr.au-team.irpo.
2.100 IN PTR hq-srv.au-team.irpo.
2.200 IN PTR hq-cli.au-team.irpo.
EOF

chown :named /etc/bind/zone/au-team.irpo /etc/bind/zone/168.192.in-addr.arpa
systemctl enable --now bind
service network restart
Проверка: nslookup hq-srv.au-team.irpo 127.0.0.1

11. Настройка часовых поясов (уже выполнена в п. 1.1)
 
МОДУЛЬ 2


Задание 1. Установка Яндекс Браузера на HQ-CLI

apt-get update && apt-get install yandex-browser-stable -y
Проверка:
yandex-browser-stable      #под обычным пользователем выполнять запуск
rpm -qa | grep yandex

2. Настройка NFS-сервера на HQ-SRV

HQ-SRV
Проверка установки пакета
rpm -qa | grep nfs

Запуск NFS-сервера
systemctl enable --now nfs-server
systemctl status nfs-server

Создание папки для расшаривания
mkdir -p /raid0/nfs
chmod 777 /raid0/nfs

Настройка экспорта (ВНИМАНИЕ: сеть HQ-CLI 192.168.200.0/28!)
cat >> /etc/exports << EOF
/raid0/nfs 192.168.200.0/28(rw,sync,no_subtree_check)
EOF

Применение настроек
exportfs -ra

Проверка
exportfs -v

HQ-CLI
Проверка установки пакета
rpm -qa | grep nfs

Создание точки монтирования
mkdir -p /mnt/nfs

Монтирование (IP HQ-SRV = 192.168.100.2)
mount -t nfs 192.168.100.2:/raid0/nfs /mnt/nfs

Проверка монтирования
df -h | grep nfs

Добавление в автозагрузку (/etc/fstab)
cat >> /etc/fstab << EOF
192.168.100.2:/raid0/nfs /mnt/nfs nfs defaults,_netdev 0 0
EOF

Проверка fstab
mount -a

Проверка
HQ-CLI 
touch /mnt/nfs/testfile
ls -la /mnt/nfs/
HQ-SRV 
ls -la /raid0/nfs/

3. Конфигурация файлового хранилища на HQ-SRV.

Установка пакета
apt-get update && apt-get install mdadm -y

Проверка доступных дисков
lsblk

Создание RAID 0 из двух дисков
mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc

Проверка статуса
cat /proc/mdstat
mdadm --detail /dev/md0

Форматирование в ext4
mkfs.ext4 /dev/md0

Сохранение конфигурации RAID
mkdir -p /etc/mdadm 2>/dev/null
echo "DEVICE partitions" > /etc/mdadm.conf
mdadm --detail --scan >> /etc/mdadm.conf

Проверка
cat /etc/mdadm.conf

Создание точки монтирования
mkdir -p /raid0

Добавление в /etc/fstab
echo -e "/dev/md0\t/raid0\text4\tdefaults\t0\t0" >> /etc/fstab

Проверка монтирования
mount -av

Проверка смонтированных файловых систем
df -h | grep raid0

Проверка дисков и RAID
lsblk
cat /proc/mdstat
mdadm --detail /dev/md0

4. Настройка статической трансляции портов на маршрутизаторах

HQ-RTR

В режиме конфигурации (config)
config terminal

Статическая трансляция портов для HQ-SRV
Проброс порта 80 (HTTP) на внешний IP 172.16.1.2:8080
ip nat source static tcp 192.168.100.2 80 172.16.1.2 8080

Проброс порта 2026 (SSH) на внешний IP 172.16.1.2:2026
ip nat source static tcp 192.168.100.2 2026 172.16.1.2 2026

Выход и сохранение
exit
write memory

BR-RTR

В режиме конфигурации (config)
config terminal

Статическая трансляция портов для BR-SRV
Проброс порта 8080 на внешний IP 172.16.2.2:8080
ip nat source static tcp 192.168.1.2 8080 172.16.2.2 8080

Проброс порта 2026 (SSH) на внешний IP 172.16.2.2:2026
ip nat source static tcp 192.168.1.2 2026 172.16.2.2 2026

Выход и сохранение
exit
write memory

ISP (проверка)
Установка curl для проверки
apt-get update && apt-get install curl -y

Проверка проброса порта 8080 на HQ-SRV (через HQ-RTR)
curl http://172.16.1.2:8080

Проверка проброса порта 8080 на BR-SRV (через BR-RTR)
curl http://172.16.2.2:8080

Проверка SSH до HQ-SRV (через HQ-RTR)
ssh sshuser@172.16.1.2 -p 2026

Проверка SSH до BR-SRV (через BR-RTR)
ssh sshuser@172.16.2.2 -p 2026

5. Настройте службу сетевого времени на базе сервиса chrony на маршрутизаторе ISP

ISP

Установка chrony
apt-get update && apt-get install chrony -y

Редактируем конфиг
nano /etc/chrony.conf

Внешние серверы времени
pool pool.ntp.org iburst

Локальный стратум 5 (если нет доступа к внешним серверам)
local stratum 5

Разрешаем доступ клиентам (ВАШИ СЕТИ)
allow 172.16.1.0/28      # сеть к HQ-RTR
allow 172.16.2.0/28      # сеть к BR-RTR
allow 192.168.100.0/27   # сеть HQ-SRV
allow 192.168.200.0/28   # сеть HQ-CLI
allow 192.168.1.0/28     # сеть BR-SRV

Запуск и проверка
systemctl enable --now chronyd
systemctl restart chronyd
chronyc tracking

На HQ-SRV и HQ-CLI

Установка chrony
apt-get update && apt-get install chrony -y

Редактируем конфиг
nano /etc/chrony.conf

На HQ-SRV и HQ-CLI
sed -i 's/^pool pool.ntp.org iburst/#&/' /etc/chrony.conf
echo "server 172.16.1.1 iburst" >> /etc/chrony.conf
systemctl restart chronyd

Добавить:
server 172.16.1.1 iburst

Запуск
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources

На BR-SRV:

Установка chrony
apt-get update && apt-get install chrony -y

Редактируем конфиг
nano /etc/chrony.conf

На BR-SRV
sed -i 's/^pool pool.ntp.org iburst/#&/' /etc/chrony.conf
echo "server 172.16.2.1 iburst" >> /etc/chrony.conf
systemctl restart chronyd

Добавить:
server 172.16.2.1 iburst
Запуск
systemctl enable --now chronyd
systemctl restart chronyd
chronyc sources

BR-RTR (EcoRouter)

В режиме конфигурации
configure terminal

Указываем NTP-сервер
ntp server 172.16.2.1

Сохраняем
write memory

Проверка
show ntp status
show ntp date


ЛИТЕРАТУРА:
https://github.com/stepanovs2005/Demo2026
https://github.com/Evgeniy64Bits/demo-exam-2026-realize
