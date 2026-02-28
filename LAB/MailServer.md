# Лабораторная работа № X  
## Развёртывание почтового сервера с использованием отдельного DNS-сервера

**Специальность:** Сетевое и системное администрирование  
**Курс:** 3  
**Операционные системы:** Debian / Ubuntu Server  
**Уровень сложности:** средний  
**Форма выполнения:** индивидуально  

---

## 1. Цель лабораторной работы

Получить практические навыки развёртывания и настройки почтового сервера в ОС Linux с использованием отдельного DNS-сервера, а также изучить зависимость работы почтовых сервисов от корректной DNS-конфигурации.

---

## 2. Задачи лабораторной работы

В процессе выполнения лабораторной работы студент должен:

- развернуть DNS-сервер в локальной сети;
- создать DNS-зону и записи для почтового домена;
- развернуть почтовый сервер на отдельной виртуальной машине;
- настроить SMTP и IMAP сервисы;
- создать локальных пользователей для почтовых ящиков;
- проверить приём и отправку электронной почты;
- проанализировать работу почтовых сервисов и DNS.

---

## 3. Краткие теоретические сведения

**DNS (Domain Name System)** — система доменных имён, используемая для преобразования доменных имён в IP-адреса.  

**MX-запись** — тип DNS-записи, указывающий почтовый сервер, обслуживающий домен.  

**SMTP (Simple Mail Transfer Protocol)** — протокол передачи электронной почты.  

**IMAP (Internet Message Access Protocol)** — протокол доступа пользователей к почтовым ящикам.  

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
- OpenSSL  
- Почтовый клиент (Thunderbird / mailutils)

---

## 7. Ход работы

---

### Шаг 1. Развёртывание DNS-сервера

1. Установить ОС Linux на первую виртуальную машину.
2. Задать имя хоста:
   ```bash
   hostnamectl set-hostname dns.studentXX.local


3. Установить DNS-сервер:

   ```bash
   apt update
   apt install bind9 -y
   ```

---

### Шаг 2. Настройка DNS-зоны

1. Открыть файл `/etc/bind/named.conf.local` и добавить зону:

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

3. Отредактировать файл зоны:

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

4. Перезапустить DNS:

   ```bash
   systemctl restart bind9
   ```

---

### Шаг 3. Проверка работы DNS

На DNS-сервере или клиенте:

```bash
nslookup mail.studentXX.local
nslookup -type=MX studentXX.local
```

---

### Шаг 4. Подготовка почтового сервера

1. Установить ОС Linux на вторую виртуальную машину.
2. Задать имя хоста:

   ```bash
   hostnamectl set-hostname mail.studentXX.local
   ```
3. В файле `/etc/resolv.conf` указать DNS-сервер:

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

   * Тип: `Internet Site`
   * Почтовый домен: `studentXX.local`

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

### Шаг 6. Создание пользователей

```bash
adduser user1
adduser user2
```

---

### Шаг 7. Установка и настройка Dovecot

1. Установить Dovecot:

   ```bash
   apt install dovecot-imapd -y
   ```

2. Проверить параметр в `/etc/dovecot/conf.d/10-mail.conf`:

   ```
   mail_location = maildir:~/Maildir
   ```

3. Перезапустить сервис:

   ```bash
   systemctl restart dovecot
   ```

---

### Шаг 8. Тестирование почтового сервера

1. Отправить тестовое письмо:

   ```bash
   echo "Test message" | mail -s "Test" user2@studentXX.local
   ```

2. Проверить получение письма.

3. Просмотреть логи:

   ```bash
   tail /var/log/mail.log
   ```

---

## 8. Контрольные вопросы

1. Почему почтовый сервер не может работать без DNS?
2. Назначение MX-записи
3. Чем отличается A-запись от MX?
4. Роль Postfix и Dovecot
5. Какие проблемы возникнут при неверной DNS-настройке?

---

## 9. Требования к отчёту

Отчёт должен содержать:

* схему сети;
* DNS-зону;
* конфигурацию Postfix и Dovecot;
* результаты тестирования;
* выводы.
