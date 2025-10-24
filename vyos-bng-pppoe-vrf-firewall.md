# VyOS BNG: PPPoE, VRF и Firewall для провайдерской сети

## Введение

[Разработано на основе оригинальной статьи vyos bng](https://pages.vyos.io/hubfs/BRAS-BNG/VyOS%20BNG%20-%20Deployment%20Guide.pdf)

VyOS способен выполнять функции BNG (Broadband Network Gateway), аналогично BRAS в классических провайдерских сетях. Он поддерживает PPPoE, IPoE, интеграцию с RADIUS, VRF, QoS и динамическое управление сессиями (CoA/DM). Этот документ описывает практическое применение VyOS в роли BNG для авторизации абонентов по PPPoE, управления трафиком через VRF и построения изоляции на уровне firewall.

---

## 1. Концепция PPPoE и RADIUS

PPPoE остаётся одним из самых надёжных способов предоставления доступа. Он обеспечивает:
- Авторизацию по логину и паролю через RADIUS.
- Изоляцию абонентов на уровне L3 (каждая сессия — отдельный интерфейс).
- Простое управление профилями, скоростью и VRF.
- Полноценный учёт (Accounting) и динамическое управление (CoA/Disconnect).

### Пример базовой конфигурации PPPoE-сервера VyOS

```bash
set service pppoe-server access-concentrator 'BNG1'
set service pppoe-server authentication mode radius
set service pppoe-server authentication radius server 10.11.0.5 key vyos
set service pppoe-server accounting mode radius
set service pppoe-server accounting radius server 10.11.0.5 key vyos
set service pppoe-server client-ip-pool PPPoE-POOL range 100.70.0.10-100.70.0.254
set service pppoe-server gateway-address 100.70.0.1
set service pppoe-server interface eth1
```

### Пример записи в FreeRADIUS

```text
user123 Cleartext-Password := "pass"
    Framed-IP-Address = 81.1.206.164,
    Accel-VRF-Name := "internet",
    Filter-Id := "10000/10000"
```

---

## 2. Управление VRF

VyOS поддерживает VRF (Virtual Routing and Forwarding), что позволяет отделить разные категории пользователей, например:
- `internet` — активные пользователи.
- `blocked` — неоплатившие пользователи, доступ только к порталу и DNS.

### Пример настройки VRF

```bash
set vrf name internet table 100
set vrf name internet protocols static route 0.0.0.0/0 next-hop 203.0.113.254

set vrf name blocked table 200
set vrf name blocked protocols static route 0.0.0.0/0 next-hop 10.11.10.4
```

RADIUS может динамически назначать VRF через атрибут `Accel-VRF-Name`.  
При изменении статуса абонента биллинговая система отправляет CoA-запрос, меняя VRF.

### Пример CoA для блокировки пользователя

```bash
echo 'User-Name = "user123", Accel-VRF-Name := "blocked"' | radclient -x 10.11.0.5:3799 coa vyos-secret
```

---

## 3. Изоляция абонентов

PPPoE создаёт точку-точку между абонентом и BNG, но для дополнительной безопасности рекомендуется запретить трафик между адресами из клиентских пулов.

```bash
set firewall group network-group SUBSCRIBER-POOL network 100.70.0.0/16
set firewall group network-group SUBSCRIBER-POOL network 81.1.206.160/27
```

---

## 4. Настройка Firewall

### 4.1. Защита самого VyOS

```bash
set firewall name BNG-LOCAL default-action drop
set firewall name BNG-LOCAL rule 10 action accept
set firewall name BNG-LOCAL rule 10 source address 10.11.10.0/24
set firewall name BNG-LOCAL rule 20 action accept
set firewall name BNG-LOCAL rule 20 protocol icmp
set firewall name BNG-LOCAL rule 30 action accept
set firewall name BNG-LOCAL rule 30 protocol udp
set firewall name BNG-LOCAL rule 30 destination port 1812,1813,3799,53
set interfaces ethernet eth0 firewall local name BNG-LOCAL
```

---

### 4.2. Фильтрация для VRF `internet`

```bash
set firewall name INTERNET-IN default-action accept
set firewall name INTERNET-IN rule 1 state established enable
set firewall name INTERNET-IN rule 1 state related enable
set firewall name INTERNET-IN rule 1 action accept

set firewall name INTERNET-IN rule 10 action drop
set firewall name INTERNET-IN rule 10 source group network-group SUBSCRIBER-POOL
set firewall name INTERNET-IN rule 10 destination group network-group SUBSCRIBER-POOL
set firewall name INTERNET-IN rule 10 description 'Drop subscriber-to-subscriber'

set firewall name INTERNET-IN rule 20 action accept
set firewall name INTERNET-IN rule 20 protocol udp
set firewall name INTERNET-IN rule 20 destination port 53
set firewall name INTERNET-IN rule 20 description 'Allow DNS'

set firewall name INTERNET-IN rule 30 action drop
set firewall name INTERNET-IN rule 30 destination address 10.0.0.0/8
set firewall name INTERNET-IN rule 31 action drop
set firewall name INTERNET-IN rule 31 destination address 172.16.0.0/12
set firewall name INTERNET-IN rule 32 action drop
set firewall name INTERNET-IN rule 32 destination address 192.168.0.0/16

set vrf name internet firewall in name INTERNET-IN
```

---

### 4.3. Фильтрация для VRF `blocked`

```bash
set firewall name BLOCKED-IN default-action drop

set firewall name BLOCKED-IN rule 10 action accept
set firewall name BLOCKED-IN rule 10 protocol udp
set firewall name BLOCKED-IN rule 10 destination port 53
set firewall name BLOCKED-IN rule 10 description 'Allow DNS'

set firewall name BLOCKED-IN rule 20 action accept
set firewall name BLOCKED-IN rule 20 destination address 10.11.10.4
set firewall name BLOCKED-IN rule 20 protocol tcp
set firewall name BLOCKED-IN rule 20 destination port 80,443
set firewall name BLOCKED-IN rule 20 description 'Allow payment portal'

set firewall name BLOCKED-IN rule 30 action drop
set firewall name BLOCKED-IN rule 30 source group network-group SUBSCRIBER-POOL
set firewall name BLOCKED-IN rule 30 destination group network-group SUBSCRIBER-POOL
set firewall name BLOCKED-IN rule 30 description 'Drop subscriber-to-subscriber'

set vrf name blocked firewall in name BLOCKED-IN
```

---

## 5. Проверка и мониторинг

Проверка статистики firewall:
```bash
show firewall
show firewall name INTERNET-IN statistics
```

Просмотр таблиц маршрутизации VRF:
```bash
show ip route vrf internet
show ip route vrf blocked
```

Просмотр активных PPPoE-сессий:
```bash
show pppoe-server sessions
```

---

## 6. Заключение

PPPoE на VyOS в связке с RADIUS позволяет построить надёжную и гибкую систему управления абонентами.  
Использование VRF даёт возможность разделять активных и заблокированных пользователей, а firewall обеспечивает полную изоляцию и безопасность.  
Подход масштабируется, легко интегрируется с биллингом и подходит для операторов связи любого уровня.
