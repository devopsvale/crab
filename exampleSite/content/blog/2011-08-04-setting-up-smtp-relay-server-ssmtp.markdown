---
tags:
- amazon ec2
- soft
comments: true
date: 2011-08-04T14:45:45Z
title: Настройка smtp relay server - ssmtp
slug: setting-up-smtp-relay-server-ssmtp
---

Если у вас есть свой сервер, вы наверняка не раз задумывались о том, чтобы иметь возможность отсылать с него логи себе на почту, или дать возможность веб-сайту на этом сервере рассылать письма.

Обычно настраивается postfix или sendmail или что-нибудь чуть менее громоздкое. Я предлагаю вам вместо этого настроить smtp relay server, ssmtp, который предназначен именно для пересылки отправляемых писем на "большой" сервер - [Яндекс почта для домена](https://pdd.yandex.ru/), [Google Apps for domain](https://www.google.com/work/apps/business/), или на почтовый аккаунт [Gmail](https://mail.google.com) или [Яндекс](https://mail.yandex.ru)'а, если вам потребуется посылать письма только себе.

Настройка на примере Ubuntu.

Установим ssmtp и отредактируем revaliases. Вставьте либо секцию для gmail, либо секцию для yandex mail, заменив ubuntu на ваше имя пользователя, а sendlogs на имя пользователя почты. Если вы используете почту для домена, введите полное имя пользователя своего домена, например, `noreply@terrty.net`. **Ctrl**+**O**, **Enter** для сохранения, **Ctrl**+**X** для выхода.

    sudo apt-get install ssmtp
    sudo nano /etc/ssmtp/revaliases

    #for yandex
    root:sendlogs@yandex.ru:smtp.yandex.ru:465
    ubuntu:sendlogs@yandex.ru:smtp.yandex.ru:465
    #for gmail
    root:sendlogs@gmail.com:smtp.gmail.com:587
    ubuntu:sendlogs@gmail.com:smtp.gmail.com:587

Отредактируем ssmtp.conf. Замените содержимое файла текстом из секции general ниже, и либо частью для gmail, либо частью для yandex mail. Советую использовать для учетной записи отправки почты пароль только из букв и цифр; мне не удалось заставить сервер понимать пароль со специальными символами - он выдавал ошибку при попытке отправить письмо.

    sudo nano /etc/ssmtp/ssmtp.conf

    #general
    hostname=localhost
    FromLineOverride=NO
    AuthUser=yourrobotuser@domain.com
    AuthPass=password
    #for yandex
    mailhub=smtp.yandex.ru:465
    UseTLS=YES
    #for gmail
    mailhub=smtp.gmail.com:587
    UseSTARTTLS=YES

Если вы хотите настроить отправку писем для своего веб-сайта на php, измените файл **php.ini**:

    #for apache
    sudo nano /etc/php5/apache2/php.ini
    #for nginx and php-fpm
    sudo nano /etc/php5/fpm/php.ini[/bash]

Найдите (**Ctrl**+**W**) строку **sendmail_path** (она задокументирована с помощью **;**) и ниже неё вставьте:

    sendmail_path = ssmtp -t

Вот и всё, вы и ваш сайт можете отправлять письма. Для тестирования можете создать и отправить тестовое письмо:

    cat > test << "EOF"
    To:youraccount@gmail.com
    From:yourrobotuser@domain.com
    Subject: Test
    This is a test mail.
    EOF
    ssmtp -t < test
