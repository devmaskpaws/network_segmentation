# Vinfra Reference

Этот файл — учебный справочник по CLI `vinfra` для `Кибер Инфраструктуры`.

Он нужен для типового сценария:

- посмотреть существующие сети
- создать сети
- загрузить образы
- создать flavor'ы
- поднять виртуальные машины
- подключить ВМ к нескольким сетям
- проверить, что все реально создалось

Ниже команды приведены в практической логике, а не в алфавитном порядке.

---

## 1. Что такое `vinfra`

`vinfra` — это CLI `Кибер Инфраструктуры` для управления вычислительным кластером, сетями, образами и виртуальными машинами.

Для учебного стенда он нужен, когда ты хочешь:

- создать виртуальные сети под `LAN`, `MGMT`, `DMZ`
- импортировать образ ВМ
- создать ВМ с несколькими NIC
- проверить состояние стенда без GUI

---

## 2. Базовая идея рабочего процесса

Обычно порядок такой:

1. посмотреть доступные сети, образы и flavor'ы
2. при необходимости создать сеть
3. при необходимости загрузить образ
4. при необходимости создать flavor
5. создать ВМ
6. проверить ВМ
7. при необходимости добавить еще один интерфейс

---

## 3. Посмотреть сети

```bash
vinfra service compute network list
```

Что делает:

- выводит список вычислительных сетей в кластере.

Зачем нужно:

- чтобы понять, существуют ли уже сети `LAN`, `MGMT`, `DMZ`, `Public` или `Mirror`.

Если нужен более удобный вид:

```bash
vinfra service compute network list -c id -c name -c cidr -c allocation_pools
```

Что делает:

- выводит только ключевые поля: ID, имя, подсеть и пул адресов.

Зачем нужно:

- так проще быстро ориентироваться в сетях, когда их много.

---

## 4. Посмотреть подробности по конкретной сети

```bash
vinfra service compute network show LAN
```

Что делает:

- показывает подробную информацию о сети `LAN`.

Что можно увидеть:

- `cidr`
- `gateway_ip`
- `enable_dhcp`
- `physical_network`
- `type`
- `vlan_id`

Зачем нужно:

- чтобы проверить, что сеть реально создана с правильной адресацией и это именно та сеть, которую ты собираешься использовать.

---

## 5. Создать виртуальную сеть

Пример учебной внутренней сети:

```bash
vinfra service compute network create LAN --cidr 192.0.2.0/24 --gateway 192.0.2.1
```

Что делает:

- создает виртуальную сеть `LAN` с подсетью `192.0.2.0/24` и шлюзом `192.0.2.1`.

Что реализует:

- отдельный логический сегмент для внутренних сервисов.

Пример сети управления:

```bash
vinfra service compute network create MGMT --cidr 198.51.100.0/24 --gateway 198.51.100.1
```

Пример `DMZ`:

```bash
vinfra service compute network create DMZ --cidr 203.0.113.0/24 --gateway 203.0.113.1
```

Если нужна сеть без настройки шлюза:

```bash
vinfra service compute network create MIRROR --no-gateway
```

Что делает:

- создает сеть без L3-шлюза.

Что реализует:

- вспомогательный сегмент, например под IDS capture или mirror-трафик.

---

## 6. Полезные опции `network create`

Часто полезны такие параметры:

```bash
--cidr 192.0.2.0/24
--gateway 192.0.2.1
--dns-nameserver 8.8.8.8
--allocation-pool 192.0.2.100-192.0.2.200
--dhcp
--no-dhcp
--physical-network Public
--vlan 10
```

Что они значат:

- `--cidr` задает подсеть
- `--gateway` задает IP-шлюза
- `--dns-nameserver` задает DNS для DHCP/IPAM
- `--allocation-pool` задает диапазон выдаваемых IP
- `--dhcp` включает DHCP
- `--no-dhcp` отключает DHCP
- `--physical-network` связывает сеть с физической инфраструктурной сетью
- `--vlan` задает VLAN ID для VLAN-сети

Зачем нужно:

