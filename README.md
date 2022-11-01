# **Vagrant-стенд для обновления ядра и создания образа системы**


MS Windows 7
VirtualBox 6.1
Vagrant
Packer
Notepad++

## **Обновить ядро ОС из репозитория ELRepo**

Создан публичный репозиторий и файл Vagrantfile. 

Подключен к локальному репозиторию C:\Users\user\Documents\GitHub\otus-lessons

Запущена команда 
```
C:\Users\users\Documents\GitHub\otus-lessons>vagrant up
```

Получена ошибка
```
…

URL: ["https://vagrantcloud.com/centos/stream8"]

Error: The requested URL returned error: 404
```

Подключил VPN-сервис, повторил команду, выполнена успешно.

Виртуальная машина создана и запущена.

Проверена текущая версия ядра

```
[vagrant@kernel-update ~]$ uname -r

4.18.0-277.el8.x86_64
```

Подключен репозиторий
```
sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm

…

Installed:

elrepo-release-8.3-1.el8.elrepo.noarch
```

Установлено новое ядро из репозитория elrepo-kernel
```
sudo yum --enablerepo elrepo-kernel install kernel-ml -y

…

Installed:
  
  kernel-ml-6.0.6-1.el8.elrepo.x86_64

  
  kernel-ml-core-6.0.6-1.el8.elrepo.x86_64

  
  kernel-ml-modules-6.0.6-1.el8.elrepo.x86_64

Complete!
```
Сконфигурирован загрузчик для запуска свежего ядра по умолчанию.
```
[vagrant@kernel-update ~]$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Generating grub configuration file ...
done

[vagrant@kernel-update ~]$ sudo grub2-set-default 0
```
ВМ перезапущена.

Ядро успешно обновлено
```
[vagrant@kernel-update ~]$ uname -r
6.0.6-1.el8.elrepo.x86_64
```

## **Создание образа системы с помощью packer**

В репозитории otus-lessons создана папка packer и в ней файл centos.json
Внутри packer созданы папки scripts и http там разместил файлы со скриптами и конфигурацию запуска.
stage-1-kernel-update.sh
stage-2-clean.sh
ks.cfg 

Запустил создание образа 
```
packer build centos.json
```
Были выявлены и исправлены ошибки


в ks.cfg 

Ошибка несуществующая опция

**поменял firewall -disabled на firewall --disabled**

в centos.json

Ошибка 404 при попытке скачать ISO

**Поменял устаревшую ссылку на скачивание iso и соответственно cheksum**

Синтаксическая ошибка

**добавил "{" в начале файла**

Скрипт ожидал вода пароля для sudo

*"execute_command": "{{.Vars}} sudo -S -E bash '{{.Path}}'"*, **заменил на** *"execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E bash '{{.Path}}'",*

*\*"shutdown_command": sudo -S /sbin/halt -h -p",* **заменил на** *"shutdown_command": "echo 'vagrant' | sudo -S /sbin/halt -h -p",*

в stage-1-kernel-update.sh

Скрипт и так выполняется под sudo

**убрал sudo в строке "yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm"**

Процесс завершился успешно
```
==> Builds finished. The artifacts of successful builds are:
--> centos-8: 'virtualbox' provider box: centos-8-kernel-5-x86_64-Minimal.box
```

## **Загрузить Vagrant box в Vagrant Cloud**

Залогинился в Vagrand Cloud
```
C:\Users\user\Documents\GitHub\otus-lessons\packer>vagrant cloud auth login
…
Vagrant Cloud username or email: 
Password (will be hidden):
Token description (Defaults to "Vagrant login from wsnbtest"): 
You are now logged in.
```
Запустил публикацию образа
```
C:\Users\user\Documents\GitHub\otus-lessons\packer>vagrant cloud publish --release …/centos8-kernel5 1.0 virtualbox centos-8-kernel-5-x86_64-Minimal.box
```

Процесс пошел
```
Uploading provider with file C:/Users/filippovda/Documents/GitHub/otus-lessons/packer/centos-8-kernel-5-x86_64-Minimal.box
Progress: 0% (Rate: 37743/s, Estimated time remaining: 31:59:57)
```

Дожидаться результата не стал, так как очень длительный процесс.

