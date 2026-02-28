# Лабораторная работа № X  
## Развёртывание и настройка почтового сервера с использованием отдельного DNS-сервера

**Специальность:** Сетевое и системное администрирование  
**Курс:** 3  
**Операционные системы:** Debian / Ubuntu Server  
**Уровень сложности:** средний  
**Форма выполнения:** индивидуально  

---

## 1. Цель лабораторной работы

Получить практические навыки развёртывания и администрирования почтового сервера в операционной системе Linux с использованием отдельного DNS-сервера, а также изучить зависимость работы почтовых сервисов от корректной DNS-настройки.

---

## 2. Задачи лабораторной работы

В процессе выполнения лабораторной работы студент должен:

- развернуть DNS-сервер в локальной сети;
- создать DNS-зону и записи для почтового домена;
- развернуть почтовый сервер на отдельной виртуальной машине;
- настроить SMTP и IMAP сервисы;
- создать локальных пользователей Linux;
- выполнить тестирование почты из консоли;
- выполнить тестирование почты с использованием почтового клиента Thunderbird;
- проанализировать логи почтовых сервисов;
- выполнить диагностику и устранение типовой ошибки Postfix.

---

## 3. Краткие теоретические сведения

**DNS (Domain Name System)** — система преобразования доменных имён в IP-адреса.  

**MX-запись** — DNS-запись, указывающая почтовый сервер, обслуживающий домен.  

**SMTP (Simple Mail Transfer Protocol)** — протокол передачи электронной почты между серверами.  

**IMAP (Internet Message Access Protocol)** — протокол доступа пользователей к почтовым ящикам.  

**Maildir** — формат хранения электронной почты, при котором каждое письмо хранится в отдельном файле.

---

## 4. Схема лабораторной установки

В лабораторной работе используются **две виртуальные машины**:

```

+-------------------+        +-------------------+
|   DNS-сервер      |        |   Почтовый сервер |
|                   |        |                   |
| dns.studentXX     | -----> | mail.studentXX    |
| 192.168.XX.10     |        | 192.168.XX.20     |
+-------------------+        +-------------------+

```

---

## 5. Исходные данные

| Параметр | Значение |
|--------|--------|
| Домен | `studentXX.local` |
| DNS-сервер | `dns.studentXX.local` |
| Почтовый сервер | `mail.studentXX.local` |
| Количество пользователей | 2 |
| Формат хранения почты | Maildir |

---

## 6. Используемое программное обеспечение

- Debian / Ubuntu Server  
- BIND9 (DNS-сервер)  
- Postfix (SMTP-сервер)  
- Dovecot (IMAP-сервер)  
- mailutils  
- Почтовый клиент Thunderbird  

---

## 7. Ход работы

---

### Шаг 1. Развёртывание DNS-сервера

