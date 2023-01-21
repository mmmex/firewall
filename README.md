## Firewall

Задачи:

- [X] реализовать knocking port
- [X] centralRouter может попасть на ssh inetrRouter через knock скрипт.
- [X] добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.
- [X] запустить nginx на centralServer.
- [X] пробросить 80й порт на inetRouter2 8080.
- [X] дефолт в инет оставить через inetRouter.
- [X] реализовать проход на 80й порт без маскарадинга
- [X] Формат сдачи ДЗ - vagrant + ansible

### Knocking port

* Для запуска проекта необходимо склонировать репозиторий: `git clone ....`

* Переходим в каталог `firewall`: `cd firewall`

* Запускаем проект: `vagrant up`

* Будет запущено 4 ВМ: `inetRouter`, `inetRouter2`, `centralRouter`, `centralServer`

* Скрипт ansible выполнит подготовку ВМ автоматически согласно ТЗ.

* После завершения работы скрипта, выполняем вход на ВМ `centralRouter`: `vagrant ssh centralRouter`

* Повышаем привелегии до суперадминистратора: `sudo -i`

* Выполняем вход на `inetRouter` по SSH через knock скрипт: `knock -v -d 100 192.168.255.1 2222:udp 3333:tcp 4444:udp; ssh vagrant@192.168.255.1`

```bash
test@test-virtual-machine:~/Otus/firewall$ vagrant ssh centralRouter
Last login: Sat Jan 21 13:04:25 2023 from 10.0.2.2
[vagrant@centralRouter ~]$ sudo -i
[root@centralRouter ~]# knock -v -d 100 192.168.255.1 2222:udp 3333:tcp 4444:udp; ssh vagrant@192.168.255.1
hitting udp 192.168.255.1:2222
hitting tcp 192.168.255.1:3333
hitting udp 192.168.255.1:4444
The authenticity of host '192.168.255.1 (192.168.255.1)' can't be established.
ECDSA key fingerprint is SHA256:0C2hhTdJ82xxKMMO5WZFvji9DdEAfYvfxXizSDoOSKo.
ECDSA key fingerprint is MD5:ff:e5:b0:1e:2b:ae:f8:5b:cb:ab:2d:5d:28:e3:60:0e.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.255.1' (ECDSA) to the list of known hosts.
vagrant@192.168.255.1's password:
Last login: Sat Jan 21 13:08:06 2023 from 10.0.2.2
[vagrant@inetRouter ~]$
```

* В логах `/var/log/knockd.log` на ВМ `inetRouter` можем наблюдать работу сервиса knock-server:

```
[2023-01-21 13:06] starting up, listening on eth1
[2023-01-21 13:09] 192.168.255.2: opencloseSSH: Stage 1
[2023-01-21 13:09] 192.168.255.2: opencloseSSH: Stage 2
[2023-01-21 13:09] 192.168.255.2: opencloseSSH: Stage 3
[2023-01-21 13:09] 192.168.255.2: opencloseSSH: OPEN SESAME
[2023-01-21 13:09] opencloseSSH: running command: /sbin/iptables -I INPUT -s 192.168.255.2 -p tcp --dport ssh -j ACCEPT
[2023-01-21 13:09] 192.168.255.2: opencloseSSH: command timeout
[2023-01-21 13:09] opencloseSSH: running command: /sbin/iptables -D INPUT -s 192.168.255.2 -p tcp --dport ssh -j ACCEPT
```

* Демонстрация работы knocking port:

[![asciicast](https://asciinema.org/a/HjJ1vHOHARzAI7KzLdCfKvVwj.svg)](https://asciinema.org/a/HjJ1vHOHARzAI7KzLdCfKvVwj)

### Iptables

* Согласно ТЗ подготовлена топология сети:

![image](https://raw.githubusercontent.com/mmmex/firewall/master/firewall-diagram.png)

* Таблица хостов:

| Имя хоста     | Адреса                                              | Кол-во интерфейсов |
|:--------------|:----------------------------------------------------|:------------------:|
| inetRouter    | 192.168.255.1/30                                    | 1                  |
| inetRouter2   | 192.168.254.1/30                                    | 1                  |
| centralRouter | 192.168.255.2/30, 192.168.254.2/30, 192.168.0.1/28  | 3                  |
| centralServer | 192.168.0.2/28                                      | 1                  |

* Выполняем вход с хостовой машины через браузер: http://127.0.0.1:8080

* Видим страничку nginx, которая находится на `centralServer`.