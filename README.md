## Задание
#### 1. Попасть в систему без пароля несколькими способами
#### 2. Установить систему с LVM, после чего переименовать VG
#### 3. Добавить модуль в initrd


## Решение
Поднимаем последнюю систему Centos "CentOS-Stream-8-20240101.0-x86_64-dvd1" в VirtualBox.
Установку системы выбираем на LVM диск.

### 1. Попасть в систему без пароля несколькими способами
#### - Способ 1. init=/bin/sh
● В конце строки начинающейся с linux добавляем init=/bin/sh и нажимаем сtrl-x для
загрузки в систему
● В целом на этом все, Вы попали в систему. Но есть один нюанс. Рутовая файловая
система при этом монтируется в режиме Read-Only. Если вы хотите перемонтировать ее в
режим Read-Write можно воспользоватьсā командой:
[root@otuslinux ~]# mount -o remount,rw /

● После чего можно убедиться записав данные в любой файл или прочитав вывод
команды:
[root@otuslinux ~]# mount | grep root

#### см. Pic. 1

#### - Способ 2. rd.break
● В конце строки начинающейся с linux обавляем rd.break и нажимаем сtrl-x для
загрузки в систему
● Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же
в режиме Read-Only, но мы не в ней. Далее будет пример как попасть в нее и поменять
пароль администратора:
[root@otuslinux ~]# mount -o remount,rw /sysroot
[root@otuslinux ~]# chroot /sysroot
[root@otuslinux ~]# passwd root
[root@otuslinux ~]# touch /.autorelabel
● После чего можно перезагружатþся и заходитя в систему с новым паролем. Полезно
когда вы потеряли или вообще не имели пароль администратора.

#### см. Pic. 2

#### - Способ 3. rw init=/sysroot/bin/sh
● В строке начинающейсā с linux заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-x
для загрузки в систему
● В целом то же самое что и в прошлом примере, но файловая система сразу
смонтирована в режим Read-Write
● В прошлых примерах тоже можно заменить ro на rw
#### см. Pic. 3

### 2. Установить систему с LVM, после чего переименовать VG

● Первым делом посмотрим текущее состояние системы:
[root@otuslinux ~]# vgs
● Приступим к переименованию:
[root@otuslinux ~]# vgrename OR Otus
Volume group "OR" successfully renamed to "Otus"
● Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое
название на новое. По ссылкам можно увидеть примеры получившихся файлов.
● Пересоздаем initrd image, чтобы он знал новое название Volume Group
[root@otuslinux ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

#### см. Pic. 4
Перезагружаемся.
Перед загрузкой ОС выбираем первую строку, жмем “e” и правим загрузочную запись, указывая верный Lovical Volume / Volume Group.
#### см. Pic. 5 и 6
Жмем ctrl+x для загрузки.
Далее:
Создаем новый grub.cfg файл для GPT (UEFI-based) систем
$ sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
#### см. Pic. 7
Теперь можно перезагружаться, запись grub корректна

### 3. Добавить модуль в initrd

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы
добавить свой модуль создаем там папку с именем 01test:
[root@otuslinux ~]# mkdir /usr/lib/dracut/modules.d/01test
В нее поместим два скрипта:
1. module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh:
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}

2. test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик:
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."

Пересобираем образ initrd
[root@otuslinux ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
или
[root@otuslinux ~]# dracut -f -v
● Можно проверить/посмотреть какие модули загружены в образ:
[root@otuslinux ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test

После проведения этих пунктов перезагружаемся. При загрузке ОС выбираем первую строку, жмем “e” и правим загрузочную запись:
руками удаляем опции rghb и quiet и нажимаем ctrl+x для загрузки.

В итоге при загрузке будет пауза на 10 секунд и вы увидите пингвина в выводе:
#### см. Pic. 8