- чтобы делать не только простые внутренние сети, но и внешние или VLAN-сегменты.

---

## 7. Изменить параметры сети

```bash
vinfra service compute network set LAN --dns-nameserver 1.1.1.1
```

или

```bash
vinfra service compute network set LAN --no-dhcp
```

Что делает:

- меняет свойства уже существующей сети.

Что реализует:

- донастройку сети без пересоздания.

---

## 8. Удалить сеть

```bash
vinfra service compute network delete MIRROR
```

Что делает:

- удаляет сеть `MIRROR`.

Зачем нужно:

- для очистки учебного стенда от лишних сетей.

Осторожно:

- сеть не удалится, если к ней еще подключены ВМ или другие ресурсы.

---

## 9. Посмотреть образы

```bash
vinfra service compute image list
```

Что делает:

- показывает все доступные образы виртуальных машин.

Зачем нужно:

- чтобы увидеть, есть ли уже образ Linux, firewall, appliance или другой нужный шаблон.

---

## 10. Загрузить образ

```bash
vinfra service compute image create mylinux --file /path/to/linux.qcow2
```

Что делает:

- загружает локальный файл образа в вычислительный кластер.

Что реализует:

- подготовку шаблона, из которого потом создаются ВМ.

Можно дополнительно указывать:

```bash
--min-disk 20
--min-ram 2048
--disk-format qcow2
--protected
```

Что это значит:

- `--min-disk` минимальный размер диска для запуска из образа
- `--min-ram` минимальный объем RAM
- `--disk-format` формат диска
- `--protected` защита образа от удаления

---

## 11. Посмотреть сведения об образе

```bash
vinfra service compute image show mylinux
```

Что делает:

- показывает подробности об образе.

Зачем нужно:

- чтобы проверить статус образа и убедиться, что он загрузился корректно.

---

## 12. Создать flavor

```bash
vinfra service compute flavor create sec-small --vcpus 2 --ram 4096
```

Что делает:

- создает новый тип ВМ с `2 vCPU` и `4096 MB RAM`.

Что реализует:

- стандартный шаблон ресурсов для виртуальных машин.

Примеры:

```bash
vinfra service compute flavor create fw-medium --vcpus 4 --ram 8192
vinfra service compute flavor create ids-medium --vcpus 4 --ram 8192
```

Зачем нужно:

- чтобы быстро и одинаково раздавать ресурсы нескольким ВМ.

---

## 13. Посмотреть flavor'ы

```bash
vinfra service compute flavor list
```

Что делает:

- показывает доступные типы ВМ.

Зачем нужно:

- чтобы не создавать дубли и выбрать подходящий шаблон.

Посмотреть детали:

```bash
vinfra service compute flavor show sec-small
```

Что делает:

- показывает RAM, vCPU и другие свойства конкретного flavor'а.

---

## 14. Создать ВМ из образа

Минимальный пример:

```bash
vinfra service compute server create APP01 \
  --network id=DMZ \
  --volume source=image,id=app-linux,size=20 \
  --flavor sec-small
```

Что делает:

- создает виртуальную машину `APP01`
- подключает ее к сети `DMZ`
- создает диск из образа `app-linux`
- использует flavor `sec-small`

Что реализует:

- стандартный способ поднять сервисную ВМ в одном сегменте.

---

## 15. Создать ВМ с несколькими интерфейсами

Пример для firewall:

```bash
vinfra service compute server create FW01 \
  --network id=PROVIDER \
  --network id=LAN \
  --network id=MGMT \
  --network id=DMZ \
  --volume source=image,id=ideco-ngfw,size=80 \
  --flavor fw-medium
```

Что делает:

- создает ВМ `FW01`
- подключает ее сразу к нескольким сетям
- создает загрузочный диск из образа `ideco-ngfw`

Что реализует:

- многоногий firewall или маршрутизатор, через который проходят разные сегменты.

Почему это важно:

- для задач сегментации одна ВМ часто должна быть одновременно в `WAN`, `LAN`, `MGMT` и `DMZ`.

---

## 16. Создать ВМ с фиксированным IP на интерфейсе

