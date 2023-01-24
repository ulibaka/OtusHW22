### создадим 3х пользователей
 
```
sudo -i
useradd day && useradd night && useradd friday
```

#### Назначим им пароли

```
echo "Password123" | passwd --stdin day && echo "Password123" | passwd --stdin night && echo "Password123" | passwd --stdin friday
```

#### Разрешаем авторизацию через ssh

#### Видим, что авторизация закрыта:
```
grep 'PasswordAuthentication' /etc/ssh/sshd_config 
#   PasswordAuthentication yes
```

#### Открываем
```
sed -i 's/.*PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
```

### Разрешим пользователям подключаться только в указанные часы\дни. Для этого в файл /etc/security/time.conf  добавим в конец строки:
```
*;*;day;Al0800-2000
*;*;night;!Al0800-2000
*;*;friday;Fr
```

#### Пытаемся подключиться ночным пользователей днем
```
ssh  night@192.168.56.7
night@192.168.56.7's password:
Connection closed by 192.168.56.7 port 22
```

#### В журнале видим запрет на подключение днем пользователю night:
```
Jan 18 19:02:37 localhost sshd[4227]: pam_time(sshd:account): no/bad times specified (rule #3)
Jan 18 19:02:37 localhost sshd[4227]: Failed password for night from 192.168.56.1 port 41714 ssh2
Jan 18 19:02:37 localhost sshd[4227]: fatal: Access denied for user night by PAM account configuration [preauth]
```

### дать конкретному пользователю права работать с докером и возможность рестартить докер сервис

#### Создаем правило (для пользователя day) в Polkit /etc/polkit-1/rules.d/01-systemd.rules 

```
polkit.addRule(function(action,subject) {
 if(action.id.match("org.freedesktop.systemd1.manage-units") &&
subject.user==="day") {
 return polkit.Result.YES; 
 }
});
```

#### Проверяем возможность запуска и остановки docker:

```
[day@server ~]$ systemctl start docker
[day@server ~]$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Пн 2023-01-23 05:43:02 UTC; 3min 55s ago
```
