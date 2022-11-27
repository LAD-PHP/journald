# Journald #  
**systemd** использует журнал (journal), собственную систему ведения логов, в связи с чем больше не требуется запускать отдельный демон логирования. Для чтения логов используется команда **journalctl**  

**Journald** — системный демон журналов systemd. Systemd спроектирован так, чтобы централизованно управлять системными логами от процессов, приложений и т.д. Все такие события обрабатываются демоном journald, он собирает логи со всей системы и сохраняет их в бинарных файлах.

Преимуществ централизованного логирования событий в бинарном формате достаточно много, например системные логи можно транслировать в различные форматы, такие как простой текст, или в JSON, при необходимости. Так же довольно просто можно отследить лог вплоть до одиночного события используя фильтры даты и времени.

Файлы логов journald могут собирать тысячи событий и они обновляются с каждым новым событием, поэтому если ваша Linux-система работает достаточно долго — размер файлов с логами может достигать несколько гигабайт и более. Поэтому анализ таких логов может происходить с задержками, в таком случае, при анализе логов можно фильтровать вывод, чтобы ускорить работу.

> **Journalctl**  — отличный инструмент для анализа логов, обычно один из первых с которым знакомятся начинающие администраторы linux систем. Встроенные возможности ротации, богатые возможности фильтрации и возможность просматривать логи всех systemd unit-сервисов одним инструментом очень удобны и заметно облегчают работу системным администраторам.

### Файл конфигурации journald

Файл конфигурации можно найти по следующему пути: **/etc/systemd/journald.conf**, он содержит различные настройки journald.

Каталог с журналом journald располагается **/run/log/journal** (в том случае, если не настроено постоянное хранение журналов, но об этом позже).

Файлы хранятся в бинарном формате, поэтому нормально их просмотреть с помощью cat или nano, как привыкли многие администраторы — не получится.

### Настройка хранения журналов

По умолчанию journald перезаписывает свои журналы логов при каждой перезагрузке, и вызов journalctl выведет журнал логов начиная с текущей загрузки системы.

Если необходимо настроить постоянное сохранение логов, потребуется отдельно это настроить, т.к. разработчики отказались от постоянного хранения всех журналов, чтобы не дублировать rsyslog.
> Rsyslog — это мощный сервис для управления логами в Linux.

Файл конфигурации находится по пути /etc/systemd/journald.conf и выглядит следующем образом:
```
Journald Configuration File
# See journald.conf(5) for details.
[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitInterval=30s
#RateLimitBurst=1000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#SystemMaxFiles=100
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#RuntimeMaxFiles=100
#MaxRetentionSec=
#MaxFileSec=1month
#ForwardToSyslog=yes
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
```

Для постоянного хранения логов необходимо раскоментировать занчение Storage= и установить для него значение persistent (постоянный)
```
[Journal]
Storage=persistent
```
После чего необходимо перезагрузить службу systemd-journald.service
```
systemctl restart systemd-journald.service
```
Это создаст каталог /var/log/journal, и все файлы журнала будут сохранены в эту директорию.


Полное описание:
> **Storage=** Указывает, где хранить журнал. Доступны следующие параметры:
> * **volatile** Журнал хранится в оперативной памяти, т.е. в каталоге /run/log/journal.    
> * **persistent** Данные хранятся на диске, т.е. в каталоге /var/log/journal  
> * **auto** используется по-умолчанию  
> * **none** Журнал не ведётся 

> **Compress=** Принимает значения "yes" или "no". Если включена (по-умолчанию) сообщения перед записью в журнал, будут сжиматься.  

> **Seal=** Принимает значения "yes" или "no". Если включена (по-умолчанию) будет включена защита Forward Secure Sealing (FSS), которая позволяет накладывать криптографические отпечатки на журнал системных логов.

> **SplitMode=** Определяет доступ к журналу пользователям. Доступны следующие параметры:
> * **uid** Все пользователи получают доступ к чтению системного журнала. Используется по-умочанию.  
> * **login** Каждый пользователь может читать только сообщения, относящиеся к его сеансу.  
> * **none** Пользователи не имеют доступа к системному журналу.