```bash
vinfra service compute server create myvm \
  --network id=LAN,fixed-ip=192.0.2.10 \
  --volume source=image,id=mylinux,size=20 \
  --flavor sec-small
```

Что делает:

- при создании сразу просит конкретный IP для интерфейса.

Что реализует:

- более предсказуемую адресацию, если в сети включено IPAM или DHCP.

Замечание:

- на практике в учебных стендах часто все равно настраивают IP внутри гостевой ОС вручную.

---

## 17. Посмотреть список ВМ

```bash
vinfra service compute server list
```

Что делает:

- показывает виртуальные машины, их ID, имя, статус и хост.

Зачем нужно:

- чтобы быстро понять, какие ВМ уже созданы и в каком они состоянии.

Фильтр по имени:

```bash
vinfra service compute server list --name APP01
```

---

## 18. Посмотреть подробности ВМ

```bash
vinfra service compute server show FW01
```

Что делает:

- показывает подробную информацию по ВМ.

Что особенно полезно смотреть:

- `status`
- `vm_state`
- `host`
- `networks`
- `volumes`

Зачем нужно:

- это главный способ проверить, к каким сетям реально подключена ВМ.

---

## 19. Добавить интерфейс к уже созданной ВМ

```bash
vinfra service compute server iface attach --network MGMT --server IDS01
```

Что делает:

- подключает сеть `MGMT` к уже существующей ВМ `IDS01`.

Что реализует:

- сценарий, когда ты сначала поднял ВМ, а потом понял, что ей нужен еще один интерфейс.

С фиксированным IP:

```bash
vinfra service compute server iface attach --network MGMT --fixed-ip 198.51.100.20 --server IDS01
```

Что делает:

- добавляет интерфейс и пытается закрепить конкретный IP.

---

## 20. Посмотреть интерфейсы ВМ

Если нужно детально проверить сетевые интерфейсы ВМ, полезно после `server show` использовать и команды интерфейсов:

```bash
vinfra service compute server iface list --server FW01
```

Что делает:

- показывает интерфейсы, присоединенные к ВМ.

Зачем нужно:

- чтобы убедиться, что все NIC реально добавлены и посмотреть их идентификаторы.

---

## 21. Запустить и остановить ВМ

Запуск:

```bash
vinfra service compute server start APP01
```

Остановка:

```bash
vinfra service compute server stop APP01
```

Перезагрузка:

```bash
vinfra service compute server reboot APP01
```

Что делает:

- управляет состоянием питания ВМ.

Зачем нужно:

- для исправления зависших ВМ, изменения конфигурации или создания шаблона.

---

## 22. Посмотреть лог консоли ВМ

```bash
vinfra service compute server log APP01
```

Что делает:

- выводит журнал виртуальной машины.

Зачем нужно:

- помогает быстро понять, загрузилась ли ОС, не упала ли appliance и нет ли ошибки на раннем старте.

---

## 23. Практический набор команд для учебного стенда

### 23.1. Сначала проверить, что уже существует

```bash
vinfra service compute network list
vinfra service compute image list
vinfra service compute flavor list
vinfra service compute server list
```

Что реализует:

- быстрый аудит стартового состояния кластера.

### 23.2. Создать сети

```bash
vinfra service compute network create LAN --cidr 192.0.2.0/24 --gateway 192.0.2.1
vinfra service compute network create MGMT --cidr 198.51.100.0/24 --gateway 198.51.100.1
vinfra service compute network create DMZ --cidr 203.0.113.0/24 --gateway 203.0.113.1
vinfra service compute network create MIRROR --no-gateway
```

### 23.3. Создать flavor'ы

```bash
vinfra service compute flavor create fw-medium --vcpus 4 --ram 8192
vinfra service compute flavor create sec-small --vcpus 2 --ram 4096
vinfra service compute flavor create ids-medium --vcpus 4 --ram 8192
```

### 23.4. Загрузить образы

