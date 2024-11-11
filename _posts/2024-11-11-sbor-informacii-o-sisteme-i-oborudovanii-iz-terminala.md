---
layout: post
title: Сбор информации о системе и оборудовании из терминала
description: Рассматриваются различные методы сбора информации о опреционной системе
  и оборудовании из терминала в среде Linux
image:
categories:
- Linux
- TLDR
tags:
- linux
- pocketbook
- hardware
date: 2024-11-11 17:09 +0300
---
### Информация о дистрибутиве

**1. Debian**

```bash
cat /etc/os-release
```

**Примерный вывод**

```
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

**2. Ubuntu**

```bash
cat /etc/os-release
```

**Примерный вывод**

```bash
PRETTY_NAME="Ubuntu 22.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.4 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

```bash
cat /etc/lsb-release

```

**Примерный вывод**

```bash
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.4 LTS"
```

---

**3. С использованием `lsb_release`** _(В Debian по умолчанию не установлена)_

```bash
lsb_release -a
```

**Примерный вывод**

```bash
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
Release:        22.04
Codename:       jammy
```

**Ключи**

```bash
  -i, --id показать идентификатор дистрибутива
  -d, ---description показать описание этого дистрибутива
  -r, --release показать номер релиза этого дистрибутива
  -c, ---codename показать кодовое имя этого дистрибутива
  -a, --all показать всю вышеперечисленную информацию
  -s, --short показать запрашиваемую информацию в коротком формате
```

**Пример**

```bash
lsb_release -r
Release:        22.04

lsb_release -rs
22.04
```
---

### Информация о процессере (CPU)

```bash
lscpu
```

**Примерный вывод**

```bash
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         39 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               GenuineIntel
  Model name:            Intel(R) Core(TM) i5-7500 CPU @ 3.40GHz
    CPU family:          6
    Model:               158
    Thread(s) per core:  1
    Core(s) per socket:  4
    Socket(s):           1
    Stepping:            9
    BogoMIPS:            6816.00
    Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl
                          xtopology cpuid pni pclmulqdq ssse3 fma cx16 pdcm pcid sse4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single pti ssbd ib
                         rs ibpb stibp fsgsbase bmi1 hle avx2 smep bmi2 erms invpcid rtm rdseed adx smap clflushopt xsaveopt xsavec xgetbv1 xsaves md_clear flush_l1d arch_capabilities
Virtualization features: 
  Hypervisor vendor:     Microsoft
  Virtualization type:   full
Caches (sum of all):     
  L1d:                   128 KiB (4 instances)
  L1i:                   128 KiB (4 instances)
  L2:                    1 MiB (4 instances)
  L3:                    6 MiB (1 instance)
Vulnerabilities:         
  Gather data sampling:  Unknown: Dependent on hypervisor status
  Itlb multihit:         KVM: Mitigation: VMX unsupported
  L1tf:                  Mitigation; PTE Inversion
  Mds:                   Mitigation; Clear CPU buffers; SMT Host state unknown
  Meltdown:              Mitigation; PTI
  Mmio stale data:       Mitigation; Clear CPU buffers; SMT Host state unknown
  Retbleed:              Mitigation; IBRS
  Spec rstack overflow:  Not affected
  Spec store bypass:     Mitigation; Speculative Store Bypass disabled via prctl and seccomp
  Spectre v1:            Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:            Mitigation; IBRS, IBPB conditional, STIBP disabled, RSB filling, PBRSB-eIBRS Not affected
  Srbds:                 Unknown: Dependent on hypervisor status
  Tsx async abort:       Mitigation; Clear CPU buffers; SMT Host state unknown
```







### Сводная таблица

| Команда                                                  | Описание                                                        |
| -------------------------------------------------------- | --------------------------------------------------------------- |
| `cat /etc/*-release`                                     | Информация о дистрибутиве                                       |
| `lsb_release -cs`                                        | Кодовое имя дистрибутива (в Debian по умолчанию не установлено) |
| `lscpu` и `sudo lshw -class processor`                   | Информация о процессоре                                         |
| `cat /proc/cpuinfo  | grep name  | uniq`                 | Модель процессора                                               |
| `arch` или `uname -m`                                    | Архитектура процессора                                          |
| `nproc` или `cat /proc/cpuinfo  | grep process  | wc -l` | Количество ядер процессора                                      |
| `free -m`                                                | Доступная и используемая оперативная память                     |
| `cat /proc/meminfo`                                      | Подробная информация по оперативной и виртуальной памяти        |
| `sudo dmidecode --type 17`                               | Тип и размер оперативной памяти                                 |
| `sudo fdisk -l | grep "Disk /dev/sd"`                    | Список дисков                                                   |
| `cat /proc/uptime`                                       | Сек., время работы CPU и время простоя в сумме для каждого ядра |
| `sudo lshw -C display`                                   | Информация о видеоадаптерах (по умолчанию не установлено)       |
| `sudo dmidecode -t baseboard`                            | Модель материнской платы (по умолчанию не установлено)          |
| `sudo smartctl -A /dev/nvme0`                            | Проверить состояние SSD (по умолчанию не установлено)           |
| `df -h --total`                                          | Информация об используемом месте на различных носителях 

