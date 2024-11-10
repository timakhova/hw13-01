# Домашнее задание к занятию «Уязвимости и атаки на информационные системы» Тимахова Наталья

# Задание 1

```
timakhova@WIN-NR2FUSCI5RM:~$ nmap -sV 192.168.67.128
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-09 20:30 MSK
Nmap scan report for 192.168.67.128
Host is up (0.018s latency).
Not shown: 978 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login?
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  rpcbind
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8180/tcp open  unknown
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 126.32 seconds
```
### Найденные сетевые службы
Некоторые из служб, которые можно изучить для уязвимостей:
- **FTP (21/tcp):** vsftpd 2.3.4
- **SSH (22/tcp):** OpenSSH 4.7p1
- **HTTP (80/tcp):** Apache httpd 2.2.8
- **MySQL (3306/tcp):** MySQL 5.0.51a
- **PostgreSQL (5432/tcp):** PostgreSQL 8.3.0 - 8.3.7
- **IRC (6667/tcp):** UnrealIRCd

### Поиск уязвимостей
Теперь, зная версии сервисов, ищем их на [Exploit Database](https://www.exploit-db.com/). Несколько примеров:

1. **vsftpd 2.3.4 Backdoor Command Execution**  
   - Уязвимость позволяет выполнить произвольный код через заднюю дверь, оставленную в версии 2.3.4.
   - [Exploit link](https://www.exploit-db.com/exploits/17491)

2. **UnrealIRCd 3.2.8.1 Backdoor Command Execution**  
   - Уязвимость позволяет злоумышленнику получить доступ через командную оболочку на порту 6667.
   - [Exploit link](https://www.exploit-db.com/exploits/13853)

3. **Apache 2.2.8 mod_negotiation Buffer Overflow**  
   - Уязвимость переполнения буфера в модуле mod_negotiation, позволяющая потенциально выполнить произвольный код.
   - [Exploit link](https://www.exploit-db.com/exploits/42121)

# Задание 2

Файл записи из Вайршарк: https://github.com/timakhova/hw13-01/blob/main/nmap.pcapng

1. **SYN-сканирование** (быстрое и незаметное):
   ```bash
   nmap -sS 192.168.67.128
   ```
   SYN-сканирование отправляет SYN-пакеты (запросы на установку соединения) на порты, но не завершает соединение.

2. **FIN-сканирование** (обход некоторых фильтров):
   ```bash
   nmap -sF 192.168.67.128
   ```
   FIN-сканирование посылает FIN-пакеты (пакеты завершения соединения) без предварительного установления соединения, пытаясь скрыть факт сканирования.

3. **Xmas-сканирование** (типа "рождественская ёлка"):
   ```bash
   nmap -sX 192.168.67.128
   ```
   Этот тип сканирования посылает пакеты с установленными флагами FIN, PSH и URG, что может иногда обходить сетевые фильтры.

4. **UDP-сканирование** (для проверки UDP-портов):
   ```bash
   nmap -sU 192.168.67.128
   ```
   UDP-сканирование отправляет UDP-пакеты, чтобы проверить открытые порты.

### Шаг 3: Анализ в Wireshark:

1. **SYN-сканирование**
   - **Сетевой трафик**: отправляются пакеты SYN, и если порт открыт, сервер отвечает пакетом SYN/ACK; если порт закрыт — RST.
   - **Ответы сервера**: открытые порты отвечают SYN/ACK, закрытые порты отвечают RST. Это делает SYN-сканирование быстрым и эффективным.

2. **FIN-сканирование**
   - **Сетевой трафик**: отправляются FIN-пакеты, не пытаясь установить соединение.
   - **Ответы сервера**: открытые порты не отвечают, что затрудняет обнаружение; закрытые порты отправляют RST. Этот метод иногда обходится мимо межсетевых экранов, которые ожидают SYN.

3. **Xmas-сканирование**
   - **Сетевой трафик**: отправляются пакеты с установленными флагами FIN, PSH, и URG.
   - **Ответы сервера**: аналогично FIN-сканированию, открытые порты не отвечают, закрытые порты отвечают RST. Этот тип сканирования тоже может обходить фильтры.

4. **UDP-сканирование**
   - **Сетевой трафик**: отправляются пустые UDP-пакеты на каждый порт.
   - **Ответы сервера**: открытые порты не отвечают, закрытые порты могут отправлять ICMP-сообщения "Destination Unreachable - Port Unreachable". UDP-сканирование медленнее из-за задержек между пакетами.

### Итог
**Отличия в трафике**:  
- SYN-сканирование использует трехэтапный процесс и получает быстрые ответы, эффективный для TCP-сканирования.
- FIN и Xmas посылают пакеты, на которые открытые порты не реагируют, что затрудняет обнаружение сканирования.
- UDP-сканирование медленное, требует больше времени из-за того, что открытые порты молчат, и в основном отвечает только ICMP-ошибками для закрытых портов.

**Ответы сервера**:  
- **SYN**: SYN/ACK от открытых портов, RST от закрытых.
- **FIN и Xmas**: нет ответа от открытых портов, RST от закрытых.
- **UDP**: ICMP-ответы для закрытых портов, молчание для открытых.