> **SyncIntervalSec=** Таймаут, после которого происходит синхронизация и запись журнала на диск. Относится только к уровням ERR, WARNING, NOTICE, INFO, DEBUG. Сообщения уровня CRIT, ALERT, EMERG записываются сразу на диск.

> **RateLimitInterval= и RateLimitBurst=** Настройки ограничения скорости генерации сообщений для каждой службы. Если в интервале времени, определяемого RateLimitInterval=, больше сообщений, чем указано в RateLimitBurst= регистрируются службой, все дальнейшие сообщения в интервале отбрасываются, пока интервал не закончится.При этом генерируется сообщение о количестве отброшенных сообщений. По умолчанию 1000 сообщений за 30 секунд. Единицы измерения: "s", "min", "h", "ms", "us". Чтобы выключить ограничение скорости, установите значения в 0.

> **MaxRetentionSec=** Максимальное время хранения записей журнала. Единицы измерения: year, month, week, day, h или m

> **MaxFileSec=** Максимальное время хранения записей в одном файле журнала, после которого он переводится в следующий.

> **ForwardToSyslog=, ForwardToKMsg=, ForwardToConsole=, ForwardToWall=** Определяют куда направлять сообщения: в традиционный системный журнал Syslog, в буфер журнала ядра (kmsg), на системную консоль, или на стену, чтобы было видно всем зарегистрированным пользователям. Эти опции принимают логические аргументы. Если переадресация на Syslog включен, но Syslog демон не работает, соответствующий параметр не имеет никакого эффекта. По умолчанию, только стена включена. Эти параметры могут быть переопределены во время загрузки с параметрами командной строки ядра systemd.journald.forward_to_syslog =, systemd.journald.forward_to_kmsg =, systemd.journald.forward_to_console = и systemd.journald.forward_to_wall =. При пересылке в консоль, должен быть установлен TTYPath =, как будет описано ниже.

> **TTYPath=** Назначает консоль TTY, для вывода сообщений, если установлен параметр ForwardToConsole=yes. По-умолчанию, используется /dev/console. Для того, чтобы вывести на 12 консоль, устанавливаем TTYPath=/dev/tty12. Для того, чтобы вывести на последовательный порт, устанавливаем TTYPath=/dev/ttySX, где X номер com-порта.

> **MaxLevelStore=, MaxLevelSyslog=, MaxLevelKMsg=, MaxLevelConsole=, MaxLevelWall=** Определяет максимальный уровень сообщений который сохраняется в журнал, выводится на традиционный системный журнал Syslog, буфер журнала ядра (kmsg), консоль или стену. Значения: emerg, alert, crit, err, warning, notice, info, debug или цифры от 0 до 7 (соответствуют уровням).

## Управление логированием

Определение текущего объёма логов

Со временем объём логов растёт, и они занимают всё больше места на жёстком диске. Узнать объём имеющихся на текущий момент логов можно с помощью команды:
```
journalctl --disk-usage
```
### Ротация логов

Настройка ротации логов осуществляется с помощью опций **−−vacuum-size** и **−−vacuum-time**. Первая из них устанавливает предельно допустимый размер для хранимых на диске логов (в нашем примере — 1 ГБ):
```
journalctl --vacuum-size=1G
```
>Как только объём логов превысит указанную цифру, лишние файлы будут автоматические удалены. Аналогичным образом работает опция −−vacuum-time. Она устанавливает для логов срок хранения, по истечении которого они будут автоматически удалены:
```
journalctl --vacuum-time=1years
```

>Настройка ротации в конфигурационном файле. Настройки ротации логов можно также прописать в конфигурационном файле **/еtc/systemd/journald.conf**, который включает в числе прочих следующие параметры:  
* **SystemMaxUse=** — максимальный объём, который логи могут занимать на диске;  
* **SystemKeepFree=** — объём свободного места, которое должно оставаться на диске после сохранения логов;  
* **SystemMaxFileSize=** — объём файла лога, по достижении которого он должен быть удален с диска;
* **RuntimeMaxUse=** — максимальный объём, который логи могут занимать в файловой системе /run;
* **RuntimeKeepFree=** — объём свободного места, которое должно оставаться в файловой системе /run после сохранения логов;  
* **RuntimeMaxFileSize=** — объём файла лога, по достижении которого он должен быть удален из файловой системы /run.  
> **Единицы измерения: K, M, G, T, P, E.**

