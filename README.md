# OTUS PRO Homework 24 Users and Groups (PAM)

## Домашняя работа 24: Пользователи и группы. Авторизация и аутентификация

### Домашнее задание:
1. Подготовка рабочего места   
2. Запретить всем пользователям кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников   
* дать конкретному пользователю права работать с докером и возможность перезапускать докер сервис
---
## Выполнение задания:
### 1. Подготовка рабочего места:
Выполнение домашнего задания предполагает, что на компьютере установлен Vagrant+VirtualBox   
**[Как установить Vagrant на Debian 12](https://github.com/avlikh/Install_Vagrant_Debian12/blob/main/README.md)**   

Развернем Vagrant-стенд:
  - Создайте папку с проектом и зайдите в нее (например: /opt/otus/users-groups):
```
mkdir -p /opt/otus/users-groups ; cd /opt/otus/users-groups
```
  - Клонируете проект с Github, набрав команду:
```
apt update -y && apt install git -y ; git clone https://github.com/avlikh/Otus_pro_24.git .
```
  - Запустите проект из папки, в которую склонировали проект (в нашем примере /opt/otus/users-groups):
```
vagrant up
```
Результатом выполнения команды vagrant up станет созданная виртуальная машина, с **Debian12**.   
  - Зайдите в виртуальную машину (box):
```
vagrant ssh
```
  - Дальнейшие действия выполняются от пользователя root. Переходим в root пользователя:
```
sudo -i
```
---
### 2. Определить алгоритм с наилучшим сжатием

<details>
<summary> Введение (вводные данные): </summary>

**Введение**   
Почти все операционные системы Linux — многопользовательские. Администратор Linux должен уметь создать и настраивать пользователей.   
В Linux есть 3 группы пользователей:   
●	**Администраторы** — привилегированные пользователи с полным доступом к системе. По умолчанию в ОС есть такой пользователь - **root**   
●	**Локальные пользователи** — их учетные записи создает администратор, их **права ограничены**. Администраторы могут изменять права локальных пользователей   
●	**Системные пользователи** — учетный записи, которые создаются системой для **внутренних процессов и служб**. Например пользователь — nginx   
   
У каждого пользователя есть свой уникальный идентификатор — **UID**.   
   
Чтобы упростить процесс настройки прав для новых пользователей, их объединяют в группы. Каждая группа имеет свой набор прав и ограничений. Любой пользователь, создаваемый или добавляемый в такую группу, автоматически их наследует. Если при добавлении пользователя для него не указать группу, то у него будет своя, индивидуальная группа — с именем пользователя. Один пользователь может одновременно входить в несколько групп.  
   
Информацию о каждом пользователе сервера можно посмотреть в файле **/etc/passwd**   
Для **более точных настроек пользователей** можно использовать **подключаемые модули аутентификации (PAM)**   
**PAM (Pluggable Authentication Modules** - подключаемые модули аутентификации) — набор библиотек, которые позволяют интегрировать различные методы аутентификации в виде единого API.     
   
**PAM решает следующие задачи:**   
●	**Аутентификация** — процесс подтверждения пользователем своей подлинности. Например: ввод логина и пароля, ssh-ключ и т д.   
●	**Авторизация** — процесс наделения пользователя правами   
●	**Отчетность** — запись информации о произошедших событиях   
   
**PAM может быть реализован несколькими способами:**   
●	Модуль **pam_time** — настройка доступа для пользователя с учетом времени   
●	Модуль **pam_exec** — настройка доступа для пользователей с помощью скриптов   
●	И т.д.

</details>

**Создаём пользователя otusadm и otus:**   
```
useradd otusadm && sudo useradd otus
```
Создаём пользователям **otusadm** и **otus** пароли (создадим одинаковый пароль: **Otus2024#**)
```
passwd otusadm
```
вводим пароль: **Otus2024#**   
```
passwd otus
```
вводим пароль: **Otus2024#**   
   
Создадим группу **admin**:   
```
groupadd -f admin
```  

Добавляем пользователей **vagrant**, **root** и **otusadm** в группу **admin**:
```
usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
```  

Проверим что пользователям назначились группы:
```
id vagrant && id root && id otusadm && id otus
```

<details>
<summary> результат выполнения команды: </summary>

```
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),1003(admin)
uid=0(root) gid=0(root) groups=0(root),1003(admin)
uid=1001(otusadm) gid=1001(otusadm) groups=1001(otusadm),1003(admin)
uid=1002(otus) gid=1002(otus) groups=1002(otus)
```
</details>
   
   - Примечание: так же можно grep-нуть файл /etc/group   
      
Попробуем зайти на хостовую машину под пользователем otus:
```
ssh otus@localhost
```

Если мы подключаемся первый раз, то на вопрос о сертификате необходимо ответить: yes
```
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

Проверим что мы зашли под пользователем otus:
```
whoami
```

Выйдем из сесии пользователя otus:
```
exit
```

Повторим действия для пользователя otusadm:
```
ssh otusadm@localhost   
whoami
exit
```

<details>
<summary> результат выполнения команд: </summary>

```
root@pam:~# ssh otus@localhost
The authenticity of host 'localhost (::1)' can't be established.
ED25519 key fingerprint is SHA256:ncgV5CcHot4QFN/6rwIVymPudAdhNbFGrlb8lkUjW9Y.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
otus@localhost's password:
Linux pam 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Dec  4 13:12:01 2024 from 192.168.57.10
Could not chdir to home directory /home/otus: No such file or directory
$ whoami
otus
$ exit
Connection to localhost closed.
root@pam:~# ssh otusadm@localhost
otusadm@localhost's password:
Linux pam 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Could not chdir to home directory /home/otusadm: No such file or directory
$ whoami
otusadm
$ exit
Connection to localhost closed.
root@pam:~#
```
</details>
   
Выберем метод PAM-аутентификации, так как у нас используется только ограничение по времени, то было бы логично использовать метод pam_time, однако, данный метод не работает с локальными группами пользователей и, получается, что использование данного метода добавит нам большое количество однообразных строк с разными пользователями. В текущей ситуации лучше написать небольшой скрипт контроля и использовать модуль pam_exec   
   
Создадим файл-скрипт **/usr/local/bin/login.sh**     
   
<details>
<summary>Текст скрипта:</summary>

```
#!/bin/bash

  # Получаем текущий день недели (1 для понедельника, 7 для воскресенья)
day_of_week=$(date +%u)
  # Определим группу пользователей, которым разрешен вход по выходным:
allowed_group=admin
  # Получаем имя текущего пользователя
user=$(whoami)
  # Проверяем, суббота или воскресенье
    if [ "$day_of_week" -eq 6 ] || [ "$day_of_week" -eq 7 ]; then
     echo "Сегодня суббота или воскресенье"
       # Проверяем, является ли пользователь членом группы allowed_group
       if ! groups $user | grep -wq "$allowed_group" ; then
           echo "Доступ пользователю $user запрещен на выходных"
           exit 1
       fi
    fi
echo "Сегодня $day_of_week день недели"
echo "доступ разрешен"
exit 0 
```
</details>

         
<details>
<summary> ●	 Примечание: первое условие можно сделать альтернативным способом (используя awk) </summary>

`if [ $(date | awk '{print $1}') = "Sat" ] || [ $(date | awk '{print $1}') = "Sun" ]; then`
</details>

   **Создадим скрипт:**
  
```
cat > /usr/local/bin/login.sh
#!/bin/bash

  # Получаем текущий день недели (1 для понедельника, 7 для воскресенья)
day_of_week=$(date +%u)
  # Определим группу пользователей, которым разрешен вход по выходным:
allowed_group=admin
  # Получаем имя текущего пользователя
user=$(whoami)
  # Проверяем, суббота или воскресенье
    if [ "$day_of_week" -eq 6 ] || [ "$day_of_week" -eq 7 ]; then
     echo "Сегодня суббота или воскресенье"
       # Проверяем, является ли пользователь членом группы allowed_group
       if ! groups $user | grep -wq "$allowed_group" ; then
           echo "Доступ пользователю $user запрещен на выходных"
           exit 1
       fi
    fi
echo "Сегодня $day_of_week день недели"
echo "доступ разрешен"
exit 0
```
Нажмите [**enter**], затем комбинацию клавиш [**ctrl+d**]    
    
Сделаем файл **/usr/local/bin/login.sh** исполняемым:

```
chmod +x /usr/local/bin/login.sh
```

Проверим текущую дату:
`date`   
Результат: `Fri Dec  6 13:01:28 UTC 2024`   
   - Примечание: Если у вас выходной день, то ничего делать не надо. В моем случае, меняем дату на выходной:
`date -s "7 DEC 2024 13:03:00"`
Результат: `Sat Dec  7 13:03:00 UTC 2024`
