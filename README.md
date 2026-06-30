# GNS3: Oddiy VLAN Segmentatsiyasi va 802.1Q Trunking

Ushbu repozitoriyada **GNS3** dasturida **VLAN (Virtual Local Area Network)** va **IEEE 802.1Q Trunking** texnologiyalaridan foydalangan holda tarmoqni segmentatsiyalash (bo‘lish) loyihasi keltirilgan.

Loyihada **NM-16ESW EtherSwitch** moduli o‘rnatilgan **Cisco 3745** routerlaridan switch sifatida foydalanilgan.

---

# 🌐 Tarmoq topologiyasi va IP manzillar

![alt text](<../screen/Screenshot from 2026-06-29 14-12-26.png>)\
Loyiha quyidagi qurilmalardan iborat:

- **SW1** — Birinchi switch
- **SW2** — Ikkinchi switch
- **PC1**
- **PC2**
- **PC3**
- **PC4**

### VLAN taqsimoti

- **VLAN 10 (HR bo'limi)** → PC1, PC3
- **VLAN 20 (Moliya bo'limi)** → PC2, PC4

| Qurilma | VLAN | Bo'lim | IP manzil | Subnet Mask | Port |
|:--------|:----:|:-------|:-----------|:------------|:-----|
| **PC1** | 10 | HR | 10.10.10.1 | 255.255.255.0 | SW1 Fa1/0 |
| **PC2** | 20 | Moliya | 10.10.20.2 | 255.255.255.0 | SW1 Fa1/1 |
| **PC3** | 10 | HR | 10.10.10.3 | 255.255.255.0 | SW2 Fa1/0 |
| **PC4** | 20 | Moliya | 10.10.20.4 | 255.255.255.0 | SW2 Fa1/1 |

> **Eslatma**
>
> SW1 va SW2 o‘rtasidagi **FastEthernet1/15** portlari **802.1Q Trunk** rejimida ishlaydi va barcha VLAN trafiklarini uzatadi.

---

# ⚙️ SW1 konfiguratsiyasi

```text
SW1# vlan database
! VLAN ma'lumotlar bazasi rejimiga kirish (Cisco 3745 EtherSwitch uchun)

SW1(vlan)# vlan 10
! VLAN 10 yaratish

SW1(vlan)# vlan 20
! VLAN 20 yaratish

SW1(vlan)# exit
! O'zgarishlarni saqlab chiqish

SW1# configure terminal
! Global konfiguratsiya rejimiga kirish

SW1(config)# interface fastEthernet 1/0
! PC1 ulangan port

SW1(config-if)# switchport mode access
! Access port

SW1(config-if)# switchport access vlan 10
! VLAN 10 ga biriktirish

SW1(config-if)# exit

SW1(config)# interface fastEthernet 1/1
! PC2 ulangan port

SW1(config-if)# switchport mode access
! Access port

SW1(config-if)# switchport access vlan 20
! VLAN 20 ga biriktirish

SW1(config-if)# exit

SW1(config)# interface fastEthernet 1/15
! SW2 bilan trunk ulanish

SW1(config-if)# switchport mode trunk
! Trunk rejimiga o'tkazish

SW1(config-if)# end
! Konfiguratsiyani yakunlash

SW1# write memory
! Sozlamalarni saqlash
```

---

# ⚙️ SW2 konfiguratsiyasi

```text
SW2# vlan database
! VLAN ma'lumotlar bazasi rejimiga kirish

SW2(vlan)# vlan 10
! VLAN 10 yaratish

SW2(vlan)# vlan 20
! VLAN 20 yaratish

SW2(vlan)# exit
! Saqlash va chiqish

SW2# configure terminal
! Global konfiguratsiya rejimiga kirish

SW2(config)# interface fastEthernet 1/0
! PC3 ulangan port

SW2(config-if)# switchport mode access
! Access port

SW2(config-if)# switchport access vlan 10
! VLAN 10 ga biriktirish

SW2(config-if)# exit

SW2(config)# interface fastEthernet 1/1
! PC4 ulangan port

SW2(config-if)# switchport mode access
! Access port

SW2(config-if)# switchport access vlan 20
! VLAN 20 ga biriktirish

SW2(config-if)# exit

SW2(config)# interface fastEthernet 1/15
! SW1 bilan trunk ulanish

SW2(config-if)# switchport mode trunk
! Trunk rejimiga o'tkazish

SW2(config-if)# end
! Konfiguratsiyani yakunlash

SW2# write memory
! Sozlamalarni saqlash
```

---

# 🖥️ VPCS kompyuterlarini sozlash

GNS3 dagi har bir virtual kompyuter terminalida quyidagi buyruqlar bajariladi.

### PC1

```text
ip 10.10.10.1 255.255.255.0
```

### PC2

```text
ip 10.10.20.2 255.255.255.0
```

### PC3

```text
ip 10.10.10.3 255.255.255.0
```

### PC4

```text
ip 10.10.20.4 255.255.255.0
```

---

# 🧪 Tarmoqni tekshirish

## 1. Bir xil VLAN ichidagi aloqa (Intra-VLAN)

PC1 dan PC3 ga ping.

```bash
PC1> ping 10.10.10.3

84 bytes from 10.10.10.3 icmp_seq=1 ttl=64 time=1.352 ms
84 bytes from 10.10.10.3 icmp_seq=2 ttl=64 time=1.011 ms
```
**Natija**

✅ Ping muvaffaqiyatli.

Bu trunk port orqali **VLAN 10** trafigi to'g'ri uzatilayotganini ko'rsatadi.

---

## 2. Turli VLAN lar o'rtasidagi aloqa (Inter-VLAN)

PC1 dan PC2 ga ping.

```bash
PC1> ping 10.10.20.2

*10.10.10.1 icmp_seq=1 timeout
```

**Natija**

❌ Aloqa mavjud emas.

Bu holat normal hisoblanadi, chunki **Inter-VLAN Routing** sozlanmagan.

---
![alt text](<../screen/Screenshot from 2026-06-29 14-13-11.png>)
## 3. VLAN holatini tekshirish

Cisco EtherSwitch modulida VLANlarni tekshirish.

```text
SW1# show vlan-switch brief
```
![alt text](<../screen/Screenshot from 2026-06-29 14-14-13.png>)\
Bu buyruq:

- yaratilgan VLANlarni;
- VLAN nomlarini;
- har bir VLANga biriktirilgan portlarni ko'rsatadi.


# GNS3: EtherChannel (Port-Channel) texnologiyasi

Ushbu loyihada **EtherChannel (Port-Channel)** texnologiyasi yordamida ikkita jismoniy Ethernet ulanishi bitta mantiqiy kanalga birlashtiriladi.

EtherChannel quyidagi afzalliklarni beradi:

- **Bandwidth oshadi** — ikkita kabel bitta kanal sifatida ishlaydi.
- **Redundancy (zaxira aloqa)** — kabellardan biri uzilsa ham tarmoq ishlashda davom etadi.
- **STP bloklanishini kamaytiradi** — Spanning Tree ikkita kabelni alohida emas, bitta mantiqiy ulanish sifatida ko'radi.

> **Eslatma**
>
> Ushbu laboratoriya **Cisco 3745 + NM-16ESW EtherSwitch** moduli asosida bajariladi. EtherChannel **Static Mode (mode on)** yordamida sozlanadi.

---

# 🌐 Topologiya

Oldingi VLAN laboratoriyasidagi topologiya saqlanadi.

SW1 va SW2 orasida ikkita parallel magistral ulanish mavjud:

- **FastEthernet1/14**
- **FastEthernet1/15**

Ushbu ikkala port **Port-Channel1 (Po1)** tarkibiga birlashtiriladi.

```text
          +----------------------+
          |        SW1           |
          |                      |
 Fa1/14 ================= Fa1/14
 Fa1/15 ================= Fa1/15
          |                      |
          |        SW2           |
          +----------------------+

         EtherChannel (Port-Channel1)
```

---


# ⚙️ SW1 konfiguratsiyasi

```text
SW1# configure terminal
! Global konfiguratsiya rejimiga kirish

SW1(config)# interface range fastEthernet 1/14 - 15
! Fa1/14 va Fa1/15 portlarini birgalikda tanlash

SW1(config-if-range)# channel-group 1 mode on
! Ikkala portni Port-Channel1 tarkibiga qo'shish
! "mode on" - Static EtherChannel

SW1(config-if-range)# exit

SW1(config)# interface port-channel 1
! Port-Channel1 mantiqiy interfeysiga kirish

SW1(config-if)# switchport mode trunk
! Port-Channelni Trunk rejimiga o'tkazish

SW1(config-if)# end

SW1# write memory
! Sozlamalarni saqlash
```

---

# ⚙️ SW2 konfiguratsiyasi

```text
SW2# configure terminal
! Global konfiguratsiya rejimiga kirish

SW2(config)# interface range fastEthernet 1/14 - 15
! Fa1/14 va Fa1/15 portlarini tanlash

SW2(config-if-range)# channel-group 1 mode on
! Portlarni Port-Channel1 tarkibiga qo'shish

SW2(config-if-range)# exit

SW2(config)# interface port-channel 1
! Port-Channel interfeysiga kirish

SW2(config-if)# switchport mode trunk
! Trunk rejimini yoqish

SW2(config-if)# end

SW2# write memory
! Sozlamalarni saqlash
```

---

# 🔍 EtherChannel holatini tekshirish

Port-Channel muvaffaqiyatli yaratilganini tekshirish uchun:

```text
SW1# show etherchannel summary
```

Kutilayotgan natija:

```text
Group  Port-channel  Protocol    Ports
------+-------------+---------+-------------------------

1      Po1(SU)       -         Fa1/14(P) Fa1/15(P)
```

### Natijadagi belgilar

| Belgi | Ma'nosi |
|--------|---------|
| **Po1** | Port-Channel 1 |
| **S** | Layer-2 (Switching) Channel |
| **U** | Channel ishlayapti (In Use) |
| **P** | Port EtherChannel tarkibida faol ishlayapti |

---

# 🧪 Aloqani tekshirish

## 1. Oddiy holat

PC1 dan PC3 ga ping yuboriladi.

```bash
PC1> ping 10.10.10.3
```

Kutilayotgan natija:

```text
84 bytes from 10.10.10.3
84 bytes from 10.10.10.3
84 bytes from 10.10.10.3
```

**Natija**

✅ Aloqa muvaffaqiyatli.

---

## 2. Redundancy testi

GNS3 da quyidagi ulanishlardan birini uzing:

- **Fa1/15**

yoki

- **Fa1/14**

So'ng yana ping yuboring.

```bash
PC1> ping 10.10.10.3
```

Kutilayotgan natija:

```text
84 bytes from 10.10.10.3
84 bytes from 10.10.10.3
84 bytes from 10.10.10.3
```

**Natija**

✅ Ping uzilmaydi.

EtherChannel avtomatik ravishda qolgan faol kabel orqali trafikni uzatishni davom ettiradi.

---

# 📋 EtherChannel holatini tekshirish buyruqlari

## EtherChannel umumiy holati

```text
show etherchannel summary
```

---

## Port-Channel interfeysini tekshirish

```text
show interfaces port-channel 1
```

---

## Trunk holatini tekshirish

```text
show interfaces trunk
```

---

## VLAN holatini tekshirish

```text
show vlan-switch brief
```

---

# ✅ Xulosa

Ushbu laboratoriyada:

- ikkita fizik Ethernet ulanishi bitta **Port-Channel** ga birlashtirildi;
- **Static EtherChannel (mode on)** konfiguratsiyasi bajarildi;
- **802.1Q Trunk** Port-Channel interfeysida ishlatildi;
- **Bandwidth oshirildi** va **Redundancy** ta'minlandi;
- bitta kabel uzilganda ham aloqa uzilmasligi amalda tekshirildi.


# GNS3: OSPF Dynamic Routing (Single Area)

Ushbu loyiha GNS3 muhitida **OSPF (Open Shortest Path First)** dinamik marshrutlash protokolini **Single Area (Area 0)** topologiyasida sozlash va sinovdan o'tkazishni ko'rsatadi.

---

# 🌐 Tarmoq topologiyasi va IP reja

Topologiyada 3 ta router chiziqli (**Linear Topology**) ko'rinishida ulanadi.

```text
R1  <-------->  R2  <-------->  R3
```

Loyihada quyidagi 4 ta tarmoq ishlatiladi:

| Tarmoq | Tarmoq manzili | Maqsadi |
| :--- | :--- | :--- |
| 1 | 192.168.1.0/24 | R1 Loopback tarmog'i |
| 2 | 10.1.12.0/24 | R1 ↔ R2 ulanishi |
| 3 | 10.1.23.0/24 | R2 ↔ R3 ulanishi |
| 4 | 192.168.3.0/24 | R3 Loopback tarmog'i |

---

# 📊 IP manzillar jadvali

| Qurilma | Interfeys | IP manzili | Subnet Mask | OSPF Area |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Loopback0 | 192.168.1.1 | 255.255.255.0 | Area 0 |
| **R1** | FastEthernet0/0 | 10.1.12.1 | 255.255.255.0 | Area 0 |
| **R2** | FastEthernet0/0 | 10.1.12.2 | 255.255.255.0 | Area 0 |
| **R2** | FastEthernet0/1 | 10.1.23.2 | 255.255.255.0 | Area 0 |
| **R3** | FastEthernet0/1 | 10.1.23.3 | 255.255.255.0 | Area 0 |
| **R3** | Loopback0 | 192.168.3.1 | 255.255.255.0 | Area 0 |

> **Muhim:** Cisco IOS OSPF konfiguratsiyasida **Subnet Mask** emas, **Wildcard Mask** ishlatiladi.  
> Masalan:
>
> - `255.255.255.0`
> - Wildcard → `0.0.0.255`

---

# ⚙️ Routerlarni sozlash

## 🔹 R1 konfiguratsiyasi

```text
R1# configure terminal
! Global konfiguratsiya rejimiga kirish

R1(config)# interface loopback 0
! Virtual Loopback interfeys yaratish

R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# exit

R1(config)# interface fastEthernet 0/0
! R2 bilan bog'langan interfeys

R1(config-if)# ip address 10.1.12.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

! OSPF ni ishga tushirish
R1(config)# router ospf 1

R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
! Loopback tarmog'ini e'lon qilish

R1(config-router)# network 10.1.12.0 0.0.0.255 area 0
! R2 bilan bog'langan tarmoqni e'lon qilish

R1(config-router)# end

R1# write memory
```

---

## 🔹 R2 konfiguratsiyasi

```text
R2# configure terminal

R2(config)# interface fastEthernet 0/0
! R1 bilan bog'langan interfeys

R2(config-if)# ip address 10.1.12.2 255.255.255.0
R2(config-if)# no shutdown

R2(config)# interface fastEthernet 0/1
! R3 bilan bog'langan interfeys

R2(config-if)# ip address 10.1.23.2 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit

! OSPF ni ishga tushirish
R2(config)# router ospf 1

R2(config-router)# network 10.1.12.0 0.0.0.255 area 0

R2(config-router)# network 10.1.23.0 0.0.0.255 area 0
! Ikkala tarmoqni ham OSPF ga qo'shish

R2(config-router)# end

R2# write memory
```

---

## 🔹 R3 konfiguratsiyasi

```text
R3# configure terminal

R3(config)# interface fastEthernet 0/1
! R2 bilan bog'langan interfeys

R3(config-if)# ip address 10.1.23.3 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface loopback 0
! Virtual Loopback interfeys

R3(config-if)# ip address 192.168.3.1 255.255.255.0
R3(config-if)# exit

! OSPF ni ishga tushirish
R3(config)# router ospf 1

R3(config-router)# network 10.1.23.0 0.0.0.255 area 0

R3(config-router)# network 192.168.3.0 0.0.0.255 area 0

R3(config-router)# end

R3# write memory
```

---

# 🧪 Tekshirish (Verification)

Routerlar qo'shnichilik (Adjacency) hosil qilganda quyidagi xabar paydo bo'ladi:

```text
%OSPF-5-ADJCHG:
Process 1,
Nbr ... on FastEthernet...
from LOADING to FULL
```

Bu OSPF qo'shnichilik muvaffaqiyatli o'rnatilganini bildiradi.

---

## 1. OSPF qo'shnilarini tekshirish

R2 routerida:

```text
R2# show ip ospf neighbor
```

Agar jadvalda:

- R1
- R3

routerlari ko'rinsa va holati **FULL** bo'lsa, OSPF to'g'ri ishlayapti.

---

## 2. Routing jadvalini tekshirish

R1 routerida:

```text
R1# show ip route
```

Natijada quyidagiga o'xshash yozuv chiqadi:

```text
O 192.168.3.0/24 [110/11]
via 10.1.12.2,
FastEthernet0/0
```

Bu R1 routeri R3 ning tarmog'ini OSPF orqali avtomatik o'rganganini bildiradi.

---

# 🏁 Yakuniy sinov (Ping Test)

## R1 → R3

```text
R1# ping 192.168.3.1 source loopback 0
```

Natija:

```text
Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 192.168.3.1,
timeout is 2 seconds:

Packet sent with a source address of 192.168.1.1

!!!!!

Success rate is 100 percent (5/5),
round-trip min/avg/max = 4/12/28 ms
```

---

## R3 → R1

```text
R3# ping 192.168.1.1 source loopback 0
```

Natija:

```text
Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 192.168.1.1,
timeout is 2 seconds:

Packet sent with a source address of 192.168.3.1

!!!!!

Success rate is 100 percent (5/5)
```

---

# ✅ Xulosa

- OSPF Process ID **1** bilan ishga tushirildi.
- Barcha interfeyslar **Area 0** ga qo'shildi.
- Routerlar o'zaro **FULL Adjacency** holatiga o'tdi.
- Marshrutlar dinamik ravishda **OSPF** orqali almashildi.
- `show ip ospf neighbor` buyrug'i qo'shnichilikni tasdiqladi.
- `show ip route` buyrug'i OSPF orqali o'rganilgan marshrutlarni (`O`) ko'rsatdi.
- Loopback interfeyslari orasidagi ping muvaffaqiyatli bajarildi, bu dinamik marshrutlash to'g'ri ishlayotganini isbotlaydi.

# GNS3: HSRP (Hot Standby Router Protocol)

Ushbu loyiha GNS3 muhitida **HSRP (Hot Standby Router Protocol)** protokolini sozlash va sinovdan o'tkazishni ko'rsatadi.

HSRP — Cisco kompaniyasiga tegishli **First Hop Redundancy Protocol (FHRP)** bo'lib, bitta router ishdan chiqsa, ikkinchi router foydalanuvchiga sezilmagan holda **Default Gateway** vazifasini avtomatik ravishda davom ettiradi.

---

# 🌐 Tarmoq topologiyasi va IP reja

Topologiyada bitta oddiy **Ethernet Switch**, ikkita router va bitta virtual kompyuter (VPCS) ishlatiladi.
![alt text](<../screen/Screenshot from 2026-06-30 10-41-07.png>)

Barcha qurilmalar **192.168.1.0/24** tarmog'ida joylashgan.

---

# 📊 IP manzillar jadvali

| Qurilma | Interfeys | Jismoniy IP | Virtual IP (HSRP Gateway) |
| :--- | :--- | :--- | :--- |
| **R1 (Active)** | FastEthernet0/0 | 192.168.1.2 | 192.168.1.1 |
| **R2 (Standby)** | FastEthernet0/0 | 192.168.1.3 | 192.168.1.1 |
| **PC1 (VPCS)** | eth0 | 192.168.1.10 | Default Gateway → 192.168.1.1 |

> **Muhim:** Kompyuter Default Gateway sifatida routerlarning jismoniy IP manzilini emas, balki **Virtual IP (192.168.1.1)** manzilini ishlatadi.

---

# ⚙️ Routerlarni sozlash

## 🔹 R1 konfiguratsiyasi (Active Router)

R1 routeriga yuqori **Priority (110)** beriladi, shuning uchun u **Active** holatda ishlaydi.

```text
R1# configure terminal

R1(config)# interface fastEthernet 0/0

R1(config-if)# ip address 192.168.1.2 255.255.255.0

R1(config-if)# no shutdown

! HSRP Virtual Gateway
R1(config-if)# standby 1 ip 192.168.1.1

! Priority oshirish
R1(config-if)# standby 1 priority 110

! Router qayta ishga tushganda Active holatini qaytarib oladi
R1(config-if)# standby 1 preempt

R1(config-if)# end

R1# write memory
```

---

## 🔹 R2 konfiguratsiyasi (Standby Router)

R2 da **Priority** ko'rsatilmaydi. Cisco standart qiymati **100** bo'lgani sababli u avtomatik ravishda **Standby** holatida ishlaydi.

```text
R2# configure terminal

R2(config)# interface fastEthernet 0/0

R2(config-if)# ip address 192.168.1.3 255.255.255.0

R2(config-if)# no shutdown

! HSRP Virtual Gateway
R2(config-if)# standby 1 ip 192.168.1.1

! Active router qaytganda unga ustunlikni qaytarish
R2(config-if)# standby 1 preempt

R2(config-if)# end

R2# write memory
```

---

# 🖥️ PC1 (VPCS) konfiguratsiyasi

Virtual kompyuterda IP manzil va Default Gateway ni sozlang.

```text
PC1> ip 192.168.1.10 255.255.255.0 192.168.1.1
```

Bu yerda:

- IP Address → `192.168.1.10`
- Subnet Mask → `255.255.255.0`
- Default Gateway → **192.168.1.1 (Virtual IP)**

---

# 🧪 Tekshirish (Verification)

Routerlar HSRP holatini aniqlashi uchun taxminan **10–15 soniya** kuting.

R1 konsolida quyidagiga o'xshash xabar paydo bo'ladi:

```text
%HSRP-5-STATECHANGE:
FastEthernet0/0 Grp 1
state Speak -> Active
```

Bu R1 Active holatga o'tganini bildiradi.

---

## 1. HSRP holatini tekshirish

### R1

```text
R1# show standby brief
```

Natija:

```text
                     P indicates configured to preempt.

Interface   Grp Pri P State    Active  Standby      Virtual IP

Fa0/0       1   110 P Active   local   192.168.1.3 192.168.1.1
```

Bu yerda:

- **State** → Active
- **Priority** → 110
- **Virtual IP** → 192.168.1.1

---

### R2

```text
R2# show standby brief
```

Natija quyidagicha:

```text
                     P indicates configured to preempt.

Interface   Grp Pri P State     Active        Standby  Virtual IP

Fa0/0       1   100 P Standby   192.168.1.2   local    192.168.1.1
```

Bu yerda:

- **State** → Standby
- **Active Router** → 192.168.1.2 (R1)

---

# 🏁 Failover testi

## 1-qadam

PC1 dan Virtual Gateway manziliga uzluksiz ping yubordim.

```text
PC1> ping 192.168.1.1 -t
```

Ping muvaffaqiyatli davom etadi.

---

## 2-qadam

R1 routerida interfeysni o'chirdim.

```text
R1# configure terminal

R1(config)# interface fastEthernet 0/0

R1(config-if)# shutdown
```

---

## 3-qadam

PC1 dagi ping natijasini kuzatdim.

```text
Reply from 192.168.1.1 ...

Reply from 192.168.1.1 ...

Request timed out.

Request timed out.

Reply from 192.168.1.1 ...

Reply from 192.168.1.1 ...
```

Odatda faqat **1–2 ta paket** yo'qoladi.

Keyin ping avtomatik ravishda davom etadi.

---

## 4-qadam

R2 da holatni tekshiring.

```text
R2# show standby brief
```

Natija:

```text
Interface   Grp Pri P State   Active  Standby  Virtual IP

Fa0/0       1   100 P Active  local   unknown  192.168.1.1
```

Bu R2 avtomatik ravishda **Active Router** bo'lganini bildiradi.

---

# ✅ Xulosa

- HSRP yordamida ikkita router bitta Virtual Gateway hosil qildi.
- Virtual Gateway sifatida **192.168.1.1** ishlatildi.
- R1 yuqori **Priority (110)** sababli **Active Router** bo'ldi.
- R2 standart **Priority (100)** bilan **Standby Router** sifatida ishladi.
- `preempt` buyrug'i Active router qaytganda uning ustunligini tiklaydi.
- `show standby brief` yordamida HSRP holati tekshirildi.
- Active router ishdan chiqqanda Standby router avtomatik ravishda uning vazifasini egalladi.
- Failover vaqtida faqat 1–2 ta ping yo'qoldi va tarmoq ishlashda davom etdi.