Для совместимости с существующими системами journald пересылает все сообщения в классический syslogd. Этот поведение можно отключить в конфигурации /etc/systemd/journald.conf установив параметр
```
ForwardToSyslog=no
```

### Использование journalctl для просмотра и анализа логов

Основная команда для просмотра:
> journalctl    
> journalctl --utc посмотреть логи в UTC

*Она выведет все записи из всех журналов, включая ошибки и предупреждения, начиная с того момента, когда система начала загружаться. Старые записи событий будут наверху, более новые — внизу, вы можете использовать PageUp и PageDown чтобы перемещаться по списку, Enter — чтобы пролистывать журнал построчно, G -чтобы переместится в конец журнала, / - для инициализации поиска и Q — чтобы выйти.*

### Фильтрация событий по важности

Система записывает события с различными уровнями важности, какие-то события могут быть предупреждением, которое можно проигнорировать, какие-то могут быть критическими ошибками. Если мы хотим просмотреть только ошибки, игнорируя другие сообщения, введем команду с указанием кода важности:

> journalctl -p 0

Для уровней важности, приняты следующие обозначения:

0: emergency (неработоспособность системы)  
1: alerts (предупреждения, требующие немедленного вмешательства)  
2: critical (критическое состояние)  
3: errors (ошибки)  
4: warning (предупреждения)  
5: notice (уведомления)  
6: info (информационные сообщения)  
7: debug (отладочные сообщения)  

Когда вы указываете код важности, journalctl выведет все сообщения с этим кодом и выше. Например если мы укажем опцию -p 2, journalctl покажет все сообщения с уровнями 2, 1 и 0.



## Просмотр журналов загрузки

Если journald был настроен на постоянное хранение журналов, мы можем просматривать журналы логов по каждой отдельной загрузке, следующая команда выведет список журналов:
```
journalctl --list-boots
root@os3:~# journalctl --list-boots
-11 477da6c3a3e04f89822df4b2e7e1c7dd Tue 2022-06-07 11:55:04 UTC—Tue 2022-06-07 11:55:35 UTC
-10 606a147122db4f80aee333632c35810a Thu 2022-10-06 12:00:53 UTC—Thu 2022-10-06 16:22:37 UTC
 -9 46ad511ac8cb4f0cb652f49d143b7b2e Mon 2022-10-10 06:29:15 UTC—Mon 2022-10-10 15:51:15 UTC
 -8 34b100e1cbab45c88afa3dd262a1f075 Tue 2022-10-11 05:49:14 UTC—Tue 2022-10-11 06:03:02 UTC
 -7 7e4bf94c82bf4751b2b67514fdb8dfdf Tue 2022-10-11 06:16:42 UTC—Tue 2022-10-11 06:30:02 UTC
 -6 a55949b55ba7420f9f68f2716c835b80 Tue 2022-10-11 07:27:44 UTC—Tue 2022-10-11 07:36:13 UTC
 -5 0de98bb3e3964f01943c6ea30aa21e66 Tue 2022-10-11 07:36:46 UTC—Tue 2022-10-11 09:48:28 UTC
 -4 a46ff741fa8e4a59bf3a464b4f548002 Tue 2022-10-11 11:44:09 UTC—Tue 2022-10-11 12:09:35 UTC
 -3 8064fb13a8c04a66a6db7245a53a3a11 Tue 2022-10-11 12:19:00 UTC—Tue 2022-10-11 12:26:19 UTC
 -2 d836151f2ff948129a550e44da6e3b90 Mon 2022-10-17 06:36:22 UTC—Mon 2022-10-17 08:32:15 UTC
 -1 7f5139b8b895491495875e67652558e7 Tue 2022-10-18 08:49:43 UTC—Tue 2022-10-18 08:52:57 UTC
  0 64b7ca6b7e1349a69138c02483e928a0 Sat 2022-11-26 15:42:10 UTC—Sun 2022-11-27 10:07:22 UTC

```


Первый номер показывает номер журнала, который можно использовать для просмотра журнала определенной сессии. Второй номер boot ID так же можно использовать для просмотра отдельного журнала.

Следующие две даты, промежуток времени в течении которого в него записывались логи, это удобно если вы хотите найти логи за определенный период.

