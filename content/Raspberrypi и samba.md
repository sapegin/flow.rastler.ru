Title: RaspberryPI и Samba
Date: 2013-07-08 20:00
Category: Администрирование
Tags: raspberrypi, debian, raspbian, samba
Slug: post003
Author: Rastler
Summary: Продолжение предыдущей статьи, настройка samba-сервера для использования на RaspberryPI.

В прошлом посте я рассказывал как можно сделать torrent-качалку из RaspberryPI. Сегодня вторая заметка на эту тему. 
И так, мы загрузили первый torrent, как же нам получить доступ. Все очень просто, достаточно установить пакет Samba, который реализует протокол cifs. Устанавливается элементарно:
`sudo apt-get install samba`.

В результате установится Samba 3.x. Далее редактируем конфигурационный файл примерно вот так:


<pre><code class="text">[global]
workgroup = workgroup
server string = %h
wins support = no
dns proxy = no
security = share
encrypt passwords = yes
panic action = /usr/share/samba/panic-action %d

[torrents]
path = /mnt/usbdisk/torrents
writeable = yes
browseable = No
read only = no
guest ok = yes
force user = root
create mask = 0644
</code></pre>

После перезапуска `sudo service samba restart` можно попробовать получить доступ.

- `Windows: \\<адрес>\torrents`
- `Mac OS: smb://<адрес>/torrents`

