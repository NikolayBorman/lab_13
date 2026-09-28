# Настройка NAT для IPv4

## Топология

<img width="1110" height="335" alt="image" src="https://github.com/user-attachments/assets/0af1dbce-0475-4303-9457-8e7d52283a72" />


## Таблица адресации

| Устройство | Интерфейс | IP-адрес        | Маска подсети   |
|------------|-----------|-----------------|-----------------|
| R1         | G0/0/0    | 209.165.200.230 | 255.255.255.248 |
| R1         | G0/0/1    | 192.168.1.1     | 255.255.255.0   |
| R2         | G0/0/0    | 209.165.200.225 | 255.255.255.248 |
| R2         | Lo1       | 209.165.200.1   | 255.255.255.224 |
| S1         | VLAN 1    | 192.168.1.11    | 255.255.255.0   |
| S2         | VLAN 1    | 192.168.1.12    | 255.255.255.0   |
| PC-A       | NIC       | 192.168.1.2     | 255.255.255.0   |
| PC-B       | NIC       | 192.168.1.3     | 255.255.255.0   |

## Цели
- Часть 1. Создание сети и настройка основных параметров устройства
- Часть 2. Настройка и проверка NAT для IPv4
- Часть 3. Настройка и проверка PAT для IPv4
- Часть 4. Настройка и проверка статического NAT для IPv4

## Необходимые ресурсы
- 2 маршрутизатора Cisco 4221 (IOS XE 16.9.4)
- 2 коммутатора Cisco 2960 (IOS 15.2(2) lanbasek9)
- 2 ПК с Windows и программой эмуляции терминала
- Консольные кабели, Ethernet-кабели

---

## Часть 1. Создание сети и настройка основных параметров устройств

### 1.1 Базовая настройка R1

```cisco
Router> enable
Router# configure terminal
Router(config)# no ip domain-lookup
Router(config)# hostname R1
R1(config)# enable secret class
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# logging synchronous
R1(config-line)# exec-timeout 0 0
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exec-timeout 0 0
R1(config-line)# exit
R1(config)# service password-encryption
R1(config)# banner motd ^CUnauthorized access prohibited^C
R1(config)# interface g0/0/0
R1(config-if)# ip address 209.165.200.230 255.255.255.248
R1(config-if)# no shutdown
R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
R1(config-if)# exit
R1(config)# interface g0/0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
R1(config-if)# exit
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.225
R1(config)# end
R1# copy running-config startup-config
```

### 1.2 Базовая настройка R2

```cisco
Router> enable
Router# configure terminal
Router(config)# no ip domain-lookup
Router(config)# hostname R2
R2(config)# enable secret class
R2(config)# line console 0
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# logging synchronous
R2(config-line)# exec-timeout 0 0
R2(config-line)# exit
R2(config)# line vty 0 4
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# exec-timeout 0 0
R2(config-line)# exit
R2(config)# service password-encryption
R2(config)# banner motd ^CUnauthorized access prohibited^C
R2(config)# interface g0/0/0
R2(config-if)# ip address 209.165.200.225 255.255.255.248
R2(config-if)# no shutdown
R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
R2(config-if)# exit
R2(config)# interface loopback 1
R2(config-if)#
%LINK-3-UPDOWN: Interface Loopback1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up
R2(config-if)# ip address 209.165.200.1 255.255.255.224
R2(config-if)# exit
R2(config)# end
R2# copy running-config startup-config
```

### 1.3 Базовая настройка S1

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname S1
S1(config)# no ip domain-lookup
S1(config)# enable secret class
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# logging synchronous
S1(config-line)# exit
S1(config)# line vty 0 15
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# service password-encryption
S1(config)# banner motd ^CUnauthorized access prohibited^C
S1(config)# interface vlan 1
S1(config-if)# ip address 192.168.1.11 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)#
%LINK-3-UPDOWN: Interface Vlan1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.1.1
S1(config)# end
S1# copy running-config startup-config
```

### 1.4 Базовая настройка S2

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname S2
S2(config)# no ip domain-lookup
S2(config)# enable secret class
S2(config)# line console 0
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# logging synchronous
S2(config-line)# exit
S2(config)# line vty 0 15
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# exit
S2(config)# service password-encryption
S2(config)# banner motd ^CUnauthorized access prohibited^C
S2(config)# interface vlan 1
S2(config-if)# ip address 192.168.1.12 255.255.255.0
S2(config-if)# no shutdown
S2(config-if)#
%LINK-3-UPDOWN: Interface Vlan1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.1.1
S2(config)# end
S2# copy running-config startup-config
```

