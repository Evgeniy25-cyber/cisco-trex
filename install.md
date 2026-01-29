Установка на Bare-metal ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-85-generic x86_64)
=========================================================================
Проверка релизов и обновления
cat /etc/*-release
lsb_release -a
cat /etc/debian_version
sudo apt upgrade
sudo apt full-upgrade
sudo apt autoremove
sudo apt autoclean
=========================================================================
подходят репозитории установка в каталоге
nano /etc/apt/sources.list
deb http://deb.debian.org/debian bookworm-updates main
deb http://deb.debian.org/debian-security bookworm-security main
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
#
deb http://deb.debian.org/debian-security/ bookworm-security main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main contrib non-free non-free-firmware
#
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
=========================================================================
Для работы сетевых карт intel необходимо выполнить 
/etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
===========================================================
или 
GRUB_CMDLINE_LINUX="iommu=pt, intel_iommu=on"
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
далее обновить и перезагрузить
sudo  update-grub
reboot
==========================================================================
настройка менеджмент сети на ubuntu 24.04
-------------------------------------------------------------------------
пример:
nano /etc/netplan/50-cloud-init.yaml
-------------------------------------------------------------------------
network:
  version: 2
  ethernets:
    ens13f0:
      addresses:
      - "10.20.51.149/24"
      nameservers:
        addresses:
        - 8.8.8.8
        search: []
      routes:
      - to: "default"
        via: "10.20.51.254"
------------------------------------------------------------------------
После обязательно применить команду
sudo netplan apply
==========================================================================
1. Применяем изменения для grub Debian 12-13
update-grub
2. Перезагружаемся
# find / -name update-grub
/usr/sbin/update-grub
Ran it, just to get the next error message.
# /usr/sbin/update-grub
/usr/sbin/update-grub: 4: exec: grub-mkconfig: not found
Searched for grub-mkconfig and found it under /usr/sbin/grub-mkconfig. Then it came to me, let's see how the update-grub script looks like?
#cat /usr/sbin/update-grub |grep grub-mkconfig
exec grub-mkconfig -o /boot/grub/grub.cfg "$@"
Modified /usr/sbin/update-grub in order to call grub-mkconfig by it's explicit path ...
exec /usr/sbin/grub-mkconfig -o /boot/grub/grub.cfg "$@"
==========================================================================================================================
Скачиваем cisco-trex в каталог /opt/
wget --no-check-certificate https://trex-tgn.cisco.com/trex/release/v3.07.tar.gz
разархивируем его 
tar xfvz v3.06.tar.gz
===========================================================
Установка dpdk 
apt install dpdk
удаление  dpdk
sudo apt-get -y autoremove --purge dpdk
--------------------------------------------------------------------------
Для чего данные команды будет пример при начале работы с trex
dpdk-hugepages.py --setup 2G
echo 'vm.nr_hugepages = 2048' > /etc/sysctl.d/hugepages.conf
============================================================
переименовываем v3.06 в папку trex
mv v3.06 trex
==========================================================
так как trex  использует python 3.11 а данные версии ОП не поддерживаться в работе то необходимо установить
модуль venv для создания виртуальных окружений в Python, используя версию Python 3.11
add-apt-repository ppa:gwibber-daily/ppa
apt-get install software-properties-common
add-apt-repository ppa:deadsnakes/ppa
apt install python3.11-venv
python3.11 -m venv venv
apt-get update
apt-get install python3.11
source ./venv/bin/activate
venv/bin/python venv/bin/pip install cffi
модуль  venv нельзя установить на ubuntu-25.04 не поддерживается 
======================================================
После этого надо выполнить проверочные команды
ls /usr/bin/python*
python3.13 --version
========================================================
Работа с cisco-trex запуск trex и консоли и скриптов осуществляется только с виртуального каталога
пример:
root@cisco-trex:~# source ./venv/bin/activate
(venv) root@cisco-trex:~#
(venv) root@cisco-trex:~# cd /opt/trex/
(venv) root@cisco-trex:/opt/trex#
========================================================
Запуск  cisco-trex осуществляется в  режиме stateful
пример:
(venv) root@cisco-trex:/opt/trex#./t-rex-64 -i --astf --tso-disable --lro-disable --iom 0
Запуск службы cisco-trex осуществляется в  режиме stateless
(venv) root@cisco-trex:/opt/trex#./t-rex-64 --no-ofed-check --no-scapy-server -i
Пример входа в консоль cisco-trex
(venv) root@cisco-trex:/opt/trex# ./trex-console -f
в коносли можно выполнить проверочную команду
capture monitor start --tx 0 1  --rx 0 1
========================================================
В случаи с Debian 12-13
придётся удалить данную библиотеку для запуска trex
rm v3.06/so/x86_64/libstdc++.so.6
========================================================
Самое важное если установленные сетевые  Ethernet Controller E810-C for QSFP
первое это сверка версий DPDK и совместимость версии сетевого драйвера данной сетевой карты 
далее надо искать драйвера для данного контроллера для обновления часть можно скачать на GitHub разных версий  так как официальный сайт intel не работает в РФ (можно через vpn)
нужно 3 драйвера
1.ice-2.3.10.tar.gz
2.ice_comms-30.1.15.pkg
3.iavf-10.16.3.tar.gz
ice-по ссылке https://github.com/intel/ethernet-linux-ice?ysclid=mh4yp45ccl498297844
iavf-https://github.com/intel/ethernet-linux-iavf
ice_comms-c официального сайта intel.
----------------------------------------------------------------------------------
Порядок установки данных драйверов описан в файлах README
также после установки надо установить tool и  обновить
apt install live-tools
(venv) root@cisco-trex:/opt/trex# update-initramfs -u
===================================================================================
Проверочные команды  при работе Ethernet Controller E810-C for QSFP
dmesg | grep "\<ice\>"
modinfo ice
lshw -class network -businfo
ethtool -i ens22f0np0
lspci -vvv -s 0000:98:00.0
lspci -vv -s 0000:98:00.0 | grep -i Serial
-------------------------------------------------------------------------------------
перезагрузка драйверов
modprobe -r ice; sudo modprobe ice
modinfo ice | grep firmware
modprobe vfio-pci
-------------------------------------------------------------------------------------
поиск модулей установки сетевой карты
find /lib/modules -name ice.ko
modinfo ice | grep firmware
lspci -vvv | grep E810
=======================================================================================
При обновлении ядра придётся  выполнить следующий сценарий для работы Ethernet Controller E810-C for QSFP
перенести разархивированный ice  в каталог /usr/src/ice-2.3.10
в  папку ice-2.3.10 положить файл dkms.conf заранее созданный в блокноте  данный файлик с таким называнием и расширением и расширением conf dkms.conf
содержание файла dkms:
PACKAGE_NAME="ice"
PACKAGE_VERSION="2.3.10"  # Update to match!
BUILT_MODULE_NAME[0]="ice"
BUILT_MODULE_LOCATION[0]="src/"
DEST_MODULE_LOCATION[0]="/kernel/drivers/net/ethernet/intel/ice/"
AUTOINSTALL="yes"
MAKE[0]="cd src; make -j"
--------------------------------------------------------------------------------------------
пример установки  dkms  и запуск dkms
apt install dkms
dkms add ice/2.3.10
dkms build ice/2.3.10
dkms install ice/2.3.10
запуска. из каталога 
root@cisco-trex:/usr/src/ice-2.3.10#dkms add ice/2.3.10
root@cisco-trex:/usr/src/ice-2.3.10#dkms build ice/2.3.10
root@cisco-trex:/usr/src/ice-2.3.10#dkms install ice/2.3.10
----------------------------------------------------------------------------------------------
Протестировал такой сценарий на Debian 12-13 
не заработало пришлось заново подирать драйвера и устанавливать 
-----------------------------------------------------------------------------------------------
Обязательно выполнить обновления после установки dkms 
пример:
update-initramfs -u
==================================================================================================
Раздел  работа с настройками сетевых карт intel  Ethernet Controller X710 for 10GbE SFP+ и Ethernet Controller E810-C for QSFP  в режиме SR-IOV (данные карты работают только в этом режиме)
--------------------------------------------------------------------------------------------------------------------------------------------------------
проверка поддержки кол-во виртуальных интерфейсов  данными сетевыми картами 
Пример для  Ethernet Controller X710:
------------------------------------------------------------------------
cat /sys/class/net/ens1f0np0/device/sriov_totalvfs
32
всего 32 виртуальных интерфейса на один порт
----------------------------------------------------------------------------------
Пример для 
Ethernet Controller E810-C for QSFP
cat /sys/class/net/ens22f0np0/device/sriov_totalvfs
128 
всего 128 виртуальных интерфейса на один порт
========================================================================================
для создание простых настроек с  использованием  одной подсети и одного виртуального интерфейса  необходимо включения  режима SR-IOV
пример команды 
echo 1 > /sys/class/net/ens22f1np1/device/sriov_numvfs
========================================================================================================================
Пример для  Ethernet Controller X710 и E810-C for QSFP  одинаковы поэтому приведу пример для E810-C for QSFP для создание 4 виртуальных интерфейсов для каждого порта
1. Посмотрим сколько у нас и каких в dpdk
(venv) root@cisco-trex:/opt/trex# ./dpdk_setup_ports.py -t
(venv) root@cisco-trex:/opt/trex# ./dpdk_setup_ports.py -t
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| ID | NUMA |   PCI   |        MAC        |                  Name                   | Driver |  Linux IF  |  Active  |
+====+======+=========+===================+=========================================+========+============+==========+
| 0  | 0    | 17:00.0 | 00:0a:cd:47:ef:a5 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 1  | 0    | 17:00.1 | 00:0a:cd:47:ef:a6 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 2  | 0    | 17:00.2 | 00:0a:cd:47:ef:a7 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 3  | 0    | 17:00.3 | 00:0a:cd:47:ef:a8 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 4  | 0    | 32:00.0 | 00:0a:cd:47:ef:9d | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 5  | 0    | 32:00.1 | 00:0a:cd:47:ef:9e | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 6  | 0    | 32:00.2 | 00:0a:cd:47:ef:9f | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 7  | 0    | 32:00.3 | 00:0a:cd:47:ef:a0 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 8  | 0    | 4b:00.0 | 6c:b3:11:99:92:78 | I350 Gigabit Network Connection         | igb    | ens13f0    | *Active* |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 9  | 0    | 4b:00.1 | 6c:b3:11:99:92:79 | I350 Gigabit Network Connection         | igb    | ens13f1    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 10 | 0    | 4b:00.2 | 6c:b3:11:99:92:7a | I350 Gigabit Network Connection         | igb    | ens13f2    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 11 | 0    | 4b:00.3 | 6c:b3:11:99:92:7b | I350 Gigabit Network Connection         | igb    | ens13f3    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 12 | 1    | 98:00.0 | 6c:fe:54:41:7c:90 | Ethernet Controller E810-C for QSFP     | ice    | ens22f0np0 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 13 | 1    | 98:00.1 | 6c:fe:54:41:7c:91 | Ethernet Controller E810-C for QSFP     | ice    | ens22f1np1 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
Обращаем на колонку Driver,Linux IF, Active 
в режиме Active находиться наша порт в  менеджмент сети для управления trex
-------------------------------------------------------------------------------------------------------------------------------------------
настроим следующие порты trex Ethernet Controller E810-C for QSFP 
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 12 | 1    | 98:00.0 | 6c:fe:54:41:7c:90 | Ethernet Controller E810-C for QSFP     | ice    | ens22f0np0 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 13 | 1    | 98:00.1 | 6c:fe:54:41:7c:91 | Ethernet Controller E810-C for QSFP     | ice    | ens22f1np1 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
на каждом из них создаём  4 виртуальных интерфейса 
на 12 включаем 
echo 4 > /sys/class/net/ens22f0np0/device/sriov_numvfs
-----------------------------------------------------------------------------------------------------------------------
проверяем пример 
(venv) root@cisco-trex:/opt/trex# ./dpdk_setup_ports.py -t
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| ID | NUMA |   PCI   |        MAC        |                  Name                   | Driver |  Linux IF  |  Active  |
+====+======+=========+===================+=========================================+========+============+==========+
| 0  | 0    | 17:00.0 | 00:0a:cd:47:ef:a5 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 1  | 0    | 17:00.1 | 00:0a:cd:47:ef:a6 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 2  | 0    | 17:00.2 | 00:0a:cd:47:ef:a7 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 3  | 0    | 17:00.3 | 00:0a:cd:47:ef:a8 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 4  | 0    | 32:00.0 | 00:0a:cd:47:ef:9d | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 5  | 0    | 32:00.1 | 00:0a:cd:47:ef:9e | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 6  | 0    | 32:00.2 | 00:0a:cd:47:ef:9f | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 7  | 0    | 32:00.3 | 00:0a:cd:47:ef:a0 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 8  | 0    | 4b:00.0 | 6c:b3:11:99:92:78 | I350 Gigabit Network Connection         | igb    | ens13f0    | *Active* |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 9  | 0    | 4b:00.1 | 6c:b3:11:99:92:79 | I350 Gigabit Network Connection         | igb    | ens13f1    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 10 | 0    | 4b:00.2 | 6c:b3:11:99:92:7a | I350 Gigabit Network Connection         | igb    | ens13f2    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 11 | 0    | 4b:00.3 | 6c:b3:11:99:92:7b | I350 Gigabit Network Connection         | igb    | ens13f3    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 12 | 1    | 98:00.0 | 6c:fe:54:41:7c:90 | Ethernet Controller E810-C for QSFP     | ice    | ens22f0np0 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 13 | 1    | 98:00.1 | 6c:fe:54:41:7c:91 | Ethernet Controller E810-C for QSFP     | ice    | ens22f1np1 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 14 | 1    | 98:01.0 | 36:d3:f2:8a:20:5f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 15 | 1    | 98:01.1 | ba:74:e4:84:cc:dd | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 16 | 1    | 98:01.2 | 7a:28:51:7a:50:2f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 17 | 1    | 98:01.3 | 2a:ce:40:94:9f:a3 | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
Видим что появилось 4 с 14-17  новых сетевых виртуальных порта с Driver iavf принадлежащие одному физическому порту ens22f0np0  префиксы v0-4  ens22f0v0-3
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Тоже самое выполним для 13 интерфейса ens22f1np1
echo 4 > /sys/class/net/ens22f1np1/device/sriov_numvfs
Пример:
(venv) root@cisco-trex:/opt/trex# ./dpdk_setup_ports.py -t
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| ID | NUMA |   PCI   |        MAC        |                  Name                   | Driver |  Linux IF  |  Active  |
+====+======+=========+===================+=========================================+========+============+==========+
| 0  | 0    | 17:00.0 | 00:0a:cd:47:ef:a5 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 1  | 0    | 17:00.1 | 00:0a:cd:47:ef:a6 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 2  | 0    | 17:00.2 | 00:0a:cd:47:ef:a7 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 3  | 0    | 17:00.3 | 00:0a:cd:47:ef:a8 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 4  | 0    | 32:00.0 | 00:0a:cd:47:ef:9d | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 5  | 0    | 32:00.1 | 00:0a:cd:47:ef:9e | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 6  | 0    | 32:00.2 | 00:0a:cd:47:ef:9f | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 7  | 0    | 32:00.3 | 00:0a:cd:47:ef:a0 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 8  | 0    | 4b:00.0 | 6c:b3:11:99:92:78 | I350 Gigabit Network Connection         | igb    | ens13f0    | *Active* |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 9  | 0    | 4b:00.1 | 6c:b3:11:99:92:79 | I350 Gigabit Network Connection         | igb    | ens13f1    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 10 | 0    | 4b:00.2 | 6c:b3:11:99:92:7a | I350 Gigabit Network Connection         | igb    | ens13f2    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 11 | 0    | 4b:00.3 | 6c:b3:11:99:92:7b | I350 Gigabit Network Connection         | igb    | ens13f3    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 12 | 1    | 98:00.0 | 6c:fe:54:41:7c:90 | Ethernet Controller E810-C for QSFP     | ice    | ens22f0np0 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 13 | 1    | 98:00.1 | 6c:fe:54:41:7c:91 | Ethernet Controller E810-C for QSFP     | ice    | ens22f1np1 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 14 | 1    | 98:01.0 | 36:d3:f2:8a:20:5f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 15 | 1    | 98:01.1 | ba:74:e4:84:cc:dd | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 16 | 1    | 98:01.2 | 7a:28:51:7a:50:2f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 17 | 1    | 98:01.3 | 2a:ce:40:94:9f:a3 | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 18 | 1    | 98:11.0 | ae:3c:ca:14:17:36 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 19 | 1    | 98:11.1 | 7a:25:11:01:51:74 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 20 | 1    | 98:11.2 | 0a:1a:be:b8:f7:33 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 21 | 1    | 98:11.3 | fa:70:43:32:39:38 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
--------------------------------------------------------------------------------------------------------------------------------
Видим что появилось 4 с 18-21  новых сетевых виртуальных порта с Driver iavf принадлежащие одному физическому порту ens22f1np1  префиксы v0-4  ens22f1v0
----------------------------------------------------------------------------------------------------------------------------------------------
=========================================================================================================================================================
Настроим данные порты в  на L3  конфигурационном файле trex
Пример 
(venv) root@cisco-trex:/opt/trex# ./dpdk_setup_ports.py -i
By default, IP based configuration file will be created. Do you want to use MAC based config? (y/N)
(venv) root@cisco-trex:/opt/trex# ./dpdk_setup_ports.py -i
By default, IP based configuration file will be created. Do you want to use MAC based config? (y/N)n -----------выбираем N (no)
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| ID | NUMA |   PCI   |        MAC        |                  Name                   | Driver |  Linux IF  |  Active  |
+====+======+=========+===================+=========================================+========+============+==========+
| 0  | 0    | 17:00.0 | 00:0a:cd:47:ef:a5 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 1  | 0    | 17:00.1 | 00:0a:cd:47:ef:a6 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 2  | 0    | 17:00.2 | 00:0a:cd:47:ef:a7 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 3  | 0    | 17:00.3 | 00:0a:cd:47:ef:a8 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens1f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 4  | 0    | 32:00.0 | 00:0a:cd:47:ef:9d | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f0np0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 5  | 0    | 32:00.1 | 00:0a:cd:47:ef:9e | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f1np1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 6  | 0    | 32:00.2 | 00:0a:cd:47:ef:9f | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f2np2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 7  | 0    | 32:00.3 | 00:0a:cd:47:ef:a0 | Ethernet Controller X710 for 10GbE SFP+ | i40e   | ens7f3np3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 8  | 0    | 4b:00.0 | 6c:b3:11:99:92:78 | I350 Gigabit Network Connection         | igb    | ens13f0    | *Active* |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 9  | 0    | 4b:00.1 | 6c:b3:11:99:92:79 | I350 Gigabit Network Connection         | igb    | ens13f1    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 10 | 0    | 4b:00.2 | 6c:b3:11:99:92:7a | I350 Gigabit Network Connection         | igb    | ens13f2    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 11 | 0    | 4b:00.3 | 6c:b3:11:99:92:7b | I350 Gigabit Network Connection         | igb    | ens13f3    |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 12 | 1    | 98:00.0 | 6c:fe:54:41:7c:90 | Ethernet Controller E810-C for QSFP     | ice    | ens22f0np0 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 13 | 1    | 98:00.1 | 6c:fe:54:41:7c:91 | Ethernet Controller E810-C for QSFP     | ice    | ens22f1np1 |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 14 | 1    | 98:01.0 | 36:d3:f2:8a:20:5f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 15 | 1    | 98:01.1 | ba:74:e4:84:cc:dd | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 16 | 1    | 98:01.2 | 7a:28:51:7a:50:2f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 17 | 1    | 98:01.3 | 2a:ce:40:94:9f:a3 | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 18 | 1    | 98:11.0 | ae:3c:ca:14:17:36 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 19 | 1    | 98:11.1 | 7a:25:11:01:51:74 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 20 | 1    | 98:11.2 | 0a:1a:be:b8:f7:33 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 21 | 1    | 98:11.3 | fa:70:43:32:39:38 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
Please choose an even number of interfaces from the list above, either by ID, PCI or Linux IF
Stateful will use order of interfaces: Client1 Server1 Client2 Server2 etc. for flows.
Stateless can be in any order.
For performance, try to choose each pair of interfaces to be on the same NUMA.
Enter list of interfaces separated by space (for example: 1 3) :  в данной строчке выбираем пары интерфейсов 
следующим образом пример:
Please choose an even number of interfaces from the list above, either by ID, PCI or Linux IF
Stateful will use order of interfaces: Client1 Server1 Client2 Server2 etc. for flows.
Stateless can be in any order.
For performance, try to choose each pair of interfaces to be on the same NUMA.
Enter list of interfaces separated by space (for example: 1 3) : 14 18 15 19 16 20 17 21 ---(у нас получается четные с четными и  нечетные с нечетными)
далле просто нажимем n (no)
пример Stateless can be in any order.
For performance, try to choose each pair of interfaces to be on the same NUMA.
Enter list of interfaces separated by space (for example: 1 3) : 14 18 15 19 16 20 17 21

For interface 14, assuming loopback to its dual interface 18.
Putting IP 1.1.1.1, default gw 2.2.2.2 Change it?(y/N).n
For interface 18, assuming loopback to its dual interface 14.
Putting IP 2.2.2.2, default gw 1.1.1.1 Change it?(y/N).n
For interface 15, assuming loopback to its dual interface 19.
Putting IP 3.3.3.3, default gw 4.4.4.4 Change it?(y/N).n
For interface 19, assuming loopback to its dual interface 15.
Putting IP 4.4.4.4, default gw 3.3.3.3 Change it?(y/N).n
For interface 16, assuming loopback to its dual interface 20.
Putting IP 5.5.5.5, default gw 6.6.6.6 Change it?(y/N).n
For interface 20, assuming loopback to its dual interface 16.
Putting IP 6.6.6.6, default gw 5.5.5.5 Change it?(y/N).n
For interface 17, assuming loopback to its dual interface 21.
Putting IP 7.7.7.7, default gw 8.8.8.8 Change it?(y/N).n
For interface 21, assuming loopback to its dual interface 17.
Putting IP 8.8.8.8, default gw 7.7.7.7 Change it?(y/N).n
Print preview of generated config? (Y/n)y
### Config file generated by dpdk_setup_ports.py ###

- version: 2
  interfaces: ['98:01.0', '98:11.0', '98:01.1', '98:11.1', '98:01.2', '98:11.2', '98:01.3', '98:11.3']
  port_info:
      - ip: 1.1.1.1
        default_gw: 2.2.2.2
      - ip: 2.2.2.2
        default_gw: 1.1.1.1

      - ip: 3.3.3.3
        default_gw: 4.4.4.4
      - ip: 4.4.4.4
        default_gw: 3.3.3.3

      - ip: 5.5.5.5
        default_gw: 6.6.6.6
      - ip: 6.6.6.6
        default_gw: 5.5.5.5

      - ip: 7.7.7.7
        default_gw: 8.8.8.8
      - ip: 8.8.8.8
        default_gw: 7.7.7.7

  platform:
      master_thread_id: 0
      latency_thread_id: 1
      dual_if:
        - socket: 1
          threads: [32,33,34,35,36,37,38,39]

        - socket: 1
          threads: [40,41,42,43,44,45,46,47]

        - socket: 1
          threads: [48,49,50,51,52,53,54,55]

        - socket: 1
          threads: [56,57,58,59,60,61,62,63]


Save the config to file? (Y/n)y
Default filename is /etc/trex_cfg.yaml
Press ENTER to confirm or enter new file:
File /etc/trex_cfg.yaml already exist, overwrite? (y/N)y (здесь выбираем да)
Saved to /etc/trex_cfg.yaml.

(venv) root@cisco-trex:/opt/trex#
----------------------------------------------------------------------------------------------------
в отличии от прежних версий сразу  бросается в глаза атематической распределение  аппаратных  ресурсов сервера для данного режима работы
 platform:
      master_thread_id: 0
      latency_thread_id: 1
      dual_if:
        - socket: 1
          threads: [32,33,34,35,36,37,38,39]

        - socket: 1
          threads: [40,41,42,43,44,45,46,47]

        - socket: 1
          threads: [48,49,50,51,52,53,54,55]

        - socket: 1
          threads: [56,57,58,59,60,61,62,63]
----------------------------------------------------------------------------------------------------
Но данный конфигурационный файл не будут работать поэтому редактируем данный файл в файле trex_cfg.yaml
nano /etc/trex_cfg.yaml
-----------------------------------------------------------
### Config file generated by dpdk_setup_ports.py ###

- version: 2
  interfaces: ['98:01.0', '98:11.0', '98:01.1', '98:11.1', '98:01.2', '98:11.2', '98:01.3', '98:11.3']
  c: 8
  port_info:
     - default_gw      : 192.168.101.1
       ip              : 192.168.101.2
       vlan: 101
     - default_gw      : 192.168.102.1
       ip              : 192.168.102.2
       vlan: 102
     - default_gw      : 192.168.103.1
       ip              : 192.168.103.2
       vlan: 103
     - default_gw      : 192.168.104.1
       ip              : 192.168.104.2
       vlan: 104
     - default_gw      : 192.168.105.1
       ip              : 192.168.105.2
       vlan: 105
     - default_gw      : 192.168.106.1
       ip              : 192.168.106.2
       vlan: 106
     - default_gw      : 192.168.107.1
       ip              : 192.168.107.2
       vlan: 107
     - default_gw      : 192.168.108.1
       ip              : 192.168.108.2
       vlan: 108

  platform:
      master_thread_id: 0
      latency_thread_id: 1
      dual_if:
        - socket: 1
          threads: [32,33,34,35,36,37,38,39]

        - socket: 1
          threads: [40,41,42,43,44,45,46,47]

        - socket: 1
          threads: [48,49,50,51,52,53,54,55]

        - socket: 1
          threads: [56,57,58,59,60,61,62,63]

-----------------------------------------------------------------------------------------------
Это итоговый готовый файл 
Важно в данном файле мы добавляем кол-во ядер процессора это значение c: 8 (без указания данного параметра при генерации трафика вы не достигнете генерации даже 5gb/c любого трафика)
Также мы  создали IEEE 802.1Q vlan (— протокол транкинга)
Сразу уточню что со стороны сетевые настройки выполнены и настроены пример сетевых настроек коммутаторов  будет позже 
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Запустим trex   с начала в режиме stateful а затем stateless 
пример режим stateful
(venv) root@cisco-trex:/opt/trex# ./t-rex-64 -i --astf --tso-disable --lro-disable --iom 0
Trying to bind to vfio-pci ...
/home/root1/venv/bin/python3 dpdk_nic_bind.py --bind=vfio-pci 0000:98:01.0 0000:98:11.0 0000:98:01.1 0000:98:11.1 0000:98:01.2 0000:98:11.2 0000:98:01.3 0000:98:11.3
The ports are bound/configured.
Starting  TRex v3.06 please wait  ...
EAL: FATAL: Cannot get hugepage information.
EAL: Cannot get hugepage information.
 You might need to run ./trex-cfg  once
EAL: Error - exiting with code: 1
  Cause: Invalid EAL arguments
(venv) root@cisco-trex:/opt/trex#
----------------------------------------------------------------------------------------------------------------
выдал ошибкуEAL: Error - exiting with code: 1 и тут самое главное включить поддержку больших страниц в DPDK
эти  команды были выше при установки dpdk
включаем данную команду вторую не беру и запустим опять trex
dpdk-hugepages.py --setup 2G
---------------------------------------------------------------------------------------------------------------
Пример выполнения запуска  trex режиме stateful
(venv) root@cisco-trex:/opt/trex# dpdk-hugepages.py --setup 2G

(venv) root@cisco-trex:/opt/trex# ./t-rex-64 -i --astf --tso-disable --lro-disable --iom 0
The ports are bound/configured.
Starting  TRex v3.06 please wait  ...
 set driver name net_iavf
 driver capability  : TCP_UDP_OFFLOAD  TSO  SLRO
Warning TSO is supported and asked to be disabled by user
Warning SLRO is supported and asked to be disabled by user
 set dpdk queues mode to MULTI_QUE
 Number of ports found: 8
zmq publisher at: tcp://*:4500
 wait 1 sec .
port : 0
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 1
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 2
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 3
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 4
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 5
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 6
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
port : 7
------------
link         :  link : Link Up - speed 100000 Mbps - full-duplex
promiscuous  : 0
 number of ports         : 8
 max cores for 2 ports   : 8
 tx queues per port      : 10
 -------------------------------
RX core uses TX queue number 65535 on all ports
 core, c-port, c-queue, s-port, s-queue, lat-queue
 ------------------------------------------
 1        0      0       1       0      0
 2        2      0       3       0      0
 3        4      0       5       0      0
 4        6      0       7       0      0
 5        0      1       1       1    255
 6        2      1       3       1    255
 7        4      1       5       1    255
 8        6      1       7       1    255
 9        0      2       1       2    255
 10        2      2       3       2    255
 11        4      2       5       2    255
 12        6      2       7       2    255
 13        0      3       1       3    255
 14        2      3       3       3    255
 15        4      3       5       3    255
 16        6      3       7       3    255
 17        0      4       1       4    255
 18        2      4       3       4    255
 19        4      4       5       4    255
 20        6      4       7       4    255
 21        0      5       1       5    255
 22        2      5       3       5    255
 23        4      5       5       5    255
 24        6      5       7       5    255
 25        0      6       1       6    255
 26        2      6       3       6    255
 27        4      6       5       6    255
 28        6      6       7       6    255
 29        0      7       1       7    255
 30        2      7       3       7    255
 31        4      7       5       7    255
 32        6      7       7       7    255
 -------------------------------
---------------------------------------------------------------------------------------------------------------------------
Видим запуск trex прошел успешно так как сетевые порты настроены на сетевом оборудовании и включены по этому порты сразу в статусе
link         :  link : Link Up - speed 100000 Mbps - full-duplex
------------------------------------------------------------------------------------------------------------------------------------
Важно при работе не с QSFP трансформами к примеру FIBO  FT-Q100-SR-BD (SN-FU024071xxxxxxx) пришлось еще перепрошить (благо был программатор и FIBO выслал новые прошивки для них)
1.Коммутацию необходимо сделать на многогодовых оптических патчкордах).
2.Отклчить  функцию Forwarding Error Correction (FEC), которая позволяет восстанавливать принимаемые данные при наличии потерь пакетов трафика на используемых каналах передачи данных
 со стороны сетевого оборудования и на trex
Если вы используете DAC то не надо 
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
пример отключения на ubuntu
ethtool --set-fec ens22f0np0 encoding off
ethtool --set-fec ens22f1np1 encoding off
-----------------------------------------------------------------
после можно проверить что данная функция выключена и со стороны коммутаторов и ubuntu следующей командой 
dmesg | grep "\<ice\>"
--------------------------------------------------------------------------------------------------------------------------------------------------------
проверочные мероприятия сетевой связности   со стороны trex  проверим в консоли trex 
для этого можно запустить новый терминал и запустить коносль так как мы не устанавливали client,
то приважу пример локального подключения к консоли 
(venv) root@cisco-trex:/opt/trex# ./trex-console -f
----------------------------------------------------------------------------------------------------------
(venv) root@cisco-trex:/opt/trex# ./trex-console -f

Using 'python3' as Python interpeter


Connecting to RPC server on localhost:4501                   [SUCCESS]


Connecting to publisher server on localhost:4500             [SUCCESS]

[SUCCESS]


Force acquiring ports [0, 1, 2, 3, 4, 5, 6, 7]:              [SUCCESS]


Server Info:

Server version:   v3.06 @ ASTF
Server mode:      Advanced Stateful
Server CPU:       32 x Intel(R) Xeon(R) Gold 6338 CPU @ 2.00GHz
Ports count:      8 x 100.0Gbps @ Ethernet Adaptive Virtual Function

-=TRex Console v3.0=-

Type 'help' or '?' for supported actions

trex>
---------------------------------------------------------------------------------------------------------------------------
Теперь проверочные команды по сетевой настройки и доступности в косоли
Пример просмотрим настройки портов 
trex>portattr
Port Status

     port       |          0           |          1           |          2           |          3
----------------+----------------------+----------------------+----------------------+---------------------
driver          |       net_iavf       |       net_iavf       |       net_iavf       |       net_iavf
description     |  Ethernet Adaptive   |  Ethernet Adaptive   |  Ethernet Adaptive   |  Ethernet Adaptive
link status     |          UP          |          UP          |          UP          |          UP
link speed      |       100 Gb/s       |       100 Gb/s       |       100 Gb/s       |       100 Gb/s
port status     |         IDLE         |         IDLE         |         IDLE         |         IDLE
promiscuous     |         off          |         off          |         off          |         off
multicast       |         off          |         off          |         off          |         off
flow ctrl       |         N/A          |         N/A          |         N/A          |         N/A
vxlan fs        |          -           |          -           |          -           |          -
--              |                      |                      |                      |
layer mode      |         IPv4         |         IPv4         |         IPv4         |         IPv4
src IPv4        |    192.168.101.2     |    192.168.102.2     |    192.168.103.2     |    192.168.104.2
IPv6            |         off          |         off          |         off          |         off
src MAC         |  36:d3:f2:8a:20:5f   |  ae:3c:ca:14:17:36   |  ba:74:e4:84:cc:dd   |  7a:25:11:01:51:74
---             |                      |                      |                      |
Destination     |    192.168.101.1     |    192.168.102.1     |    192.168.103.1     |    192.168.104.1
ARP Resolution  |  90:54:b7:79:a4:83   |  90:54:b7:79:ac:83   |  90:54:b7:79:a4:83   |  90:54:b7:79:ac:83
----            |                      |                      |                      |
VLAN            |         101          |         102          |         103          |         104
-----           |                      |                      |                      |
PCI Address     |     0000:98:01.0     |     0000:98:11.0     |     0000:98:01.1     |     0000:98:11.1
NUMA Node       |          1           |          1           |          1           |          1
RX Filter Mode  |    hardware match    |    hardware match    |    hardware match    |    hardware match
RX Queueing     |         off          |         off          |         off          |         off
Grat ARP        |  every 120 seconds   |  every 120 seconds   |  every 120 seconds   |  every 120 seconds
------          |                      |                      |                      |

Do you want to see more ports? [y/N]: y
Port Status

     port       |          4           |          5           |          6           |          7
----------------+----------------------+----------------------+----------------------+---------------------
driver          |       net_iavf       |       net_iavf       |       net_iavf       |       net_iavf
description     |  Ethernet Adaptive   |  Ethernet Adaptive   |  Ethernet Adaptive   |  Ethernet Adaptive
link status     |          UP          |          UP          |          UP          |          UP
link speed      |       100 Gb/s       |       100 Gb/s       |       100 Gb/s       |       100 Gb/s
port status     |         IDLE         |         IDLE         |         IDLE         |         IDLE
promiscuous     |         off          |         off          |         off          |         off
multicast       |         off          |         off          |         off          |         off
flow ctrl       |         N/A          |         N/A          |         N/A          |         N/A
vxlan fs        |          -           |          -           |          -           |          -
--              |                      |                      |                      |
layer mode      |         IPv4         |         IPv4         |         IPv4         |         IPv4
src IPv4        |    192.168.105.2     |    192.168.106.2     |    192.168.107.2     |    192.168.108.2
IPv6            |         off          |         off          |         off          |         off
src MAC         |  7a:28:51:7a:50:2f   |  0a:1a:be:b8:f7:33   |  2a:ce:40:94:9f:a3   |  fa:70:43:32:39:38
---             |                      |                      |                      |
Destination     |    192.168.105.1     |    192.168.106.1     |    192.168.107.1     |    192.168.108.1
ARP Resolution  |  90:54:b7:79:a4:83   |  90:54:b7:79:ac:83   |  90:54:b7:79:a4:83   |  90:54:b7:79:ac:83
----            |                      |                      |                      |
VLAN            |         105          |         106          |         107          |         108
-----           |                      |                      |                      |
PCI Address     |     0000:98:01.2     |     0000:98:11.2     |     0000:98:01.3     |     0000:98:11.3
NUMA Node       |          1           |          1           |          1           |          1
RX Filter Mode  |    hardware match    |    hardware match    |    hardware match    |    hardware match
RX Queueing     |         off          |         off          |         off          |         off
Grat ARP        |  every 120 seconds   |  every 120 seconds   |  every 120 seconds   |  every 120 seconds
------          |                      |                      |                      |

trex>
---------------------------------------------------------------------------------------------------------------
В данном примере видим  настройки виртуальных сетевых интерфейсов и что они находиться в состоянии  UP
---------------------------------------------------------------------------------------------------------------
Еще раз со стороны сетевого оборудования я настроил только на 2 коммутаторах 4 svi интерфейса  по 2 на каждом
как они распределены 
К коммутатору MES2  подключен интерфейс trex   ens22f0np0 ну и следовательно его виртуальные интерфейсы которые в совою очередь работают в режиме clieta
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 14 | 1    | 98:01.0 | 36:d3:f2:8a:20:5f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 15 | 1    | 98:01.1 | ba:74:e4:84:cc:dd | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 16 | 1    | 98:01.2 | 7a:28:51:7a:50:2f | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 17 | 1    | 98:01.3 | 2a:ce:40:94:9f:a3 | Ethernet Adaptive Virtual Function      | iavf   | ens22f0v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
-----------------------------------------------------------------------------------------------------------------------------------------------------------
К коммутатору MES1  подключен интерфейс trex    ens22f1np1 ну и следовательно его виртуальные интерфейсы которые в свою очередь работают в режиме servera
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 18 | 1    | 98:11.0 | ae:3c:ca:14:17:36 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v0  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 19 | 1    | 98:11.1 | 7a:25:11:01:51:74 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v1  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 20 | 1    | 98:11.2 | 0a:1a:be:b8:f7:33 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v2  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
| 21 | 1    | 98:11.3 | fa:70:43:32:39:38 | Ethernet Adaptive Virtual Function      | iavf   | ens22f1v3  |          |
+----+------+---------+-------------------+-----------------------------------------+--------+------------+----------+
------------------------------------------------------------------------------------------------------------------------------------------------------------------------
