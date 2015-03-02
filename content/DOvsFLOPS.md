Title: DigitalOcean vs «Российский хостинг»	
Date: 2015-03-01 08:00
Category: Администрирование
Tags: vps, vds, DigitalOcean, хостинг
Slug: digital_ocean-vs-russian_hostings
Author: Rastler
Summary: Поиск альтернативы DigitalOcean в России.

По причине изменений курса доллара к национальной валюте, а так же желанию иметь хостинг поближе к своему географическому положению(Москва), решил еще раз взглянуть на положение дел с VPS\VDS в России, точнее, в Московском регионе.
Сразу для ленивых - если не хотите читать до конца и вдаваться в подробности — смысла менять DigitalOcean вообще нет: как по цене, так и по качеству. Или я плохо искал. Диски очень медленные, это не SSD — это ерунда какая-то.
Для остальных: я не очень понимаю цены в России. Если инвестиции сделаны по старому курсу, то откуда рублевые цены по текущему курсу доллара? Единственная объективная причина — это только купленное оборудование уже по новым ценам. А такие хостеры вообще есть? :) Парадокс или жадность.
Есть такой сервис [http://poiskvps.ru](http://poiskvps.ru) для поиска хостинга. Главным критерием поиска были SSD-диски и цена ниже DigitalOcean(далее DO). Более-менее подходящими были FLOPS и Develhoster, остальные дороже, либо без SSD. FLOPS вполне адекватно работает, и есть trial на две недели за 500р.,а на Develhoster я не смог зарегистрироваться(не дождался письма с активацией аккаунта). Тестировал только FLOPS. Функционал у них близкий к DO. 
Задача — померить «сырую» производительность, то бишь просто бенчмарк по диску и процессору/памяти. Упор сделал на дисковую подсистему, поскольку обычно это слабое место всех VPS\VDS.
Диски мерил — fio, CPU — sysbench.
В двух словах — все полохо. Какие там SSD, чуть лучше одного WD Black, и то пришлось глубину параллельности операций уменьшать, чтобы получить вменяемый latency. Вышло около 320 iops при latency 44 мс – это плохо, очень плохо! Еще сложилось стойкое ощущение, что есть какое-то дисковое пенальти - после пару раз запущенных тестов производительность просела практически до нуля. Ха, и они хотят денег практически как DO. :) Возможно, плохое разделение ресурсов между виртуальными машинами, я не знаю, в чем там реально дело. Конечно, это лучше, чем Amazon EС2 small и micro, там просто слезы наворачиваются.
По процессору практически одинаково с DO. Но не за это мы DO любим. :) 

Тестировал так:
Создал файл на 3Gb, подмонтировал, как виртуальный диск, и делал все на нем. На момент dd файла диска все стало понятно.

**DO**:

<pre><code class="text">dd if=/dev/zero of=extra.disk bs=1MB count=3000 seek=1
 3000+0 records in
 3000+0 records out
3000000000 bytes (3.0 GB) copied, 15.9536 s, 188 MB/s
</code></pre>

**FLOPS**

<pre><code class="text">dd if=/dev/zero of=extra.disk bs=1MB count=3000 seek=1
 3000+0 records in
 3000+0 records out
3000000000 bytes (3.0 GB) copied, 100.921 s, 29.7 MB/s
</code></pre>

Конечно, делать выводы было рано, но в целом картина именно настолько плачевна. Хотя, может у DO такой огромный кэш на дисках, кто его знает.

Файл конфигурации fio вот:


<pre><code class="text">[readtest]
blocksize=4k
filename=/dev/loop0
rw=randread
direct=1
buffered=0
ioengine=libaio
iodepth=16

[writetest]
blocksize=4k
filename=/dev/loop0
rw=randwrite
direct=1
buffered=0
ioengine=libaio
iodepth=16
</code></pre>
Отличия для разных хостингов в длине параллельности операций iodepth. Для близкого к реальности latency на FLOPS пришлось уменьшить глубину(iodepth) до 8. Кто не в курсе: суть конфигурации теста — это писать и читать одновременно блоками по 4 кб рандомно с отключением кэш в ОС.

