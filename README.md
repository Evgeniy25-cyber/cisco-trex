Основное  при сбоке cisco-trex, надо понимать  взомосвязи сборки 
Версия linux and kernal
dpdk = fimafare=kernal and draiver  сетевого контроллера: intel-710, E810-C, Melanox
Каждая версия cisco-trex  работает с обределнной версией dpdk: есть в фоцальной докуентации
Утсанвка версии 3.06 Debian 2.11 Ubuntu-24.04
-  Debian 2.11
-  
GRUB_CMDLINE_LINUX="iommu=pt, intel_iommu=on"
---------------------
1. Применяем изменения для grub
update-grub
2. Перезагружаемся

# find / -name update-grub
/usr/sbin/update-grub
Ran it, just to get the next error message.
# /usr/sbin/update-grub
/usr/sbin/update-grub: 4: exec: grub-mkconfig: not found
Searched for grub-mkconfig and found it under /usr/sbin/grub-mkconfig. Then it came to me, let's see how the update-grub script looks like?
cat /usr/sbin/update-grub |grep grub-mkconfig
так как trex поддерживает работу только с python3.11 необходимо установить данные пакеты
apt install python3.11-venv
python3.11 -m venv venv
source ./venv/bin/activate
venv/bin/python venv/bin/pip install cffi
далее запуспуск всех служб и скриптов проводиться только после примениниея данной команыд+укзание католога где она распологаеться
source ./venv/bin/activate
=====================================================================================================================================================================
утановка dpdk (обращать на весию cisco-trex, 
Вариатнты устанвки dpdk and hugepages
apt install dpdk - hugepages.py
варинты настройки 
По умолчанию размер страницы HugePages — 2 МБ, поэтому с помощью значения 2048 выделяется 4 ГБ оперативной памяти. 
 скриптом dpdk-hugepages.py --setup 4G
 или командой
echo 'vm.nr_hugepages = 2048' > /etc/sysctl.d/hugepages.conf
===============================================================================================