### 1.5 Настройка ПК

- PC-A: 192.168.1.2/24, шлюз 192.168.1.1
- PC-B: 192.168.1.3/24, шлюз 192.168.1.1

### 1.6 Проверка базовой связности

```cisco
R1#ping 209.165.200.225

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.225, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R1#ping 209.165.200.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

---

## Часть 2. Настройка и проверка NAT для IPv4

### 2.1 Настройка NAT на R1 (пул 209.165.200.226 – 209.165.200.228)

```cisco
R1> enable
R1# configure terminal
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)# ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248
R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS
R1(config)# interface g0/0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0/0
R1(config-if)# ip nat outside
R1(config-if)# end
R1# copy running-config startup-config
```

### 2.2 Проверка NAT
#### PC-B
```cmd
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
```cisco
R1#show ip nat translations 
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.3:1      209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:2 192.168.1.3:2      209.165.200.1:2    209.165.200.1:2
icmp 209.165.200.226:3 192.168.1.3:3      209.165.200.1:3    209.165.200.1:3
icmp 209.165.200.226:4 192.168.1.3:4      209.165.200.1:4    209.165.200.1:4
icmp 209.165.200.226:5 192.168.1.3:5      209.165.200.1:5    209.165.200.1:5
icmp 209.165.200.226:6 192.168.1.3:6      209.165.200.1:6    209.165.200.1:6
icmp 209.165.200.226:7 192.168.1.3:7      209.165.200.1:7    209.165.200.1:7
icmp 209.165.200.226:8 192.168.1.3:8      209.165.200.1:8    209.165.200.1:8
```

**Вопрос:** Во что был транслирован внутренний локальный адрес PC-B?  
**Ответ:** `192.168.1.3` был транслирован в `209.165.200.226`.

**Вопрос:** Какой тип адреса NAT является переведённым адресом?  
**Ответ:** Внутренний глобальный (Inside global).

#### PC-A 
```cmd
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
```cisco
R1#show ip nat translations 
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.2:1      209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:2 192.168.1.2:2      209.165.200.1:2    209.165.200.1:2
icmp 209.165.200.226:3 192.168.1.2:3      209.165.200.1:3    209.165.200.1:3
icmp 209.165.200.226:4 192.168.1.2:4      209.165.200.1:4    209.165.200.1:4
```

#### S1 
```cisco
S1#ping 209.165.200.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
```cisco
R1#show ip nat translations 
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:10192.168.1.11:10    209.165.200.1:10   209.165.200.1:10
icmp 209.165.200.226:2 192.168.1.11:2     209.165.200.1:2    209.165.200.1:2
icmp 209.165.200.226:3 192.168.1.11:3     209.165.200.1:3    209.165.200.1:3
icmp 209.165.200.226:4 192.168.1.11:4     209.165.200.1:4    209.165.200.1:4
icmp 209.165.200.226:5 192.168.1.11:5     209.165.200.1:5    209.165.200.1:5
icmp 209.165.200.226:6 192.168.1.11:6     209.165.200.1:6    209.165.200.1:6
icmp 209.165.200.226:7 192.168.1.11:7     209.165.200.1:7    209.165.200.1:7
icmp 209.165.200.226:8 192.168.1.11:8     209.165.200.1:8    209.165.200.1:8
icmp 209.165.200.226:9 192.168.1.11:9     209.165.200.1:9    209.165.200.1:9
```

#### S2  
Ожидаемый сбой
```cisco
S2#ping 209.165.200.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.1, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```
```cisco
R1#show ip nat statistics 
Total translations: 0 (0 static, 0 dynamic, 0 extended)
Outside Interfaces: GigabitEthernet0/0/0
Inside Interfaces: GigabitEthernet0/0/1
Hits: 8  Misses: 16
Expired translations: 12
Dynamic mappings:
-- Inside Source
access-list 1 pool PUBLIC_ACCESS refCount 0
 pool PUBLIC_ACCESS: netmask 255.255.255.248
       start 209.165.200.226 end 209.165.200.228
       type generic, total addresses 3 , allocated 0 (0%), misses 4
```

