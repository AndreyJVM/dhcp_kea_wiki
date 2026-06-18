### Установка DHCP сервера Kea и PostgreSQL

[Документация Kea](https://kea.readthedocs.io/en/latest/arm/install.html)
[Документация Stork](https://stork.readthedocs.io/en/latest/install.html)

---
- Ubuntu Server 24.04
- RAM 2G
- CPU 2
- Disk 50
- MySQL PostgresSQL
- 192.168.0.0/24
---

### Подготовка системы

```shell
sudo apt update && sudo apt upgrade -y
```

```shell
sudo timedatectl set-timezone Europe/Moscow
```

```shell
sudo apt install -y wget curl net-tools gnupg2 software-properties-common
```
### Установка PostgreSQL

```shell
sudo apt install -y postgresql postgresql-contrib
```

```shell
sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'your_postgres_admin_password';"
```

- Создание баз данных для kea и stork
```shell
sudo -u postgres psql

CREATE USER kea_user WITH PASSWORD 'kea_strong_password';
CREATE DATABASE kea_db OWNER kea_user;
GRANT ALL PRIVILEGES ON DATABASE kea_db TO kea_user;

CREATE USER stork_user WITH PASSWORD 'stork_strong_password';
CREATE DATABASE stork_db OWNER stork_user;

\c stork_db
CREATE EXTENSION IF NOT EXISTS pgcrypto;
GRANT ALL PRIVILEGES ON DATABASE stork_db TO stork_user;

\c kea_db
CREATE EXTENSION IF NOT EXISTS pgcrypto;

\q
```
- `sudo -u postgres psql -l` Проверка создания баз данных

### Установка Kea

```shell
curl -1sLf 'https://dl.cloudsmith.io/public/isc/kea-2-4/setup.deb.sh' | sudo bash
```

```shell
sudo apt update && sudo apt install -y kea-dhcp4-server kea-admin postgresql-client kea-ctrl-agent
```
- Инициализация базы данных Kea
```shell
sudo kea-admin db-init pgsql -h localhost -u kea_user -p "kea_strong_password" -n kea_db
```
- Делам копию файла перед изменениями 
```shell
cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.back
```
- меняем конфигурационный файл и вносим правки согласно подсети
- проверяем синтаксис

```shell
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
```

- Запускаем службу

```shell
sudo systemctl enable kea-dhcp4-server && sudo systemctl start kea-dhcp4-server
```
### Установка Stork (веб-интерфейс)

```shell
curl -1sLf 'https://dl.cloudsmith.io/public/isc/stork/setup.deb.sh' | sudo bash
```

```shell
sudo apt update && sudo apt install -y isc-stork-server isc-stork-agent
```
- Настройка Stork Server
- Делам копию перед изменениями
```shell
sudo /etc/stork/server.env /etc/stork/server.env.back
```