Честно, я не дожидался «перелопачивания» всего диска, останавливал на 5-10%, для сравнения этого достаточно. Тем более, на FLOPS тест занял бы час. :) Вывод fio в конце заметки(чтобы не загружать текст).

«Выжимка» в виде средних значений такая:

**DigitalOcean(глубина 16**):</br>
**read**  — 1538 iops, 10 ms latency, 6153.9KB/s bandwidth</br>
**write** — 1239 iops, 12 ms latency, 4957.4KB/s bandwidth

**FLOPS(глубина 8**):</br>
**read**  — 320 iops, 44 ms latency, 1280.4KB/s bandwidth</br>
**write** — 336 iops, 43 ms latency, 1235.5KB/s bandwidth

Цифры говорят, что дисковая производительность хуже в несколько раз.

По процессору особо сказать нечего, только то, что выдал sysbench в конце. Как говорил выше, хостинги по cpu практически идентичны. У FLOPS чуть лучше мгновенная производительность, видимо, частота процессора выше, но немного больше разброс между минимальными и максимальными значениями, а, значит, опять не очень шейпят между виртуальными машинами, экономят, одним словом.
Все, тушим свет, а точнее забываем о trial’е и остаемся на DigitalOcean(или что там у вас), он стоит своих денег однозначно! Единственный плюс FLOPS в плане производительности - это сетевые задержки на 50 мс меньше,  и скорость upload\download, но это, скорее, не их заслуга.
***

Результаты для DO:

<pre><code class="text">fio test.ini
readtest: (g=0): rw=randread, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=16
writetest: (g=0): rw=randwrite, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=16
fio-2.1.3
Starting 2 processes
^Cbs: 2 (f=2): [rw] [61.1% done] [2025KB/1690KB/0KB /s] [506/422/0 iops] [eta 03m:50s]
fio: terminating on signal 2
Jobs: 2 (f=1): [rw] [61.3% done] [4575KB/6360KB/0KB /s] [1143/1590/0 iops] [eta 03m:49s]
readtest: (groupid=0, jobs=1): err= 0: pid=31617: Sat Feb 21 23:55:50 2015
read : io=2178.3MB, bw=6153.9KB/s, iops=1538, runt=362465msec
slat (usec): min=1, max=133174, avg=32.07, stdev=467.38
clat (usec): min=84, max=1430.2K, avg=10351.60, stdev=31299.16
lat (usec): min=101, max=1430.2K, avg=10386.40, stdev=31302.32
clat percentiles (usec):
| 1.00th=[ 660], 5.00th=[ 1208], 10.00th=[ 1496], 20.00th=[ 1848],
| 30.00th=[ 2128], 40.00th=[ 2384], 50.00th=[ 2640], 60.00th=[ 2960],
| 70.00th=[ 3504], 80.00th=[ 4896], 90.00th=[28544], 95.00th=[55552],
| 99.00th=[108032], 99.50th=[146432], 99.90th=[378880], 99.95th=[610304],
| 99.99th=[978944]
bw (KB /s): min= 11, max=23896, per=100.00%, avg=6381.88, stdev=6662.81
lat (usec) : 100=0.01%, 250=0.02%, 500=0.30%, 750=0.98%, 1000=1.47%
lat (msec) : 2=22.47%, 4=49.94%, 10=10.68%, 20=1.43%, 50=6.68%
lat (msec) : 100=4.82%, 250=1.03%, 500=0.11%, 750=0.04%, 1000=0.02%
lat (msec) : 2000=0.01%
cpu : usr=0.98%, sys=2.99%, ctx=244880, majf=0, minf=24
IO depths : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=100.0%, 32=0.0%, >=64=0.0%
submit : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
complete : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
issued : total=r=557634/w=0/d=0, short=r=0/w=0/d=0
write test: (groupid=0, jobs=1): err= 0: pid=31618: Sat Feb 21 23:55:51 2015
write: io=1754.6MB, bw=4957.4KB/s, iops=1239, runt=362420msec
slat (usec): min=2, max=149441, avg=81.47, stdev=781.72
clat (usec): min=74, max=1429.1K, avg=12793.53, stdev=34822.59
lat (usec): min=132, max=1430.7K, avg=12882.58, stdev=34818.99
clat percentiles (usec):
| 1.00th=[ 306], 5.00th=[ 660], 10.00th=[ 1592], 20.00th=[ 2288],
| 30.00th=[ 2672], 40.00th=[ 2992], 50.00th=[ 3376], 60.00th=[ 3952],
| 70.00th=[ 5024], 80.00th=[ 8384], 90.00th=[39168], 95.00th=[60672],
| 99.00th=[119296], 99.50th=[164864], 99.90th=[415744], 99.95th=[659456],
| 99.99th=[1019904]
bw (KB /s): min= 81, max=33976, per=100.00%, avg=5247.44, stdev=5038.05
lat (usec) : 100=0.01%, 250=0.03%, 500=3.77%, 750=1.66%, 1000=1.20%
lat (msec) : 2=8.15%, 4=45.98%, 10=20.55%, 20=2.64%, 50=8.66%
lat (msec) : 100=5.83%, 250=1.28%, 500=0.15%, 750=0.04%, 1000=0.03%
lat (msec) : 2000=0.01%
cpu : usr=0.49%, sys=3.19%, ctx=176321, majf=1, minf=8
IO depths : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=100.0%, 32=0.0%, >=64=0.0%
submit : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
complete : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
issued : total=r=0/w=449159/d=0, short=r=0/w=0/d=0