**Причина:** в пуле всего 3 адреса, а попытка четвёртого устройства — превышение лимита. NAT работает по принципу «один-к-одному».

#### Просмотр времени жизни трансляции
К сожалению, CPT  не поддерживает ключ verbose  команды show ip nat translations.
Время жизни трансляций: 24 часа для NAT «один-к-одному», 1 минута для PAT (по умолчанию IOS).
```cisco
R1#show ip nat translations verbose
                            ^
% Invalid input detected at '^' marker.
```


#### Очистка перед PAT

```cisco
R1#clear ip nat translation *
```

---

## Часть 3. Настройка и проверка PAT для IPv4

### 3.1 PAT с использованием пула адресов

#### Удаление старой команды NAT
```cisco
R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS
```

#### Настройка PAT (overload)
```cisco
R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS overload
```

#### Проверка
##### PC-B
```cmd
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

```
```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.228:5 192.168.1.3:5      209.165.200.1:5    209.165.200.1:5
icmp 209.165.200.228:6 192.168.1.3:6      209.165.200.1:6    209.165.200.1:6
icmp 209.165.200.228:7 192.168.1.3:7      209.165.200.1:7    209.165.200.1:7
icmp 209.165.200.228:8 192.168.1.3:8      209.165.200.1:8    209.165.200.1:8
```

**Вопрос:** Во что был транслирован внутренний локальный адрес PC-B?  
**Ответ:** `192.168.1.3:1` → `209.165.200.226:1`.

**Вопрос:** Какой тип адреса NAT является переведённым адресом?  
**Ответ:** Внутренний глобальный с портом (Inside global + port).

**Вопрос:** Чем отличаются выходные данные от упражнения NAT?  
**Ответ:** В PAT используется один и тот же IP-адрес с разными портами для разных сессий. Время трансляции — 1 минута вместо 24 часов.

#### Проверка
##### PC-A 
```cmd
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.228:5 192.168.1.2:5      209.165.200.1:5    209.165.200.1:5
icmp 209.165.200.228:6 192.168.1.2:6      209.165.200.1:6    209.165.200.1:6
icmp 209.165.200.228:7 192.168.1.2:7      209.165.200.1:7    209.165.200.1:7
icmp 209.165.200.228:8 192.168.1.2:8      209.165.200.1:8    209.165.200.1:8
```

Bспользуется тот же глобальный адрес `209.165.200.228`, но с другим портом при следующей сессии.

#### Одновременный трафик (PC-A и PC-B: `ping -t 209.165.200.1`)

```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.228:1024 192.168.1.3:9      209.165.200.1:9    209.165.200.1:1024
icmp 209.165.200.228:1025 192.168.1.3:10     209.165.200.1:10   209.165.200.1:1025
icmp 209.165.200.228:1026 192.168.1.3:11     209.165.200.1:11   209.165.200.1:1026
icmp 209.165.200.228:1027 192.168.1.3:12     209.165.200.1:12   209.165.200.1:1027
icmp 209.165.200.228:10 192.168.1.2:10     209.165.200.1:10   209.165.200.1:10
icmp 209.165.200.228:11 192.168.1.2:11     209.165.200.1:11   209.165.200.1:11
icmp 209.165.200.228:12 192.168.1.2:12     209.165.200.1:12   209.165.200.1:12
icmp 209.165.200.228:9 192.168.1.2:9      209.165.200.1:9    209.165.200.1:9
```

**Вопрос:** Как маршрутизатор отслеживает, куда идут ответы?  
**Ответ:** По номерам портов, например, 228:11 для PC-A и 228:1024 для PC-B.

#### Останавливаем пинги на ПК и чистим R1


```cisco
R1# clear ip nat translation *
```

### 3.2 PAT с перегрузкой интерфейса (interface overload)

#### Удаление пула и старой команды
```cisco
R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS overload
R1(config)# no ip nat pool PUBLIC_ACCESS
```

#### Настройка PAT на интерфейс
```cisco
R1(config)# ip nat inside source list 1 interface g0/0/0 overload
```

