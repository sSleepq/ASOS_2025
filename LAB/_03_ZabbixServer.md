# Лабораторная работа № Y  
## Мониторинг серверной инфраструктуры с использованием Zabbix

**Специальность:** Сетевое и системное администрирование  
**Курс:** 3  
**Операционная система:** Debian 12  
**Уровень сложности:** средний  
**Форма выполнения:** индивидуально  

---

## 1. Цель лабораторной работы

Получить практические навыки развёртывания и настройки системы мониторинга серверной инфраструктуры, а также изучить принципы контроля доступности серверов и сетевых сервисов.

---

## 2. Задачи лабораторной работы

В процессе выполнения лабораторной работы студент должен:

- развернуть сервер мониторинга;
- установить Zabbix Server из официального репозитория;
- установить Zabbix Agent на контролируемые серверы;
- подключить хосты к системе мониторинга;
- настроить мониторинг ресурсов ОС;
- настроить мониторинг сетевых сервисов;
- проверить работу триггеров;
- проанализировать типовые ошибки мониторинга.

---

## 3. Краткие теоретические сведения

**Мониторинг** — процесс непрерывного контроля состояния серверов и сервисов.  

**Zabbix Server** — центральный компонент системы мониторинга.  

**Zabbix Agent** — служба, собирающая метрики на контролируемом хосте.  

**Триггер** — логическое условие, определяющее наличие проблемы.  

---

## 4. Схема лабораторной установки

В лабораторной работе используются **три виртуальные машины**:

```

+-------------------+
| Zabbix Server     |
| zabbix.studentXX |
| 192.168.XX.30     |
+---------+---------+
|
-

|                       |
+---------+---------+   +---------+---------+
| DNS Server        |   | Mail Server       |
| dns.studentXX     |   | mail.studentXX    |
| 192.168.XX.10     |   | 192.168.XX.20     |
+-------------------+   +-------------------+

```

---

## 5. Исходные данные

| Параметр | Значение |
|--------|--------|
| Домен | `studentXX.local` |
| Сервер мониторинга | `zabbix.studentXX.local` |
| Контролируемые хосты | DNS, Mail |
| Тип мониторинга | Agent-based |

---

## 6. Используемое программное обеспечение

- Debian 12  
- Zabbix 7.0  
- Zabbix Agent  
- default-mysql-server (MySQL-совместимая СУБД)  
- Apache  
- PHP 8.2  

---

## 7. Ход работы

---

### Шаг 1. Подготовка сервера мониторинга

1. Установить Debian 12 на виртуальную машину.
2. Задать имя хоста:
   ```bash
   hostnamectl set-hostname zabbix.studentXX.local
   ```

3. Обновить систему:

   ```bash
   apt update && apt upgrade -y
   ```

---

### Шаг 2. Установка Zabbix Server (официальный сайт)

#### 2.1 Добавление официального репозитория Zabbix

```bash
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_7.0-1+debian12_all.deb
dpkg -i zabbix-release_7.0-1+debian12_all.deb
apt update
```

---

#### 2.2 Установка компонентов Zabbix

```bash
apt install zabbix-server-mysql zabbix-frontend-php \
zabbix-apache-conf zabbix-agent zabbix-sql-scripts -y
```

---

### Шаг 3. Установка и запуск СУБД

```bash
apt install default-mysql-server default-mysql-client -y
systemctl start mysql
systemctl enable mysql
```

---

### Шаг 4. Создание базы данных Zabbix

```bash
mysql
```

```sql
CREATE DATABASE zabbix
CHARACTER SET utf8mb4
COLLATE utf8mb4_bin;

CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'zabbix';

GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

---

### Шаг 5. Импорт схемы базы данных

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

Пароль:

```
zabbix
```

---

### Шаг 6. Настройка Zabbix Server

Открыть конфигурационный файл:

```bash
nano /etc/zabbix/zabbix_server.conf
```

Указать параметры подключения к БД:

```
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
```

---

### Шаг 7. Запуск сервисов

```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

---

### Шаг 8. Первоначальная настройка Web-интерфейса

Открыть в браузере:

```
http://zabbix.studentXX.local/zabbix
```

Учётные данные по умолчанию:

* Логин: `Admin`
* Пароль: `zabbix`

---

### Шаг 9. Установка Zabbix Agent на контролируемые серверы

На DNS и Mail серверах:

```bash
apt install zabbix-agent -y
```

Открыть файл конфигурации:

```bash
nano /etc/zabbix/zabbix_agentd.conf
```

Указать:

```
Server=192.168.XX.30
ServerActive=192.168.XX.30
Hostname=dns.studentXX.local
```

Перезапустить агент:

```bash
systemctl restart zabbix-agent
systemctl enable zabbix-agent
```

---

### Шаг 10. Добавление хостов в Zabbix

В Web-интерфейсе:
**Configuration → Hosts → Create host**

Добавить шаблон:

* `Linux by Zabbix agent`

---

### Шаг 11. Мониторинг ресурсов

Проверить отображение:

* CPU
* RAM
* Disk
* Network

---

### Шаг 12. Мониторинг сервисов

Добавить проверки:

* DNS — порт 53
* SMTP — порт 25
* IMAP — порт 143

---

### Шаг 13. Проверка триггеров

Остановить службу на одном из серверов:

```bash
systemctl stop postfix
```

Убедиться в появлении проблемы.

Запустить обратно:

```bash
systemctl start postfix
```

---

## 8. Контрольные вопросы

1. Назначение системы мониторинга
2. Разница между Zabbix Server и Zabbix Agent
3. Что такое триггер?
4. Какие параметры можно мониторить?
5. Почему мониторинг важен для администратора?

---

## 9. Требования к отчёту

Отчёт должен содержать:

* схему инфраструктуры;
* список добавленных хостов;
* используемые шаблоны;
* скриншоты метрик;
* скриншоты сработавших триггеров;
* выводы.
