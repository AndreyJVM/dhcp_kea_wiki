### Установка DHCP сервера Kea и mySQL MariaDB

---
- Ubuntu Server 24.04
- RAM 2G
- CPU 2
- Disk 50
- MySQL MariaDB
- 192.168.0.0/24
---

Перед установкой необходимо установить и настроить **MariaDB** для хранения
данных об аренде, резервировании хостов, опциях, а также часть параметров конфигурации сервера.

```shell
sudo apt update -y
```

- Замените на ваш часовой пояс.
```shell
sudo timedatectl set-timezone Europe/Moscow 
```
- Устанавливаем mariadb.
```shell
sudo apt install -y mariadb-server mariadb-client libmariadb-dev curl
```
- Настраиваем mariadb.
```shell
sudo mysql -u root -p
```

- Можете заменить пользователя, пароль и адрес DB.
```shell
CREATE DATABASE kea;
CREATE USER 'kea'@'localhost' IDENTIFIED BY 'ваш_надёжный_пароль';
GRANT ALL PRIVILEGES ON kea.* TO 'kea'@'localhost';
FLUSH PRIVILEGES;
# Рекомендации повышения производительности MariaDB
SET GLOBAL innodb_flush_log_at_trx_commit=2;
SHOW SESSION VARIABLES LIKE 'innodb_flush_log%';
EXIT;
```
- Устанавливаем Kea.
```shell
sudo apt install -y kea-dhcp4-server kea-admin libmariadb-dev kea-ctrl-agent
```

- Связываем Kea и MariaDB. Меняем пароль от DB. (Пароль лучше указывать в ковычках)
```shell
sudo kea-admin db-init mysql -u kea -p 'ваш_надёжный_пароль' -n kea
```

- Проверяем работу. Должны отобразиться таблицы (lease4, schema_version и др.).
```shell
mysql -u kea -p -e "SHOW TABLES;" kea
```

### Настройка DHCP

- Делаем резервную копию конфигурационного файла 
```shell
sudo cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.bak
```

- Редактируем содержимое  файла `/etc/kea/kea-dhcp4.conf` из примера **менаяем на свои данные**

### Настройка веб интерфейса Stork

- Скачиваем и устанавливаем Stork сервер и агент

```shell
curl -1sLf 'https://dl.cloudsmith.io/public/isc/stork/cfg/setup/bash.deb.sh' | sudo bash
```
```shell
sudo apt install isc-stork-server
```

```shell
sudo apt install isc-stork-agent
```
-  Отредактируйте файл `/etc/kea/kea-ctrl-agent.conf`

- Перезапускаем службы

```shell
sudo systemctl restart kea-dhcp4-server kea-ctrl-agent isc-stork-agent
```