#### Проверка
##### PC-B
```cmd
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.230:13 192.168.1.3:13     209.165.200.1:13   209.165.200.1:13
icmp 209.165.200.230:14 192.168.1.3:14     209.165.200.1:14   209.165.200.1:14
icmp 209.165.200.230:15 192.168.1.3:15     209.165.200.1:15   209.165.200.1:15
icmp 209.165.200.230:16 192.168.1.3:16     209.165.200.1:16   209.165.200.1:16
```

#### Множественный трафик (PC-A, PC-B, S1, S2)

```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.230:1024 192.168.1.12:6     209.165.200.1:6    209.165.200.1:1024
icmp 209.165.200.230:1025 192.168.1.12:7     209.165.200.1:7    209.165.200.1:1025
icmp 209.165.200.230:1026 192.168.1.12:8     209.165.200.1:8    209.165.200.1:1026
icmp 209.165.200.230:1027 192.168.1.12:9     209.165.200.1:9    209.165.200.1:1027
icmp 209.165.200.230:1028 192.168.1.12:10    209.165.200.1:10   209.165.200.1:1028
icmp 209.165.200.230:10 192.168.1.11:10    209.165.200.1:10   209.165.200.1:10
icmp 209.165.200.230:13 192.168.1.2:13     209.165.200.1:13   209.165.200.1:13
icmp 209.165.200.230:14 192.168.1.2:14     209.165.200.1:14   209.165.200.1:14
icmp 209.165.200.230:15 192.168.1.2:15     209.165.200.1:15   209.165.200.1:15
icmp 209.165.200.230:16 192.168.1.2:16     209.165.200.1:16   209.165.200.1:16
icmp 209.165.200.230:17 192.168.1.3:17     209.165.200.1:17   209.165.200.1:17
icmp 209.165.200.230:18 192.168.1.3:18     209.165.200.1:18   209.165.200.1:18
icmp 209.165.200.230:19 192.168.1.3:19     209.165.200.1:19   209.165.200.1:19
icmp 209.165.200.230:20 192.168.1.3:20     209.165.200.1:20   209.165.200.1:20
icmp 209.165.200.230:6 192.168.1.11:6     209.165.200.1:6    209.165.200.1:6
icmp 209.165.200.230:7 192.168.1.11:7     209.165.200.1:7    209.165.200.1:7
icmp 209.165.200.230:8 192.168.1.11:8     209.165.200.1:8    209.165.200.1:8
icmp 209.165.200.230:9 192.168.1.11:9     209.165.200.1:9    209.165.200.1:9
```

Все внутренние глобальные адреса используют один IP-адрес `209.165.200.230` с разными портами.

---

## Часть 4. Настройка и проверка статического NAT для IPv4

### 4.1 Очистка трансляций

```cisco
R1# clear ip nat translation *
```

### 4.2 Настройка статического NAT

Статическое сопоставление PC-A (`192.168.1.2`) с публичным адресом `209.165.200.229`:

```cisco
R1(config)# ip nat inside source static 192.168.1.2 209.165.200.229
```

### 4.3 Проверка таблицы NAT

```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
---  209.165.200.229   192.168.1.2        ---                ---
```

### 4.4 Проверка доступа из внешней сети


```cisco
R2#ping 209.165.200.229

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.229, timeout is 2 seconds:
!!!!!
Succes
```

### 4.5 Проверка трансляций при входящем трафике

```cisco
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.229:10192.168.1.2:10     209.165.200.225:10 209.165.200.225:10
icmp 209.165.200.229:2 192.168.1.2:2      209.165.200.225:2  209.165.200.225:2
icmp 209.165.200.229:3 192.168.1.2:3      209.165.200.225:3  209.165.200.225:3
icmp 209.165.200.229:4 192.168.1.2:4      209.165.200.225:4  209.165.200.225:4
icmp 209.165.200.229:5 192.168.1.2:5      209.165.200.225:5  209.165.200.225:5
icmp 209.165.200.229:6 192.168.1.2:6      209.165.200.225:6  209.165.200.225:6
icmp 209.165.200.229:7 192.168.1.2:7      209.165.200.225:7  209.165.200.225:7
icmp 209.165.200.229:8 192.168.1.2:8      209.165.200.225:8  209.165.200.225:8
icmp 209.165.200.229:9 192.168.1.2:9      209.165.200.225:9  209.165.200.225:9
---  209.165.200.229   192.168.1.2        ---                ---
```

Статический NAT работает корректно.
