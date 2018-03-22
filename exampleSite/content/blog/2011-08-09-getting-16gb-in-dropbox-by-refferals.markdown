---
tags:
- ubuntu
- soft
comments: true
date: 2011-08-09T04:43:45Z
title: Скрипт прокачки рефералов Dropbox для 16Gb бесплатного места
slug: getting-16gb-in-dropbox-by-refferals
---

C помощью скрипта возможно автоматическое привлечение рефералов к вашему аккаунту dropbox, каждый реферал даёт [500Мб места](https://blog.dropbox.com/2012/04/dropbox-referrals-are-now-twice-as-nice/), 32 дадут вам 16Гб дополнительного места.

Ныне не работает, Dropbox прикрыло контору.

Последовательность действий:

Скачайте [Ubuntu](http://www.ubuntu.com/download/ubuntu/download) версии 14.04 ([.torrent](http://releases.ubuntu.com/trusty/ubuntu-14.04.5-desktop-i386.iso.torrent) or [.iso](http://releases.ubuntu.com/trusty/ubuntu-14.04.5-desktop-i386.iso)). Запишите образ на CD или [USB](http://unetbootin.sourceforge.net/) и загрузитесь с него (или запустите его в [виртуальной машине](https://www.virtualbox.org/wiki/Downloads)).
Запустите терминал (**Ctrl**+**Alt**+**T**).

    sudo nano /etc/apt/sources.list

Сотрите знак комментария **#** перед строками с **universe**, сохраните файл (**Ctrl**+**O**, **Enter**) и закройте редактор (**Ctrl**+**X**). Пример искомой строки:

    #deb http://archive.ubuntu.com/ubuntu/ natty main restricted universe

Запустите в терминале следующие команды (можете выделить этот текст, сделать Alt+Tab в окно терминала и вставить его средней кнопкой мыши):

    wget https://terrty.net/files/script-dropbox.tar && tar xvf script*.tar && cd dbhacking/
    sudo apt-get update
    sudo apt-get -y install xautomation epiphany-browser gtk2-engines-pixbuf
    wget https://www.dropbox.com/download?dl=packages/ubuntu/dropbox_1.6.0_i386.deb
    sudo dpkg -i dropbox*
    sudo apt-get -y remove network-manager network-manager-gnome
    sudo sh -c "echo auto lo > /etc/network/interfaces"
    sudo sh -c "echo iface lo inet loopback >> /etc/network/interfaces"
    sudo sh -c "echo auto eth0 >> /etc/network/interfaces"
    sudo sh -c "echo iface eth0 inet dhcp >> /etc/network/interfaces"
    sudo /etc/init.d/networking restart
    gsettings set org.gnome.Epiphany user-agent ""
    dropbox

Этим мы скачиваем и распаковываем скрипт (20Кб), устанавливаем пакеты, перенастраиваем сеть нужным нам образом, запускаем dropbox. **Внимание**! В строке: **`gsettings set org.gnome.Epiphany user-agent ""`** в кавычках вам нужно вписать некий user-agent, иначе рефералы работать не будут! Примеры:

    Mozilla/5.0 (X11; Linux x86_64)
    AppleWebKit/535.11 (KHTML, like Gecko)
    Chrome/17.0.963.12
    Safari/535.11

Если команда не работает, попробуйте вместо неё использовать **`gconftool-2 set org.ghome.Epiphany user-agent ""`**

В процессе разрешите Dropbox скачать пропиетарное ядро (25Мб), после успешного скачивания закройте окно с "I don't have a Dropbox account" / "I already have a Dropbox account".

Далее вставьте следующую команду, вставив свою реферальную ссылку с [этой страницы](https://www.dropbox.com/referrals).

    while [ 1 ]; do sh script db.tt/XXXXX; done

Для прекращения работы скрипта выделите окно терминала и нажмите **Ctrl**+**C**.

Ну и ещё совсем [немного места](https://www.dropbox.com/free) совершенно бесплатно.
