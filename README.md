# Настройка NAT для IPv4

## Топология



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
R1(config-if)# exit
R1(config)# interface g0/0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
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
R2(config-if)# exit
R2(config)# interface loopback 1
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
R1# ping 209.165.200.225
!!!!!
Success rate is 100 percent (5/5)

R1# ping 209.165.200.1
!!!!!
Success rate is 100 percent (5/5)
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

#### PC-B → ping 209.165.200.1

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.226    192.168.1.3        ---                ---
icmp 209.165.200.226:1 192.168.1.3:1     209.165.200.1:1    209.165.200.1:1
Total number of translations: 2
```

**Вопрос:** Во что был транслирован внутренний локальный адрес PC-B?  
**Ответ:** `192.168.1.3` был транслирован в `209.165.200.226`.

**Вопрос:** Какой тип адреса NAT является переведённым адресом?  
**Ответ:** Внутренний глобальный (Inside global).

#### PC-A → ping 209.165.200.1

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.227    192.168.1.2        ---                ---
icmp 209.165.200.227:1 192.168.1.2:1     209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:1 192.168.1.3:1     209.165.200.1:1    209.165.200.1:1
Total number of translations: 4
```

#### S1 → ping 209.165.200.1

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.226    192.168.1.3        ---                ---
--- 209.165.200.227    192.168.1.2        ---                ---
--- 209.165.200.228    192.168.1.11       ---                ---
icmp 209.165.200.226:1 192.168.1.3:1     209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.228:0 192.168.1.11:0    209.165.200.1:0    209.165.200.1:0
Total number of translations: 5
```

#### S2 → ping 209.165.200.1 (ожидаемый сбой)

```
%NAT-6-ADDR_ALLOC_FAILURE: Address allocation failed; pool 1 may be exhausted
```

**Причина:** в пуле всего 3 адреса, а попытка четвёртого устройства — превышение лимита. NAT работает по принципу «один-к-одному».

#### Просмотр времени жизни трансляции

```cisco
R1# show ip nat translations verbose
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.226    192.168.1.3        ---                ---
    create: 09/23/19 15:35:27, use: 09/23/19 15:35:27, timeout: 23:56:42
    Map-Id(In): 1
Total number of translations: 5
```

**Ответ:** время жизни — 24 часа (23:56:42).

#### Очистка перед PAT

```cisco
R1# clear ip nat translations *
R1# clear ip nat statistics
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

#### Проверка: PC-B → ping 209.165.200.1

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.3:1     209.165.200.1:1    209.165.200.1:1
Total number of translations: 1
```

**Вопрос:** Во что был транслирован внутренний локальный адрес PC-B?  
**Ответ:** `192.168.1.3:1` → `209.165.200.226:1`.

**Вопрос:** Какой тип адреса NAT является переведённым адресом?  
**Ответ:** Внутренний глобальный с портом (Inside global + port).

**Вопрос:** Чем отличаются выходные данные от упражнения NAT?  
**Ответ:** В PAT используется один и тот же IP-адрес с разными портами для разных сессий. Время трансляции — 1 минута вместо 24 часов.

#### Проверка: PC-A → ping 209.165.200.1

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.2:1     209.165.200.1:1    209.165.200.1:1
Total number of translations: 1
```

> **Наблюдение:** используется тот же глобальный адрес `209.165.200.226`, но с другим портом при следующей сессии.

#### Одновременный трафик (PC-A и PC-B: `ping -t 209.165.200.1`)

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.2:1     209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:2 192.168.1.3:1     209.165.200.1:1    209.165.200.1:2
Total number of translations: 2
```

**Вопрос:** Как маршрутизатор отслеживает, куда идут ответы?  
**Ответ:** По номерам портов (например, 226:1 для PC-A и 226:2 для PC-B).

#### Остановка пингов и очистка
```
(на PC-A и PC-B нажать Ctrl+C)
```
```cisco
R1# clear ip nat translations *
R1# clear ip nat statistics
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