```bash
vinfra service compute image create ideco-ngfw --file /path/to/ideco.qcow2
vinfra service compute image create vipnet-prime --file /path/to/prime.qcow2
vinfra service compute image create vipnet-ids --file /path/to/ids.qcow2
vinfra service compute image create admin-ws --file /path/to/admin.qcow2
vinfra service compute image create app-linux --file /path/to/app.qcow2
```

### 23.5. Поднять ВМ

```bash
vinfra service compute server create FW01 \
  --network id=PROVIDER \
  --network id=LAN \
  --network id=MGMT \
  --network id=DMZ \
  --volume source=image,id=ideco-ngfw,size=80 \
  --flavor fw-medium

vinfra service compute server create PRIME01 \
  --network id=LAN \
  --volume source=image,id=vipnet-prime,size=80 \
  --flavor sec-small

vinfra service compute server create IDS01 \
  --network id=MGMT \
  --network id=MIRROR \
  --volume source=image,id=vipnet-ids,size=80 \
  --flavor ids-medium

vinfra service compute server create ADM01 \
  --network id=MGMT \
  --volume source=image,id=admin-ws,size=40 \
  --flavor sec-small

vinfra service compute server create APP01 \
  --network id=DMZ \
  --volume source=image,id=app-linux,size=20 \
  --flavor sec-small
```

### 23.6. Проверить итог

```bash
vinfra service compute server list
vinfra service compute server show FW01
vinfra service compute server show PRIME01
vinfra service compute server show IDS01
vinfra service compute server show ADM01
vinfra service compute server show APP01
```

---

## 24. На что смотреть в выводе `vinfra`

Когда читаешь вывод команд, особенно проверяй:

- `status`
- `vm_state`
- `task_state`
- `networks`
- `volumes`
- `gateway_ip`
- `cidr`
- `enable_dhcp`

Почему:

- именно по этим полям обычно видно, создан ли ресурс корректно и готов ли он к использованию.

---

## 25. Типовые ошибки

### Сеть создалась, но не так как ожидалось

Что проверить:

- правильный ли `cidr`
- не забыл ли `--gateway`
- не включен ли DHCP там, где не нужен

### ВМ создалась, но не в тех сетях

Что проверить:

- порядок и количество `--network`
- имя сети или ID сети
- не подключилась ли ВМ к сети с похожим именем

### Образ не находится

Что проверить:

- `vinfra service compute image list`
- точное имя образа
- завершилась ли загрузка образа

### ВМ есть, но не стартует

Что проверить:

- `vinfra service compute server show <vm>`
- `vinfra service compute server log <vm>`
- хватает ли RAM/CPU во flavor'е

---

## 26. Минимальный набор команд, который стоит помнить

Если нужен совсем короткий список:

```bash
vinfra service compute network list
vinfra service compute network create LAN --cidr 192.0.2.0/24 --gateway 192.0.2.1
vinfra service compute image list
vinfra service compute image create mylinux --file /path/to/linux.qcow2
vinfra service compute flavor list
vinfra service compute flavor create sec-small --vcpus 2 --ram 4096
vinfra service compute server create APP01 --network id=DMZ --volume source=image,id=app-linux,size=20 --flavor sec-small
vinfra service compute server list
vinfra service compute server show APP01
vinfra service compute server iface attach --network MGMT --server APP01
vinfra service compute server start APP01
vinfra service compute server stop APP01
```

Этого уже достаточно для большинства учебных стендов начального уровня.

---

## 27. Официальные источники

Основной справочник CLI:

- `Cyber Infrastructure Command Line Guide 5.0`
- https://docs.cyberprotect.ru/ru-RU/CyberInfrastructure/5.0/CyberInfrastructure_CommandLineGuide_ru-RU.pdf

Страница про загрузку образов:

- https://docs.cyberprotect.ru/ru-RU/CyberInfrastructure/5.0/admin/uploading-images.html

Страница про создание ВМ:

- https://docs.cyberprotect.ru/ru-RU/CyberInfrastructure/5.0/admin/creating-virtual-machines.html

Примечание:

- синтаксис `vinfra` зависит от версии `Кибер Инфраструктуры`, поэтому перед практикой всегда полезно сверять команды с документацией своей версии.
