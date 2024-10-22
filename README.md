# nfqws-keenetic

Пакеты для установки `nfqws` на маршрутизаторы с поддержкой `opkg`.

> Данный материал подготовлен в научно-технических целях.
> Использование предоставленных материалов в целях отличных от ознакомления может являться нарушением действующего законодательства.
> Автор не несет ответственности за неправомерное использование данного материала.

> **Вы пользуетесь этой инструкцией на свой страх и риск!**
> 
> Автор не несёт ответственности за порчу оборудования и программного обеспечения, проблемы с доступом и потенцией.
> Подразумевается, что вы понимаете, что вы делаете.

Изначально написано для роутеров Keenetic с установленным entware.
Однако, работоспособность также была проверена на прошивках Padavan и OpenWRT (читайте ниже).

Списки проверенного оборудования собираем в [отдельной теме](https://github.com/Anonym-tsk/nfqws-keenetic/discussions/1).

Поделиться опытом можно в разделе [Discussions](https://github.com/Anonym-tsk/nfqws-keenetic/discussions) или в [чате](https://t.me/nfqws).

Если nfqws работает как-то не так, можете попробовать [tpws](https://github.com/Anonym-tsk/tpws-keenetic).

### Что это?

`nfqws` - утилита для модификации TCP соединения на уровне пакетов, работает через обработчик очереди NFQUEUE и raw сокеты.

Почитать подробнее можно на [странице авторов](https://github.com/bol-van/zapret) (ищите по ключевому слову `nfqws`).

### Подготовка Keenetic

- Прочитайте инструкцию полностью, прежде, чем начать что-то делать!

- Рекомендуется игнорировать предложенные провайдером адреса DNS-серверов. Для этого в интерфейсе роутера отметьте пункты ["игнорировать DNS от провайдера"](https://help.keenetic.com/hc/ru/articles/360008609399) в настройках IPv4 и IPv6.
 
- Вместе с этим рекомендуется [настроить использование DoT/DoH](https://help.keenetic.com/hc/ru/articles/360007687159).

- Установить entware на маршрутизатор по инструкции [на встроенную память роутера](https://help.keenetic.com/hc/ru/articles/360021888880) или [на USB-накопитель](https://help.keenetic.com/hc/ru/articles/360021214160).

- Через web-интерфейс Keenetic установить пакеты **Протокол IPv6** (**Network functions > IPv6**) и **Модули ядра подсистемы Netfilter** (**OPKG > Kernel modules for Netfilter** - не путать с "Netflow"). Обратите внимание, что второй компонент отобразится в списке пакетов только после того, как вы отметите к установке первый.

- В разделе "Интернет-фильтры" отключить все сторонние фильтры (NextDNS, SkyDNS, Яндекс DNS и другие).

- Все дальнейшие команды выполняются не в cli роутера, а **в среде entware**. Подключиться в неё можно несколькими способами:
  - Через telnet: в терминале выполнить `telnet 192.168.1.1`, а потом `exec sh`.
  - Или же подключиться напрямую через SSH (логин - `root`, пароль по умолчанию - `keenetic`, порт - 222 или 22). Для этого в терминале написать `ssh 192.168.1.1 -l root -p 222`.

> **Миграция с версии 1.x.x на 2.x.x:**
> 
> Для определения версии выполните команду `opkg info nfqws-keenetic` - она работает только на версиях 2.x.x и возвращает информацию о пакете.
> Если ничего не вернула – у вас установлена старая версия.
>
> Никакой специальной миграции не требуется, просто переустановите новую версию по инструкции ниже.

---

### Установка на Keenetic и Entware

1. Установите необходимые зависимости
   ```
   opkg update
   opkg install ca-certificates wget-ssl
   opkg remove wget-nossl
   ```

2. Установите opkg-репозиторий в систему
   ```
   mkdir -p /opt/etc/opkg
   echo "src/gz nfqws-keenetic https://anonym-tsk.github.io/nfqws-keenetic/all" > /opt/etc/opkg/nfqws-keenetic.conf
   ```
   Репозиторий универсальный, поддерживаемые архитектуры: `mipsel`, `mips`, `aarch64`, `armv7`, `x86`, `x86_64`.

   <details>
     <summary>Или можете выбрать репозиторий под конкретную архитектуру</summary>

     - `mips-3.4` <sub><sup>Keenetic Giga SE (KN-2410), Ultra SE (KN-2510), DSL (KN-2010), Launcher DSL (KN-2012), Duo (KN-2110), Skipper DSL (KN-2112), Hopper DSL (KN-3610)</sup></sub>
       ```
       mkdir -p /opt/etc/opkg
       echo "src/gz nfqws-keenetic https://anonym-tsk.github.io/nfqws-keenetic/mips" > /opt/etc/opkg/nfqws-keenetic.conf
       ```

     - `mipsel-3.4` <sub><sup>Keenetic Giga (KN-1010/1011), Ultra (KN-1810), Viva (KN-1910/1912), Hero 4G (KN-2310), Hero 4G+ (KN-2311), Giant (KN-2610), Skipper 4G (KN-2910), Hopper (KN-3810)</sup></sub>
       ```
       mkdir -p /opt/etc/opkg
       echo "src/gz nfqws-keenetic https://anonym-tsk.github.io/nfqws-keenetic/mipsel" > /opt/etc/opkg/nfqws-keenetic.conf
       ```

     - `aarch64-3.10` <sub><sup>Keenetic Peak (KN-2710), Ultra (KN-1811), Hopper SE (KN-3812), Keenetic Giga (KN-1012)</sup></sub>
       ```
       mkdir -p /opt/etc/opkg
       echo "src/gz nfqws-keenetic https://anonym-tsk.github.io/nfqws-keenetic/aarch64" > /opt/etc/opkg/nfqws-keenetic.conf
       ```
   </details>

3. Установите пакет
   ```
   opkg update
   opkg install nfqws-keenetic
   ```

4. Установите веб-интерфейс (опционально, только для Keenetic)
   ```
   opkg install nfqws-keenetic-web
   ```

##### Обновление

```
opkg update
opkg upgrade nfqws-keenetic nfqws-keenetic-web
```

##### Удаление

```
opkg remove nfqws-keenetic nfqws-keenetic-web
```

##### Информация об установленной версии

```
opkg info nfqws-keenetic
opkg info nfqws-keenetic-web
```

---

### Установка на OpenWRT

Пакет работает только с `iptables`.
Если в вашей системе используется `nftables`, придется удалить `nftables` и `firewall4`, и установить `firewall3` и `iptables`.

Проверить, что ваша система использует `nftables`:
```
ls -la /sbin/fw4
which nft
```

1. Установите необходимые зависимости
   ```
   opkg update
   opkg install ca-certificates wget-ssl
   opkg remove wget-nossl
   ```

2. Установите публичный ключ репозитория
   ```
   wget -O "/tmp/nfqws-keenetic.pub" "https://anonym-tsk.github.io/nfqws-keenetic/openwrt/nfqws-keenetic.pub"
   opkg-key add /tmp/nfqws-keenetic.pub
   ```

3. Установите opkg-репозиторий в систему
   ```
   echo "src/gz nfqws-keenetic https://anonym-tsk.github.io/nfqws-keenetic/openwrt" > /etc/opkg/nfqws-keenetic.conf
   ```
   Репозиторий универсальный, поддерживаемые архитектуры: `mipsel`, `mips`, `aarch64`, `armv7`, `x86`, `x86_64`.
   Для добавления поддержки новых устройств, [создайте Feature Request](https://github.com/Anonym-tsk/nfqws-keenetic/issues/new?template=feature_request.md&title=%5BFeature+request%5D+)

4. Установите пакет
   ```
   opkg update
   opkg install nfqws-keenetic
   ```

> NB: Все пути файлов, описанные в этой инструкции, начинающиеся с `/opt`, на OpenWRT будут начинаться с корня `/`.
> Например конфиг расположен в `/etc/nfqws/nfqws.conf`
> 
> Для запуска/остановки используйте команду `service nfqws-keenetic {start|stop|restart|reload|status}`

---

### Настройки

Файл настроек расположен по пути `/opt/etc/nfqws/nfqws.conf`. Для редактирования можно воспользоваться встроенным редактором `vi` или установить `nano`.

```
# Интерфейс провайдера. Обычно `eth3` или `eth2.2` для проводного соединения, и `ppp0` для PPPoE
# Заполняется автоматически при установке
# Можно ввести несколько интерфейсов, например ISP_INTERFACE="eth3 nwg1"
ISP_INTERFACE="eth3"

# Стратегии обработки HTTPS и QUIC трафика
NFQWS_ARGS="--dpi-desync=fake,split2 --dpi-desync-ttl=0 --dpi-desync-repeats=16 --dpi-desync-split-pos=1 --dpi-desync-fooling=md5sig,badseq --dpi-desync-cutoff=d4 --dpi-desync-fake-tls=/opt/etc/nfqws/tls_clienthello.bin"
NFQWS_ARGS_QUIC="--filter-udp=443 --dpi-desync=fake --dpi-desync-repeats=11 --dpi-desync-cutoff=d4 --dpi-desync-fake-quic=/opt/etc/nfqws/quic_initial.bin"

# Стратегия обработки UDP трафика (не использует параметры из NFQWS_EXTRA_ARGS)
NFQWS_ARGS_UDP="--filter-udp=50000-50099 --dpi-desync=fake --dpi-desync-any-protocol --dpi-desync-repeats=6 --dpi-desync-cutoff=n2"

# Режим работы (auto, list, all)
NFQWS_EXTRA_ARGS="--hostlist=/opt/etc/nfqws/user.list --hostlist-auto=/opt/etc/nfqws/auto.list --hostlist-auto-debug=/opt/var/log/nfqws.log --hostlist-exclude=/opt/etc/nfqws/exclude.list"

# Обрабатывать ли IPv6 соединения
IPV6_ENABLED=1

# TCP порты для iptables
# Оставьте пустым, если нужно отключить обработку TCP
# Добавьте порт 80 для обработки HTTP (TCP_PORTS=443,80)
TCP_PORTS=443

# UDP порты для iptables
# Оставьте пустым, если нужно отключить обработку UDP
# Удалите порт 443, если не нужно обрабатывать QUIC
UDP_PORTS=443,50000:50099

# Логирование в Syslog
LOG_LEVEL=0
```

Стратегии применяются ко всем доменам из `user.list` и `auto.list`, за исключением доменов из `exclude.list`.
В конфиге есть 3 варианта параметра `NFQWS_EXTRA_ARGS` - это режим работы nfqws:
- В режиме `list` будут обрабатываться только домены из файла `user.list`
- В режиме `auto` кроме этого будут автоматически определяться недоступные домены и добавляться в список, по которому `nfqws` обрабатывает трафик. Домен будет добавлен, если за 60 секунд будет 3 раза определено, что ресурс недоступен
- В режиме `all` будет обрабатываться весь трафик кроме доменов из списка `exclude.list`

---

### Полезное

1. Конфиг-файл `/opt/etc/nfqws/nfqws.conf`
2. Скрипт запуска/остановки `/opt/etc/init.d/S51nfqws {start|stop|restart|reload|status}`
3. Вручную добавить домены в список можно в файле `/opt/etc/nfqws/user.list` (один домен на строке, поддомены учитываются автоматически)
4. Автоматически добавленные домены `/opt/etc/nfqws/auto.list`
5. Лог автоматически добавленных доменов `/opt/var/log/nfqws.log`
6. Домены-исключения `/opt/etc/nfqws/exclude.list` (один домен на строке, поддомены учитываются автоматически)
7. Проверить, что нужные правила добавлены в таблицу маршрутизации `iptables-save | grep "queue-num 200"`
   > Вы должны увидеть похожие строки
   > ```
   > -A POSTROUTING -o eth3 -p tcp -m tcp --dport 443 -m connbytes --connbytes 1:6 --connbytes-mode packets --connbytes-dir original -m mark ! --mark 0x40000000/0x40000000 -j NFQUEUE --queue-num 200 --queue-bypass
   > ```

### Если ничего не работает...

1. Если ваше устройство поддерживает аппаратное ускорение (flow offloading, hardware nat, hardware acceleration), то iptables могут не работать.
   При включенном offloading пакет не проходит по обычному пути netfilter.
   Необходимо или его отключить, или выборочно им управлять.
2. На Keenetic можно попробовать выключить или наоборот включить [сетевой ускоритель](https://help.keenetic.com/hc/ru/articles/214470905)
3. Возможно, стоит выключить службу классификации трафика IntelliQOS.
4. Можно попробовать отключить IPv6 на сетевом интерфейсе провайдера через веб-интерфейс маршрутизатора.
5. Можно попробовать запретить весь UDP трафик на 443 порт для отключения QUIC:
   > Межсетевой экран → Домашняя сеть → Добавить правило<br/>
   > Включить правило: Включено<br/>
   > Описание: Блокировать QUIC<br/>
   > Действие: Запретить<br/>
   > Протокол: UDP<br/>
   > Номер порта назначения: Равен 443<br/>
   > Остальные параметры оставляем без изменений

### Частые проблемы
1. `iptables: No chain/target/match by that name`<br/>
   Не установлен пакет "Модули ядра подсистемы Netfilter". На Keenetic он появляется в списке пакетов только после установки "Протокол IPv6"
2. `can't initialize ip6tables table` и/или `Perhaps ip6tables or your kernel needs to be upgraded`<br/>
   Не установлен пакет "Протокол IPv6". Также, проблема может появляться на старых прошивках 2.xx, выключите поддержку IPv6 в конфиге NFQWS
3. Ошибки вида `readlink: not found`, `dirname: not found`<br/>
   Обычно возникают не на кинетиках. Решение - установить busybox: `opkg install busybox` или отдельно пакеты `opkg install coreutils-readlink coreutils-dirname`

### Как использовать несколько стратегий

По-умолчанию, параметры для запуска `nfqws` формируются из двух переменных – `$NFQWS_ARGS $NFQWS_EXTRA_ARGS`.<br/>
Если вы хотите использовать несколько стратегий, можно разделять их параметром `--new`.
Например, стратегия ниже применит опцию `--dpi-desync=fake,split2` для HTTPS запросов к доменам из `custom.list`,
а для всех остальных, соответствующих настройке `NFQWS_EXTRA_ARGS`, будет использовать `--dpi-desync=disorder2 --dpi-desync-fooling=md5sig,badseq`:
```
NFQWS_ARGS="--filter-tcp=443 --dpi-desync=fake,split2 --hostlist=custom.list --new --dpi-desync=disorder2 --dpi-desync-fooling=md5sig,badseq"
```

### Как подобрать рабочую стратегию NFQWS

1. Скачать скрипт
   ```
   opkg install curl
   cd ~
   wget -O "strategy.sh" "https://raw.githubusercontent.com/Anonym-tsk/nfqws-keenetic/master/common/strategy.sh"
   chmod +x strategy.sh
   ```

2. Запустить
   ```
   ./strategy.sh www.mos.ru
   ```
   или так
   ```
   ./strategy.sh www.mos.ru --full
   ```
   где первым аргументом указывается домен, который вы хотите проверить (без https://),
   вторым можно указать параметр `--full` для полного перебора стратегий, если быстрый перебор ничего не нашел.

3. Найденную стратегию вписать в конфиге `/opt/etc/nfqws/nfqws.conf` в параметр `NFQWS_ARGS`

---

Нравится проект? [Поддержи автора](https://yoomoney.ru/to/410019180291197)! Купи ему немного :beers: или :coffee:!