Например, чтобы просмотреть журнал начиная с текущего старта системы, можно использовать команду:
```
journalctl -b 0
```
А для того, чтобы просмотреть журнал предыдущей загрузки:
```
journalctl -b -1
```

## Просмотр журнала за определенный период времени

Journalctl позволяет использовать такие служебные слова как “yesterday” (вчера), “today” (сегодня), “tomorrow” (завтра), или “now” (сейчас).

Поэтому мы можем использовать опцию "--since" (с начала какого периода выводить журнал).

С определенной даты и времени:
```
journalctl --since "2020-12-18 06:00:00"
```
С определенной даты и по определенное дату и время:
```
journalctl --since "2020-12-17" --until "2020-12-18 10:00:00
```
Со вчерашнего дня:
```
journalctl --since yesterday
```
С 9 утра и до момента, час назад:
```
journalctl --since 09:00 --until "1 hour ago"
```

## Просмотр сообщений ядра

Чтобы просмотреть сообщения от ядра Linux за текущую загрузку, используйте команду с ключом -k:
```
journalctl -k
root@os3:~# journalctl -k
-- Logs begin at Tue 2022-06-07 11:55:04 UTC, end at Sun 2022-11-27 10:11:02 UTC. --
Nov 26 15:42:10 os3 kernel: Linux version 5.4.0-110-generic (buildd@ubuntu) (gcc version 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04.1)) #124-Ubuntu SMP Thu Apr 14 19:46:19 UTC 2022 (Ubuntu 5.4.0-110.124-generic 5.4.181)
Nov 26 15:42:10 os3 kernel: Command line: BOOT_IMAGE=/vmlinuz-5.4.0-110-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
Nov 26 15:42:10 os3 kernel: KERNEL supported cpus:
Nov 26 15:42:10 os3 kernel:   Intel GenuineIntel
Nov 26 15:42:10 os3 kernel:   AMD AuthenticAMD
Nov 26 15:42:10 os3 kernel:   Hygon HygonGenuine
Nov 26 15:42:10 os3 kernel:   Centaur CentaurHauls
Nov 26 15:42:10 os3 kernel:   zhaoxin   Shanghai
Nov 26 15:42:10 os3 kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
Nov 26 15:42:10 os3 kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
Nov 26 15:42:10 os3 kernel: x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
Nov 26 15:42:10 os3 kernel: x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
Nov 26 15:42:10 os3 kernel: x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
Nov 26 15:42:10 os3 kernel: BIOS-provided physical RAM map:
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x0000000000100000-0x000000007ffeffff] usable
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x000000007fff0000-0x000000007fffffff] ACPI data
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x00000000fec00000-0x00000000fec00fff] reserved
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x00000000fee00000-0x00000000fee00fff] reserved
Nov 26 15:42:10 os3 kernel: BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Nov 26 15:42:10 os3 kernel: NX (Execute Disable) protection: active
Nov 26 15:42:10 os3 kernel: SMBIOS 2.5 present.
Nov 26 15:42:10 os3 kernel: DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
Nov 26 15:42:10 os3 kernel: Hypervisor detected: KVM
Nov 26 15:42:10 os3 kernel: kvm-clock: Using msrs 4b564d01 and 4b564d00
Nov 26 15:42:10 os3 kernel: kvm-clock: cpu 0, msr 23601001, primary cpu clock
Nov 26 15:42:10 os3 kernel: kvm-clock: using sched offset of 6568915503 cycles
....
```

## Просмотр журнала логов для определенного сервиса systemd или приложения

Вы можете отфильтровать логи по определенному сервису systemd. Например, что бы просмотреть логи от NetworkManager, можно использовать следующую команду:
```
journalctl -u NetworkManager.service
```
Если нужно найти название сервиса, используйте команду:
```
systemctl list-units --type=service
```
Так же можно просмотреть лог приложения, указав его исполняемый файл, например чтобы просмотреть все сообщения от nginx за сегодня, мы можем использовать команду:
```
journalctl /usr/sbin/nginx --since today
```
Например, если ваш процесс Nginx использует единицу PHP-FPM для обработки динамического контента, вы можете объединить их данные в хронологическом порядке, указав обе единицы:
```
journalctl -u nginx.service -u php-fpm.service --since today
```
Или указав конкретный PID:
```
journalctl _PID=1
```
Все записи для полльзователя или группы. Это можно сделать с помощью фильтров _UID или _GID. Например, если ваш веб-сервер работает под именем пользователя www-data, вы можете найти идентификатор пользователя с помощью следующей команды:
```
id -u www-data
33
```
После этого, вы сможете использовать возвращенный идентификатор для фильтрации результатов журнальной системы:
```
journalctl _UID=33 --since today
```
## Дополнительные опции просмотра

