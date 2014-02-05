Title: Использование Raspberry PI как torrent-клиента
Date: 2013-06-30 21:00
Category: Администрирование
Tags: raspberrypi, debian, raspbian, torrent, transmission
Slug: post002
Author: Rastler
Summary: Использование Raspberry PI как torrent-клиент с использованием transmission-daemon

<img src="/images/raspberrypi-logo.png" height="250px" width="250px" />

Первый более-менее серьезный пост в блоге будет о RaspberryPI. Я отлично понимаю, что только ленивый не писал об этом устройстве. Но тем не менее хотелось бы поделиться опытом, как я делал самое банальное, что можно сделать - торрент-качалку. И так, что нам понадобится:

1.  RaspberyPI Model B (та, что с 512 МБ) + блок питания к нему.
2.  SD-карта на 4 и более ГБ.
3.  USB-диск собственно для хранения скачанных файлов.
4.  Дистрибутив [Rasbian](http://www.raspbian.org/) - это такая специальная сборка Debian для raspberrypi.
5. Если кто любит отдельное приложение, а не веб-интерфейс, — [Transmission GUI](https://code.google.com/p/transmisson-remote-gui/)

Рассказывать, как записать образ на SD-карту, чтобы с нее можно было загрузить наш raspberrypi, думаю, не надо. Если нужно, то попросите в комментариях к статье, и я напишу или дам ссылку. Вообще все можно найти на сайте дистрибутива.

Первая загрузка: перед нами диалоговое окно конфигурирования.

<img src="/images/raspi-config.png" width="900px" alt="Raspberrypi configuration" class="img-thumbnail"/>

Если в дальнейшем захочется вызвать эту утилиту конфигурирования снова, то:
`sudo raspi-config`
В нем стоит задать часовой пояс, время и т. д. Еще можно указать на какой частоте будет работать процессор. Я задал 800 МГц и все работает стабильно. Самый важный момент в том, что нам не нужно устанавливать X-сервер и менеджер окон. Нам нужно самый минимум, ssh-сервер и кое-какие библиотеки.
Дальше приступим собственно к установке торрент-клиента и небольшому тюнингу системы.
Первое, что нужно сделать, — это обновить систему и установленные пакеты.

<pre><code class=sh>sudo apt-get update
sudo apt-get upgrade</code></pre>

Далее устанавливаем сам клиент.

<pre><code class=sh>sudo apt-get install transmission-daemon</code></pre>

Если загрузка файлов будет на SD-карту, то никаких дополнительных действий не потребуется. Если же загрузка будет на внешний диск, то нужно его отформатировать. Я использовал файловую систему ext4. Подключаем диск, смотрим в `/var/dmesg/`, какое имя он получил. Создаем раздел на диске. Предположим, что диск `/dev/sda`:

`sudo fdisk /dev/sda` 
в диалоговом окне набираем `n` и `w`

Далее форматируем под ext4: `mkfs.ext4 /dev/sda1`.
Осталось только подмонторовать:
`sudo mount /dev/sda1 /mnt/usbdisk` (предполагается, что папка `/mnt/usbdisk` создана заранее). 
Теперь необходимо создать структуру папок и назначить права.

<pre><code class=sh>cd /mnt/usbdisk
mkdir torrents
cd torrents
mkdir completed
mkdir progress
chmod 770 progress
chmod 770 completed
sudo chgrp debian-transmission progress
sudo chgrp debian-transmission completed</code></pre>

Далее нужно в конфигурационном файле transmission-daemon `/etc/transmission-daemon/settings.json` указать новые пути. Важно перед редактированием остановить демон `sudo service transmission-daemonon stop`, иначе он перезапишет файл конфигурации своим running значениями. Вообще можно конфигурировать прямо из веб-интерфейса `http://<адрес вашего raspberrypi>:9091/transmission/web/`, даже после перезапуска все значения сохранятся. 
В принципе все, но еще важно сделать некоторый тюнинг. Не стоит забывать, что raspberry pi имеет не очень высокую производительность.

- Изменить приоритет процесса transmission-daemon. Иначе вы не сможете даже подключиться консолью в процессе активной закачки. Для этого слегка изменяем скрипт запуска:
<pre><code class=sh>
start_daemon () {
    if [ $ENABLE_DAEMON != 1 ]; then
        log_progress_msg "(disabled)"
		log_end_msg 255 || true
    else
        start-stop-daemon --start \
        --chuid $USER \
		$START_STOP_OPTIONS \
		--nicelevel 19 \
        --exec $DAEMON -- $OPTIONS || log_end_msg $?
		log_end_msg 0
    fi
}
</code></pre>
ключевой параметр `--nice 19`.

- Изменить некоторые настройки сетевого стека для работы с torrent. В `/etc/sysctl.conf` добавить: 
		`kernel.printk_ratelimit = 30`
		`kernel.printk_ratelimit\_burst = 200`

Практика показала, что максимальная скорость скачивания, когда еще можно зайти на консоль что-то около 3000 (загрузка) / 1000 (отдача) КБ/c.

На сегодня все. Еще раз повторю, если хотите побольше деталей, то пишите в комментариях. В следующий раз расскажу как прикрутить samba для полного комфорта. 
