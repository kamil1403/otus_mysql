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
🖼️ ![Replica Status](https://github.com/kamil1403/mysql_replication/blob/main/screenshots/replica_status.png)

**2. Данные в базе (Slave):**
🖼️ ![Data Verification](https://github.com/kamil1403/mysql_replication/blob/main/screenshots/data_check.png)

### 🧭 Оглавление
- [🧰 Шаг 1 - Настройка Vagrant](#one)
- [🧰 Шаг 2 - Проверка](#two)

---

<a id="one"></a>
## 🧰 Шаг 1 - Настройка Vagrant
Всё автоматизировано в `Vagrantfile`.

**Master (192.168.56.10):**
* MySQL 8.0, конфиг `mysqld.cnf` с `GTID_MODE=ON`.
* Открыт порт 3306 (iptables).
* Создан юзер `repl`.
* При старте скрипт проверяет наличие `bet.dmp` и заливает базу.

**Slave (192.168.56.11):**
* `server-id=2`.
* Делается первичный дамп с мастера и запускается `START REPLICA`.

<a id="two"></a>
## 🧰 Шаг 2 - Проверка
**1. Проверяем статус:**
```bash
vagrant ssh slave
sudo mysql -e "SHOW REPLICA STATUS\G" | grep "Running"
Ожидаем:
Replica_IO_Running: Yes Replica_SQL_Running: Yes

2. Проверяем данные: Смотрим список таблиц на слейве:
vagrant ssh slave
sudo mysql -e "SHOW TABLES FROM bet;"