Следить за появлением новых сообщений (аналог tail -f):
```
journalctl -f
```
Следить за появлением новых сообщений сервиса:
```
journalctl -fu name.service
```
Открыть журнал «перемотав» его к последней записи:
```
journalctl -e
```
Открыть журнал «перемотав» его к последней записи и добавить к информации об ошибках пояснения, ссылки на документацию или форумы там, где это возможно;
```
journalctl -xe
```
Если в каталоге с журналами очень много данных, то фильтрация вывода journalctl может занять некоторое время, процесс можно значительно ускорить с помощью опции --file, указав journalctl только нужный нам журнал, за которым мы хотим следить:
```
journalctl --file /var/log/journal/e02689e50bc240f0bb545dd5940ac213/system.journal -f
```
По умолчанию journalctl отсекает части строк, которые не вписываются в экран по ширине, хотя иногда перенос строк может оказаться более предпочтительным. Управление этой возможностью производится посредством переменной окружения SYSTEMD_LESS, в которой содержатся опции, передаваемые в less (программу постраничного просмотра, используемую по умолчанию). По умолчанию переменная имеет значение FRSXMK, если убрать опцию S, строки не будут обрезаться.

Например:
```
SYSTEMD_LESS=FRXMK journalctl
```

## Форматы вывода
 Журнал можно выводить в множестве разных форматов. Для этого вы можете использовать опцию -o вместе с указателем формата.
 ```
 journalctl -b -u nginx -o json
 ```
Для отображения можно использовать следующие форматы:

* cat: отображает только само поле сообщения.  
* export: двоичный формат, подходящий для передачи или резервного копирования.  
* json: стандартный формат JSON с одной записью на строку.  
* json-pretty: код JSON в формате, более удобном для чтения человеком  
* json-sse: вывод в формате JSON в оболочке, совместимой с операцией add   server-sent event  
* short: вывод в формате syslog по умолчанию  
* short-iso: формат по умолчанию, дополненный для отображения временных меток часов ISO 8601.  
* short-monotonic: формат по умолчанию с однотонными временными метками.  
* short-precise: формат по умолчанию с точностью до микросекунд
verbose: показывает все поля журнальной системы, доступные для ввода, в том числе те, которые обычно скрыты на внутреннем уровне.  


Информация о сервисе systemd-journald.service - здесь также можно увидеть сколько занимает места журнал и его максимальный размер
```
root@os3:~# journalctl -u systemd-journald.service
Nov 26 15:42:10 os3 systemd-journald[360]: Journal started
Nov 26 15:42:10 os3 systemd-journald[360]: Runtime Journal (/run/log/journal/10ac0529b1ef4e1c85f504f698893be7) is 2.4M, max 19.8M, 17.3M free.
Nov 26 15:42:10 os3 systemd-journald[360]: Time spent on flushing to /var/log/journal/10ac0529b1ef4e1c85f504f698893be7 is 150.880ms for 527 entries.
Nov 26 15:42:10 os3 systemd-journald[360]: System Journal (/var/log/journal/10ac0529b1ef4e1c85f504f698893be7) is 168.0M, max 3.0G, 2.8G free.
Nov 27 10:54:30 os3 systemd-journald[360]: Journal stopped
Nov 27 10:54:30 os3 systemd-journald[3175]: Journal started
Nov 27 10:54:30 os3 systemd-journald[3175]: System Journal (/var/log/journal/10ac0529b1ef4e1c85f504f698893be7) is 288.0M, max 3.0G, 2.7G free.
Nov 27 10:54:30 os3 systemd-journald[3175]: System Journal (/var/log/journal/10ac0529b1ef4e1c85f504f698893be7) is 288.0M, max 3.0G, 2.7G free.
```