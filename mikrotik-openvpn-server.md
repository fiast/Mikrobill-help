# Инструкция по настройке **OpenVPN-сервера на MikroTik** 

---

#  Настройка OpenVPN на MikroTik

## 1. Создание сертификатов

```bash
/certificate 
add name=ca country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="ca" key-size=2048 days-valid=3650 key-usage=crl-sign,key-cert-sign
sign ca ca-crl-host=127.0.0.1

/certificate 
add name=ovpn-server country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="ovpn-server" key-size=2048 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server
sign ovpn-server ca="ca"

/certificate 
add name=cl01 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl01" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl02 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl02" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl03 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl03" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl04 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl04" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl05 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl05" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl06 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl06" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl07 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl07" key-size=2048 days-valid=720 key-usage=tls-client
add name=cl08 country="RU" state="MSK" locality="MOSCOW" organization="Example LLC" unit="IT" common-name="cl08" key-size=2048 days-valid=720 key-usage=tls-client

sign cl01 ca="ca"
sign cl02 ca="ca"
sign cl03 ca="ca"
sign cl04 ca="ca"
sign cl05 ca="ca"
sign cl06 ca="ca"
sign cl07 ca="ca"
sign cl08 ca="ca"
```

---

## 2. Экспорт сертификатов

```bash
/certificate
export-certificate ca
export-certificate cl01 type=pem export-passphrase=12345678
export-certificate cl02 type=pem export-passphrase=12345678
export-certificate cl03 type=pem export-passphrase=12345678
export-certificate cl04 type=pem export-passphrase=12345678
export-certificate cl05 type=pem export-passphrase=12345678
export-certificate cl06 type=pem export-passphrase=12345678
export-certificate cl07 type=pem export-passphrase=12345678
export-certificate cl08 type=pem export-passphrase=12345678
```

> После экспорта скачайте файлы через **Files** в WinBox:
>
> * `cert_export_ca.crt`
> * `cert_export_cl0X.crt`
> * `cert_export_cl0X.key`

---

## 3. Настройка IP-пула и профиля

```bash
/ip pool
add name=ovpn_pool0 ranges=10.8.8.2-10.8.8.20

/ppp profile
add local-address=10.8.8.1 name=ovpn remote-address=ovpn_pool0

/ppp aaa
set accounting=yes
```

---

## 4. Добавление пользователей

```bash
/ppp secret
add name=cl01 password=pass01 profile=ovpn service=ovpn
add name=cl02 password=pass02 profile=ovpn service=ovpn
add name=cl03 password=pass03 profile=ovpn service=ovpn
add name=cl04 password=pass04 profile=ovpn service=ovpn
add name=cl05 password=pass05 profile=ovpn service=ovpn
add name=cl06 password=pass06 profile=ovpn service=ovpn
add name=cl07 password=pass07 profile=ovpn service=ovpn
add name=cl08 password=pass08 profile=ovpn service=ovpn
```

---

## 5. Создание OpenVPN-сервера

```bash
/interface ovpn-server server
add auth=sha1,sha256,sha512 \
    certificate=ovpn-server \
    cipher=aes128-cbc,aes256-gcm \
    default-profile=ovpn \
    disabled=no \
    mac-address=FE:7C:BB:98:9E:89 \
    name=ovpn-server1 \
    port=21294 \
    protocol=udp \
    push-routes="192.168.88.0 255.255.255.0 vpn_gateway 9" \
    require-client-certificate=yes
```

---

## 6. Разрешение доступа в Firewall

```bash
/ip firewall filter
add action=accept chain=input dst-port=21294 protocol=udp comment="OpenVPN server"
```

---

## 7. Экспорт клиентских конфигураций

```bash
/interface/ovpn-server/server
export-client-configuration server=ovpn-server1 \
    server-address=vpn.example.com \
    ca-certificate=cert_export_ca.crt \
    client-cert-key=cert_export_cl01.key \
    client-certificate=cert_export_cl01.crt

export-client-configuration server=ovpn-server1 \
    server-address=vpn.example.com \
    ca-certificate=cert_export_ca.crt \
    client-cert-key=cert_export_cl02.key \
    client-certificate=cert_export_cl02.crt
```

> После выполнения этих команд MikroTik создаст `.ovpn`-файлы, которые можно скачать через **WinBox → Files**.

---

## 8. Пример конфигурации клиента OpenVPN

Создайте файл `client.ovpn`:

```
client
dev tun
proto udp
remote vpn.example.com 21294
resolv-retry infinite
nobind
persist-key
persist-tun
ca cert_export_ca.crt
cert cert_export_cl01.crt
key cert_export_cl01.key
remote-cert-tls server
cipher AES-256-GCM
auth SHA512
verb 3
auth-user-pass
```

> Введите логин и пароль, указанные в `/ppp secret`.

---

##  Проверка работы

После подключения клиента:

```bash
/interface ovpn-server print sessions
```

Должна появиться активная сессия с IP из пула `10.8.8.0/24`.

---

##  Примечания

* Убедитесь, что порт **21294/UDP** открыт в NAT или маршрутизаторе.
* Для повышения безопасности можно ограничить доступ по IP.
* Сертификат CA нужно хранить в защищённом месте.
* Поддерживается многопользовательская работа (по разным сертификатам и паролям).

