### Установка DHCP сервера Kea и PostgreSQL

[Документация Kea](https://kea.readthedocs.io/en/latest/arm/install.html)

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
sudo -u postgres psql <<EOF

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
EOF
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

```shell
sudo nano /etc/kea/kea-dhcp4.conf
```