1. Установить ОС Debian или Ubuntu Server на первую виртуальную машину.
2. Задать имя хоста:
   ```bash
   hostnamectl set-hostname dns.studentXX.local
3. Обновить систему и установить DNS-сервер:

   ```bash
   apt update
   apt install bind9 -y
   ```

---

### Шаг 2. Настройка DNS-зоны

1. Открыть файл `/etc/bind/named.conf.local` и добавить описание зоны:

   ```
   zone "studentXX.local" {
       type master;
       file "/etc/bind/db.studentXX.local";
   };
   ```

2. Создать файл зоны:

   ```bash
   cp /etc/bind/db.local /etc/bind/db.studentXX.local
   ```

3. Отредактировать файл `/etc/bind/db.studentXX.local`:

   ```
   @   IN  SOA dns.studentXX.local. admin.studentXX.local. (
           2
           604800
           86400
           2419200
           604800 )

       IN  NS  dns.studentXX.local.
   dns     IN  A   192.168.XX.10
   mail    IN  A   192.168.XX.20
   @       IN  MX  10 mail.studentXX.local.
   ```

4. Перезапустить DNS-сервер:

   ```bash
   systemctl restart bind9
   ```

---

### Шаг 3. Проверка работы DNS

```bash
nslookup mail.studentXX.local
nslookup -type=MX studentXX.local
```

---

### Шаг 4. Подготовка почтового сервера

1. Установить ОС Debian или Ubuntu Server на вторую виртуальную машину.
2. Задать имя хоста:

   ```bash
   hostnamectl set-hostname mail.studentXX.local
   ```
3. Указать DNS-сервер в файле `/etc/resolv.conf`:

   ```
   nameserver 192.168.XX.10
   ```

---

### Шаг 5. Установка и настройка Postfix

1. Установить Postfix:

   ```bash
   apt install postfix -y
   ```

2. При установке выбрать:

   * тип конфигурации: **Internet Site**
   * имя почтовой системы: `studentXX.local`

3. Проверить файл `/etc/postfix/main.cf`:

   ```
   myhostname = mail.studentXX.local
   mydomain = studentXX.local
   myorigin = $mydomain
   inet_interfaces = all
   home_mailbox = Maildir/
   ```

4. Перезапустить сервис:

   ```bash
   systemctl restart postfix
   ```

---

### Шаг 6. Создание локальных пользователей

```bash
adduser user1
adduser user2
```

---

### Шаг 7. Установка и настройка Dovecot

1. Установить IMAP-сервер:

   ```bash
   apt install dovecot-imapd -y
   ```

2. Проверить параметр в файле `/etc/dovecot/conf.d/10-mail.conf`:

   ```
   mail_location = maildir:~/Maildir
   ```

3. Перезапустить сервис:

   ```bash
   systemctl restart dovecot
   ```

---

### Шаг 8. Консольное тестирование почты

1. Установить утилиту:

   ```bash
   apt install mailutils -y
   ```

2. Отправить тестовое письмо:

   ```bash
   echo "Test message" | mail -s "Test" user2@studentXX.local
   ```

---

### Шаг 9. Анализ логов почтового сервера

В минимальной установке ОС файл `/var/log/mail.log` может отсутствовать.

#### Вариант 1. Установка rsyslog

```bash
apt install rsyslog -y
systemctl start rsyslog
tail /var/log/mail.log
```

#### Вариант 2. Использование journalctl

```bash
journalctl -u postfix
journalctl -u dovecot
```

---

## 8. Проверка работы почтового сервера с использованием Thunderbird

### 8.1 Установка клиента

**Linux:**

```bash
apt install thunderbird -y
```

**Windows:** установить Thunderbird с официального сайта.

---

### 8.2 Настройка почтового ящика

При добавлении учётной записи указать:

* Email: `user1@studentXX.local`
* Имя пользователя: `user1`
* Пароль: пароль пользователя Linux

---

### 8.3 Параметры серверов

**IMAP:**

* сервер: `mail.studentXX.local`
* порт: `143`
* шифрование: нет
* аутентификация: обычный пароль

**SMTP:**

* сервер: `mail.studentXX.local`
* порт: `25`
* шифрование: нет
* аутентификация: без аутентификации

---

### 8.4 Тестирование

1. Отправить письмо пользователю `user2@studentXX.local`.
2. Убедиться в получении письма.
3. При возникновении ошибок проанализировать DNS и логи.

---

## 9. Диагностика типовой ошибки Postfix

При тестировании в логах может появиться сообщение:

```
status=bounced (mail for studentXX.local loops back to myself)
```

### Причина ошибки

Почтовый домен не указан как локальный, из-за чего Postfix
пытается доставить почту через самого себя.

### Проверка

```bash
postconf | egrep "myhostname|mydomain|mydestination"
```

### Устранение ошибки

Открыть файл `/etc/postfix/main.cf` и привести параметры к виду:

```
myhostname = mail.studentXX.local
mydomain = studentXX.local
myorigin = $mydomain
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```

Перезапустить Postfix:

```bash
systemctl restart postfix
```

---

## 10. Контрольные вопросы

1. Почему почтовый сервер не может работать без DNS?
2. Назначение MX-записи
3. Разница между SMTP и IMAP
4. Назначение параметра `mydestination`
5. Почему возникает ошибка зацикливания доставки?

---

## 11. Требования к отчёту

Отчёт должен содержать:

* схему сети;
* файл DNS-зоны;
* конфигурационные файлы Postfix и Dovecot;
* скриншоты из Thunderbird;
* описание найденной и устранённой ошибки;
* выводы.