#### Проверка: PC-B → ping 209.165.200.1

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.230:1 192.168.1.3:1     209.165.200.1:1    209.165.200.1:1
Total number of translations: 1
```

#### Множественный трафик (PC-A, PC-B, S1, S2)

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 209.165.200.230:3 192.168.1.11:1    209.165.200.1:1    209.165.200.1:3
icmp 209.165.200.230:2 192.168.1.2:1     209.165.200.1:1    209.165.200.1:2
icmp 209.165.200.230:4 192.168.1.3:1     209.165.200.1:1    209.165.200.1:4
icmp 209.165.200.230:1 192.168.1.12:1    209.165.200.1:1    209.165.200.1:1
Total number of translations: 4
```

**Наблюдение:** все внутренние глобальные адреса используют один IP-адрес `209.165.200.230` с разными портами.

#### Остановка пингов
```
(на PC-A и PC-B нажать Ctrl+C)
```

---

## Часть 4. Настройка и проверка статического NAT для IPv4

### 4.1 Очистка трансляций

```cisco
R1# clear ip nat translations *
R1# clear ip nat statistics
```

### 4.2 Настройка статического NAT

Статическое сопоставление PC-A (`192.168.1.2`) с публичным адресом `209.165.200.229`:

```cisco
R1(config)# ip nat inside source static 192.168.1.2 209.165.200.229
```

### 4.3 Проверка таблицы NAT

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.229    192.168.1.2        ---                ---
Total number of translations: 1
```

### 4.4 Проверка доступа из внешней сети

С R2 выполнить ping на `209.165.200.229`:

```cisco
R2# ping 209.165.200.229
!!!!!
Success rate is 100 percent (5/5)
```

### 4.5 Проверка трансляций при входящем трафике

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 209.165.200.229    192.168.1.2        ---                ---
icmp 209.165.200.229:3 192.168.1.2:3     209.165.200.225:3  209.165.200.225:3
Total number of translations: 2
```

**Вывод:** статический NAT работает корректно.

---

## Сводная таблица по интерфейсам маршрутизаторов

| Модель | Интерфейс Ethernet №1 | Интерфейс Ethernet №2 | Последовательный №1 | Последовательный №2 |
|--------|----------------------|----------------------|---------------------|---------------------|
| 1800 | Fast Ethernet 0/0 (F0/0) | Fast Ethernet 0/1 (F0/1) | Serial 0/0/0 (S0/0/0) | Serial 0/0/1 (S0/0/1) |
| 1900 | Gigabit Ethernet 0/0 (G0/0) | Gigabit Ethernet 0/1 (G0/1) | Serial 0/0/0 (S0/0/0) | Serial 0/0/1 (S0/0/1) |
| 2801 | Fast Ethernet 0/0 (F0/0) | Fast Ethernet 0/1 (F0/1) | Serial 0/1/0 (S0/1/0) | Serial 0/1/1 (S0/1/1) |
| 2811 | Fast Ethernet 0/0 (F0/0) | Fast Ethernet 0/1 (F0/1) | Serial 0/0/0 (S0/0/0) | Serial 0/0/1 (S0/0/1) |
| 2900 | Gigabit Ethernet 0/0 (G0/0) | Gigabit Ethernet 0/1 (G0/1) | Serial 0/0/0 (S0/0/0) | Serial 0/0/1 (S0/0/1) |
| 4221 | Gigabit Ethernet 0/0/0 (G0/0/0) | Gigabit Ethernet 0/0/1 (G0/0/1) | Serial 0/1/0 (S0/1/0) | Serial 0/1/1 (S0/1/1) |
| 4300 | Gigabit Ethernet 0/0/0 (G0/0/0) | Gigabit Ethernet 0/0/1 (G0/0/1) | Serial 0/1/0 (S0/1/0) | Serial 0/1/1 (S0/1/1) |

---

**Конец лабораторной работы**
