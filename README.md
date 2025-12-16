<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/en/d/dd/MySQL_logo.svg" alt="MySQL Logo" width="300">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-mysql__replication-00758F?style=for-the-badge&logo=mysql&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-17.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
1. Поднять два сервера MySQL (Master и Slave) в Vagrant.
2. Настроить репликацию (GTID).
3. `master`: настроен на запись, заливает базу `bet.dmp`.
4. `slave`: настроен на чтение, автоматически подхватывает мастера.

### ✅ Результат
- [x] Стенд развернут (Vagrant).
- [x] Репликация работает.
- [x] Данные на слейве идентичны мастеру.

**1. Статус репликации (Slave):**
Результат см. на скриншоте 🖼️ ["1.png"](https://github.com/kamil1403/otus_mysql/blob/main/screenshots/otus_mysql_1.png)

**2. Данные в базе (Slave):**
Результат см. на скриншоте 🖼️ ["2.png"](https://github.com/kamil1403/otus_mysql/blob/main/screenshots/otus_mysql_2.png)

### 🧭 Оглавление
- [🧰 Шаг 1 - Настройка Vagrant](#one)
- [🧰 Шаг 2 - Проверка](#two)

---

<a id="one"></a>
## 🧰 Шаг 1 - Настройка Vagrant
**Master (192.168.56.10):**
**Slave (192.168.56.11):**

Все автоматизировано в `Vagrantfile`.
```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "master" do |master|
    master.vm.network "private_network", ip: "192.168.56.10"
    master.vm.hostname = "master"

    master.vm.provision "shell", inline: <<-SHELL
      echo "=== [MASTER] Старт ==="
      export DEBIAN_FRONTEND=noninteractive

      ufw disable
      iptables -F

      apt-get update
      apt-get install -y mysql-server

      cat > /etc/mysql/mysql.conf.d/mysqld.cnf <<EOF
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
bind-address    = 0.0.0.0
server-id       = 1
log_bin         = /var/log/mysql/mysql-bin.log
gtid_mode       = ON
enforce_gtid_consistency = ON
EOF

      systemctl restart mysql
      sleep 5

      mysql -e "CREATE USER IF NOT EXISTS 'repl'@'%' IDENTIFIED BY 'password';"
      mysql -e "GRANT ALL PRIVILEGES ON *.* TO 'repl'@'%';"
      mysql -e "FLUSH PRIVILEGES;"

      if [ -f /vagrant/bet.dmp ]; then
          echo "=== [MASTER] Заливаем базу ==="
          mysql -e "CREATE DATABASE IF NOT EXISTS bet;"
          mysql bet < /vagrant/bet.dmp
      fi

      echo "=== [MASTER] Готов ==="
    SHELL
  end

  config.vm.define "slave" do |slave|
    slave.vm.network "private_network", ip: "192.168.56.11"
    slave.vm.hostname = "slave"

    slave.vm.provision "shell", inline: <<-SHELL
      echo "=== [SLAVE] Старт ==="
      export DEBIAN_FRONTEND=noninteractive

      ufw disable
      iptables -F

      apt-get update
      apt-get install -y mysql-server

      cat > /etc/mysql/mysql.conf.d/mysqld.cnf <<EOF
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
bind-address    = 0.0.0.0
server-id       = 2
log_bin         = /var/log/mysql/mysql-bin.log
gtid_mode       = ON
enforce_gtid_consistency = ON
EOF

      systemctl restart mysql

      echo "15 сек"
      sleep 15
      ip neigh flush all

      mysql -e "CREATE DATABASE IF NOT EXISTS bet;"

      mysqldump -h 192.168.56.10 -u repl -ppassword bet \
        --ignore-table=bet.events_on_demand \
        --ignore-table=bet.v_same_event \
        --source-data=2 --single-transaction | mysql bet

      mysql -e "CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.56.10', SOURCE_USER='repl', SOURCE_PASSWORD='password', SOURCE_AUTO_POSITION=1, GET_SOURCE_PUBLIC_KEY=1;"
      mysql -e "START REPLICA;"

      echo "Готово"
      mysql -e "SHOW REPLICA STATUS\\G" | grep "Running"
    SHELL
  end
end
```

<a id="two"></a>
## 🧰 Шаг 2 - Проверка
**1. Проверяем статус:**
```bash
vagrant ssh slave
sudo mysql -e "SHOW REPLICA STATUS\G"
Ожидаем:
Replica_IO_Running: Yes Replica_SQL_Running: Yes
```

**2. Проверяем данные: Смотрим список таблиц на слейве:**
```bash
vagrant ssh slave
sudo mysql -e "SHOW TABLES FROM bet;"
```
