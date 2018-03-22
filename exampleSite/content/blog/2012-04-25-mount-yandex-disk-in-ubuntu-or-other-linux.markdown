---
tags:
- soft
comments: true
date: 2012-04-25T06:03:50Z
title: Mount Yandex.Disk in Ubuntu or Other Linux
slug: mount-yandex-disk-in-ubuntu-or-other-linux
---

[Яндекс.Диск](https://disk.yandex.ru/invite/?hash=9GMQ37ZU "Яндекс.Диск") is free 10Gb [WebDav](http://help.yandex.ru/disk/webdav.xml "Доступ к Диску через WebDAV — Яндекс.Помощь. Диск") storage.

    sudo apt-get install davfs2
    mkdir ~/yandex

    cat "https://webdav.yandex.ru /home/user/yandex   davfs   user,rw,noauto   0   0" >> /etc/fstab

    cat "/home/user/yandex user@ya.ru "password"" >> ~/.davfs2/secrets
