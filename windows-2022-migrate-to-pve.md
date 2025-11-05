#  Миграция Windows Server 2022 с физического сервера в виртуальную машину Proxmox VE 9.0.3

## 1. Исходные условия

* Существующий физический сервер с установленной **Windows Server 2022**.
* Цель — перенести систему в виртуальную среду **Proxmox VE 9.0.3**, сохранив работоспособность и данные.
* На хосте Proxmox установлены:

  * SSD 1.8 ТБ — под систему
  * два NVMe 1 ТБ — ZFS-зеркало под ВМ
  * два HDD 8 ТБ — архив и бэкапы

---

## 2. Подготовка и создание образа системы

1. На физической машине выполнено создание образа диска через инструмент резервного копирования (Acronis / Disk2VHD / аналог).

2. Полученный **VDI-диск** скопирован на Proxmox-хост.

3. На хосте диск конвертирован в формат **QCOW2**:

   ```bash
   qemu-img convert -f vdi -O qcow2 source.vdi server2022.qcow2
   ```

4. QCOW-файл временно размещён в каталоге `/mnt/mylv/import/` для последующего подключения.

---

## 3. Импорт и запуск виртуальной машины

1. Создана новая ВМ (например, ID 100) с параметрами:

   * BIOS: **OVMF (UEFI)**
   * EFI-диск добавлен
   * Контроллер: SATA0
   * Память: 8 ГБ / 2 vCPU
   * Machine type: **pc-i440fx-10.0**

2. Подмонтирован конвертированный QCOW2-диск:

   ```bash
   mkdir -p /mnt/pve/mylv/images/100
   mv server2022.qcow2 /mnt/pve/mylv/images/100/vm-100-disk-0.qcow2
   qm set 100 -sata0 mylv:100/vm-100-disk-0.qcow2
   ```

3. После запуска ВМ появилось сообщение `INACCESSIBLE_BOOT_DEVICE`.
   Проблема решена:

   * переключением диска на **SATA**;
   * использованием **BIOS UEFI (OVMF)**;
   * восстановлением загрузчика из ISO Windows Server через:

     ```cmd
     bcdboot C:\\Windows /s S: /f UEFI
     ```

---

## 4. Оптимизация и сжатие виртуального диска

1. Внутри Windows выполнена очистка и оптимизация тома (`Disk Management` + `cipher /w:C:`).

2. ВМ выключена, а на хосте произведено компактирование образа:

   ```bash
   qemu-img convert -p -O qcow2 vm-100-disk-0.qcow2 vm-100-disk-0-new.qcow2
   mv vm-100-disk-0-new.qcow2 vm-100-disk-0.qcow2
   ```

   Размер файла уменьшился с ≈ 300 ГБ до ≈ 80–90 ГБ.

3. Затем образ импортирован в ZFS-зеркало на NVMe:

   ```bash
   qm importdisk 100 /mnt/pve/mylv/images/100/vm-100-disk-0.qcow2 nvme-zfs
   ```

---

## 5. Настройка хранилищ и общих папок

1. На Proxmox создан пул ZFS:

   ```bash
   zpool create -f nvme-pool mirror /dev/nvme0n1 /dev/nvme1n1
   ```

2. Добавлен storage `nvme-zfs` через GUI.

3. HDD 8 ТБ подключён с NTFS и смонтирован:

   ```bash
   mount -t ntfs-3g /dev/sdb1 /mnt/hdd-backup
   ```

4. Папка расшарена по NFS:

   ```bash
   echo "/mnt/hdd-backup 192.168.88.0/24(rw,sync,no_subtree_check,no_root_squash,insecure)" >> /etc/exports
   exportfs -ra
   ```

   (для Windows подключена через `mount.exe 192.168.88.1:/mnt/hdd-backup Z:`).

   Из-за проблем с кодировкой NFS в Windows дополнительно настроен **Samba-доступ**, обеспечивший корректное отображение русских имён файлов.

---

## 6. Интеграция VirtIO и VirtIO-FS (Windows)

Для того чтобы Windows-гость в Proxmox мог использовать общий каталог хоста через **virtiofs**, нужно сделать несколько шагов в самой гостевой ОС.

Основа бралось из обсуждения на форуме Proxmox:
- [https://forum.proxmox.com/threads/proxmox-8-4-virtiofs-virtiofs-shared-host-folder-for-linux-and-or-windows-guest-vms.167435/](https://forum.proxmox.com/threads/proxmox-8-4-virtiofs-virtiofs-shared-host-folder-for-linux-and-or-windows-guest-vms.167435/)
и официальной документации по драйверам:
- [https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers)

### Шаги в гостевой Windows

1. **Снова подключить VirtIO ISO** к ВМ (то же ISO, что использовали при установке Windows в Proxmox).
2. В проводнике открыть CD/DVD с VirtIO и при необходимости **повторно запустить**:

   * `virtio-win-gt-x64.msi`
   * это обновит/доставит гостевые компоненты.
3. **Скачать и установить WinFSP**:
   - [https://winfsp.dev/rel/](https://winfsp.dev/rel/)
   При установке обязательно включить опции:

   * **CORE**
   * **DEVELOPER**
4. Перезагрузить гостевую Windows.
5. После перезагрузки в системе появляется поддержка virtiofs и можно монтировать проброшенную из Proxmox директорию.

### Что было сделано на стороне Proxmox

* В конфиг ВМ был добавлен virtiofs-share (через GUI или правкой `/etc/pve/qemu-server/<VMID>.conf`), где указана директория на хосте.
* После установки драйверов и WinFSP Windows-гость увидел этот virtiofs как общий ресурс.

В результате общий каталог с хоста Proxmox стал доступен внутри Windows-гостя без SMB/NFS, напрямую через virtiofs.

---

## 7. Результат

* Windows Server 2022 полностью перенесён с bare-metal на Proxmox VE 9.0.3.
* Система работает на ZFS-зеркале NVMe.
* Физические HDD 8 ТБ подключены как сетевое хранилище (NFS + Samba).
* VirtIO-драйверы установлены, включён QEMU Guest Agent и VirtIO-FS.
* Объём QCOW-образа оптимизирован и сжат.

---

✅ **Итог:**
Миграция физического Windows Server 2022 на Proxmox выполнена полностью:
диск сконвертирован, система адаптирована под виртуальное железо, загрузчик исправлен, хранилища настроены, общие папки проброшены, интеграция VirtIO и VirtIO-FS реализована.
