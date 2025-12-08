# Zabbix Installation & Configuration – Debian 12

## 1. Install Zabbix Repository
```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian12_all.deb
dpkg -i zabbix-release_latest_7.4+debian12_all.deb
apt update
```

---

## 2. Install Zabbix Server, Frontend, and Agent
```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

---

## 3. Create Initial Database

### a. Connect to MySQL / MariaDB
```bash
mysql -uroot -p
```
Enter root password.

### b. Create Zabbix Database and User
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
QUIT;
```

### c. Import Initial Schema and Data
```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```
Enter the `zabbix` user password when prompted.

### d. Reset `log_bin_trust_function_creators`
```bash
mysql -uroot -p
SET GLOBAL log_bin_trust_function_creators = 0;
QUIT;
```

---

## 4. Configure Zabbix Server Database

Edit `/etc/zabbix/zabbix_server.conf`:
```
DBPassword=password
```

---

## 5. Start Zabbix Server, Agent, and Apache
```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

---

## 6. Access Zabbix Web UI
Open a web browser and navigate to:

```
http://<host-ip-or-hostname>/zabbix
```

- Follow the web installer to complete setup.  
- Default credentials: `Admin / zabbix`