Run status group 0 (all jobs):
READ: io=2178.3MB, aggrb=6153KB/s, minb=6153KB/s, maxb=6153KB/s, mint=362465msec, maxt=362465msec
WRITE: io=1754.6MB, aggrb=4957KB/s, minb=4957KB/s, maxb=4957KB/s, mint=362420msec, maxt=362420msec


sys bench —test=cpu —cpu-max-prime=20000 run
sys bench 0.4.12: multi-threaded system evaluation benchmark
Running the test with following options:
Number of threads: 1
Doing CPU performance benchmark
Threads started!
Done.
Maximum prime number checked in CPU test: 20000
Test execution summary:
total time: 50.1531s
total number of events: 10000
total time taken by event execution: 50.1428
per-request statistics:
min: 3.53ms
avg: 5.01ms
max: 103.91ms
approx. 95 percentile: 12.11ms
Threads fairness:
events (avg/stddev): 10000.0000/0.00
execution time (avg/stddev): 50.1428/0.00
</code></pre>

Результаты для FLOPS:
<pre><code class="text">fio test.ini
readtest: (g=0): rw=randread, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=16
writetest: (g=0): rw=randwrite, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=16
fio-2.1.3
Starting 2 processes
^Cbs: 2 (f=2): [rw] [18.5% done] [1910KB/1982KB/0KB /s] [477/495/0 iops] [eta 31m:08s]
fio: terminating on signal 2
readtest: (groupid=0, jobs=1): err= 0: pid=3029: Sat Feb 21 23:56:46 2015
read : io=543584KB, bw=1280.4KB/s, iops=320, runt=424553msec
slat (usec): min=0, max=94396, avg= 6.60, stdev=346.11
clat (usec): min=57, max=4681.9K, avg=44182.17, stdev=184598.60
lat (usec): min=60, max=4681.9K, avg=44189.22, stdev=184598.57
clat percentiles (usec):
| 1.00th=[ 66], 5.00th=[ 73], 10.00th=[ 89], 20.00th=[ 133],
| 30.00th=[ 262], 40.00th=[ 692], 50.00th=[ 1576], 60.00th=[ 2896],
| 70.00th=[14144], 80.00th=[64768], 90.00th=[98816], 95.00th=[136192],
| 99.00th=[552960], 99.50th=[1597440], 99.90th=[2703360], 99.95th=[3096576],
| 99.99th=[3522560]
bw (KB /s): min= 0, max=90760, per=100.00%, avg=1827.18, stdev=4924.93
lat (usec) : 100=12.17%, 250=16.36%, 500=10.34%, 750=1.38%, 1000=0.63%
lat (msec) : 2=11.86%, 4=11.84%, 10=4.58%, 20=1.61%, 50=3.62%
lat (msec) : 100=16.06%, 250=7.30%, 500=1.12%, 750=0.46%, 1000=0.07%
lat (msec) : 2000=0.20%, >=2000=0.39%
cpu : usr=0.11%, sys=0.30%, ctx=26004, majf=0, minf=44
IO depths : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=100.0%, 32=0.0%, >=64=0.0%
submit : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
complete : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
issued : total=r=135896/w=0/d=0, short=r=0/w=0/d=0
write test: (groupid=0, jobs=1): err= 0: pid=3030: Sat Feb 21 23:56:46 2015
write: io=571196KB, bw=1345.5KB/s, iops=336, runt=424527msec
slat (usec): min=0, max=56943, avg= 6.17, stdev=166.52
clat (usec): min=67, max=4681.9K, avg=42042.35, stdev=178469.92
lat (usec): min=70, max=4681.9K, avg=42049.40, stdev=178469.81
clat percentiles (usec):
| 1.00th=[ 73], 5.00th=[ 78], 10.00th=[ 90], 20.00th=[ 121],
| 30.00th=[ 245], 40.00th=[ 398], 50.00th=[ 1480], 60.00th=[ 2960],
| 70.00th=[18816], 80.00th=[63744], 90.00th=[93696], 95.00th=[123392],
| 99.00th=[509952], 99.50th=[1269760], 99.90th=[2605056], 99.95th=[2801664],
| 99.99th=[4685824]
bw (KB /s): min= 0, max=104768, per=100.00%, avg=1923.73, stdev=5804.26
lat (usec) : 100=13.52%, 250=16.98%, 500=10.65%, 750=1.36%, 1000=0.71%
lat (msec) : 2=10.35%, 4=10.48%, 10=4.37%, 20=1.76%, 50=4.25%
lat (msec) : 100=17.39%, 250=6.15%, 500=1.02%, 750=0.41%, 1000=0.05%
lat (msec) : 2000=0.18%, >=2000=0.38%
cpu : usr=0.11%, sys=0.35%, ctx=22328, majf=0, minf=30
IO depths : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=100.0%, 32=0.0%, >=64=0.0%
submit : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
complete : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
issued : total=r=0/w=142799/d=0, short=r=0/w=0/d=0
Run status group 0 (all jobs):
READ: io=543584KB, aggrb=1280KB/s, minb=1280KB/s, maxb=1280KB/s, mint=424553msec, maxt=424553msec
WRITE: io=571196KB, aggrb=1345KB/s, minb=1345KB/s, maxb=1345KB/s, mint=424527msec, maxt=424527msec


sys bench —test=cpu —cpu-max-prime=20000 run
sys bench 0.4.12: multi-threaded system evaluation benchmark
Running the test with following options:
Number of threads: 1
Doing CPU performance benchmark
Threads started!
Done.
Maximum prime number checked in CPU test: 20000
Test execution summary:
total time: 47.0343s
total number of events: 10000
total time taken by event execution: 47.0294
per-request statistics:
min: 4.29ms
avg: 4.70ms
max: 160.96ms
approx. 95 percentile: 4.83ms
Threads fairness:
events (avg/stddev): 10000.0000/0.00
execution time (avg/stddev): 47.0294/0.00
</code></pre>
