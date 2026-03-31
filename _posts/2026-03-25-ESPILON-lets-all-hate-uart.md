---
title: ESPILON CTF (iot)Let's All Hate UART Write-up
description: ESPILON CTF write-up 작성
author: rlozll
date: 2026-03-25 09:00:00 +0900
categories: [write-up, ctf]
tags: [iot]
pin: false
math: true
mermaid: true
---
# 1. 문제

```
Chapitre 1 — Peripheral Access. 
Dans les sous-sols de la Clinique Sainte-Mika, vous avez obtenu un accès physique à un module de thérapie WIRED-MED. 
Vous avez identifié les pads UART sur le PCB. Connectez-vous et explorez ce device embarqué — il cache bien plus que ce qu'il montre. 
TX (lecture): nc CHALLENGE_HOST 1111 | RX (écriture): nc CHALLENGE_HOST 2222. 
Connectez les deux simultanément.
Challenge made by love
```

# 2. 풀이

- TX: ESP32 칩셋에서 뿜어내는 로그와 명령어 실행 결과만 출력되는 화면

- RX: 내가 장비로 명령어를 밀어 넣는 화면

우선 문제에서 준 주소로 nc 연결을 해보았다.

```bash
rlozll@rlozll-laptop:~$ nc espilon.net 45660
ets Jul 29 2019 12:21:46

rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:2
load:0x3fff0030,len:7176
load:0x40078000,len:15564
ho 0 tail 12 room 4
load:0x40080400,len:4
entry 0x40080698
I (31) boot: ESP-IDF v5.3.2 2nd stage bootloader
I (31) boot: compile time Jul 15 2024 08:30:00
I (31) boot: chip revision: v3.1
I (34) boot: Multiboot not supported, boot from SPI
I (40) boot.esp32: SPI Speed      : 40MHz
I (44) boot.esp32: SPI Mode       : DIO
I (49) boot.esp32: SPI Flash Size : 4MB
I (53) boot: Enabling RNG early entropy source...
I (59) boot: Partition Table:
I (62) boot: ## Label            Usage          Type ST Offset   Length
I (70) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (77) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (85) boot:  2 factory          factory app      00 00 00010000 00100000
I (92) boot: End of partition table
I (97) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1bc64h
I (149) esp_image: segment 1: paddr=0002bc8c vaddr=3ffb0000 size=03a54h
I (155) esp_image: segment 2: paddr=0002f6e8 vaddr=40080000 size=05a28h
I (165) esp_image: segment 3: paddr=00035118 vaddr=400c0000 size=00064h
I (173) boot: Loaded app from partition at offset 0x10000
I (173) boot: Disabling RNG early entropy source...
I (187) cpu_start: Unicore app
I (195) cpu_start: Pro cpu start user code
I (195) cpu_start: cpu freq: 160000000 Hz
I (195) cpu_start: Application information:
I (198) cpu_start:   Project name:     wired_med_therapy
I (204) cpu_start:   App version:      2.3.0
I (209) cpu_start:   Compile time:     Jul 15 2024 08:30:00
I (215) cpu_start:   ELF file SHA256:  a7c3f9e2b1d0...
I (221) cpu_start:   ESP-IDF:          v5.3.2
I (226) heap_init: Initializing. RAM available for dynamic allocation:
I (233) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (239) heap_init: At 3FFB8AD8 len 00027528 (157 KiB): DRAM
I (246) heap_init: At 3FFE0440 len 0001FBC0 (126 KiB): D/IRAM
I (253) spi_flash: detected chip: generic
I (256) spi_flash: flash io: dio
I (261) main_task: Started on CPU0
I (271) main_task: Calling app_main()
I (300) WIRED-MED: ========================================
I (300) WIRED-MED:   WIRED-MED Therapy Module v2.3
I (300) WIRED-MED:   Clinique Sainte-Mika
I (300) WIRED-MED: ========================================
I (320) WIRED-MED: Device serial: WM-2024-SAINTMIKA-0042
I (340) WIRED-MED: NVS initialized (6 entries loaded)
I (360) WIRED-MED: Crypto subsystem: AES-256-CBC [ACTIVE]
I (380) WIRED-MED: Debug interface: ENABLED (restricted)
I (400) wifi: wifi driver task: 3ffc1e04, prio:23, stack:6656
I (400) wifi: mode : sta (a4:cf:12:d8:3e:9c)
I (410) wifi: Set ps type: 0, coexist: 0
I (450) WIRED-MED: Connecting to SSID: WIRED-MED-AP...
I (850) wifi: new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>
I (850) wifi: state: init -> auth (b0)
I (860) wifi: state: auth -> assoc (0)
I (870) wifi: state: assoc -> run (10)
I (900) WIRED-MED: WiFi connected. IP: 192.168.1.42
I (920) WIRED-MED: MQTT connecting to wired-med.local:1883...
I (1050) WIRED-MED: MQTT client connected
I (1060) WIRED-MED: Publishing patient data to /clinic/therapy/0042
I (1100) WIRED-MED: System ready. UART console active.
I (1100) WIRED-MED: Type 'help' for available commands.

WIRED-MED> 
```

ESP32 기반 CTF 문제에서는 보통 플래그가 NVS(Non-Volatile Storage) 파티션에 저장되어 있는 경우가 많다. 부팅 로그에서도 NVS initialized (6 entries loaded)라는 문구가 보인다. 그래서 일단 전체 명령어를 확인해보겠다.

RX 터미널에 help를 입력했다.

![light_mode_only](./assets/img/2026-03-31/image3.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark_mode_only](./assets/img/2026-03-31/image3.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }

그래서 info, status, whoami를 순서대로 쳐본 결과, TX 터미널에 아래와 같이 출력됐다.

```bash
WIRED-MED> ╔══════════════════════════════════════╗
║     WIRED-MED Therapy Module v2.3    ║
╚══════════════════════════════════════╝
  Firmware:    wired_med_therapy v2.3.0
  ESP-IDF:     v5.3.2
  Chip:        ESP32-WROOM-32 rev3.1
  Flash:       4MB DIO @ 40MHz
  Serial:      WM-2024-SAINTMIKA-0042
  MAC:         a4:cf:12:d8:3e:9c
  Uptime:      0d 00h 12m 34s
  Free heap:   124832 bytes
  WiFi:        Connected (192.168.1.42)
  MQTT:        Connected (wired-med.local)
  Encryption:  AES-256-CBC [ACTIVE]
  Debug interface: ENABLED (restricted)
WIRED-MED> ┌─────────────────────────────────────┐
│      Patient Monitoring Status       │
├─────────────────────────────────────┤
│  SpO2:        98 %                  │
│  Heart Rate:  72 bpm                │
│  Blood Press: 120/80 mmHg           │
│  Temperature: 36.8 °C               │
│  Resp. Rate:  16 /min               │
│  IV Drip:     42 mL/h               │
├─────────────────────────────────────┤
│  Therapy:     ACTIVE (Protocol W-7) │
│  Last update: 2024-07-15 14:23:01   │
└─────────────────────────────────────┘
WIRED-MED> therapy-module-0042
```

info의 마지막 부분을 보면 Debug interface:ENABLED (restricted) 라고 나와있다.

플래그는 날아가면 안되기 때문에 ESP32 기반에서는 비휘발성 메모리인 NVS에 저장된다.

히든 명령어가 있을 수도 있기 때문에 다른 것들을 몇 개 더 시도해보겠다.

debug, admin, su, nvs, nvs_get, dump, ?을 각각 입력했을 때 아래와 같이 출력됐다.

```bash
WIRED-MED> Debug mode requires authentication.
Usage: debug auth <token>
       debug status
WIRED-MED> Unknown command: 'admin'. Type 'help' for available commands.
WIRED-MED> Unknown command: 'su'. Type 'help' for available commands.
WIRED-MED> Error: Restricted. Authenticate with 'debug auth <token>' first.
WIRED-MED> Unknown command: 'nvs_get'. Type 'help' for available commands.
WIRED-MED> Unknown command: 'dump'. Type 'help' for available commands.
WIRED-MED> Available commands:
  info       - Display device information
  status     - Show sensor readings
  whoami     - Display device identity
  reboot     - Restart the device
  help       - Show this help message

Debug commands (restricted):
  debug      - Debug mode authentication
  mem        - Memory read operations
  nvs        - NVS partition operations
  flash      - Flash memory operations
```

debug, nvs 등은 인증 토큰이 필요했고, mem을 입력했더니 아래의 결과를 볼 수 있었다.

```bash
WIRED-MED> ESP32 Memory Map:
  0x3FFB0000 - 0x3FFB1000  DRAM (public)    4 KiB
  0x3FFC0000 - 0x3FFC1000  DRAM (protected) 4 KiB
  0x40080000 - 0x400A0000  IRAM             128 KiB
  0x400C0000 - 0x400C0064  RTC FAST         100 B
```

```bash
mem read 0x3FFB0000 4
mem read  0x3FFC0000 4
mem read  0x40080000 128
mem read 0x400C0000 100
```

그래서 위의 명령어로 각각의 메모리 부분을 확인했다.

```bash
WIRED-MED> 3FFB0000: 57 49 52 45                                       |WIRE|
WIRED-MED> Error: Access denied. Protected memory region.
       Authenticate with 'debug auth <token>' first.
WIRED-MED> Error: No memory mapped at 0x40080000
WIRED-MED> Error: No memory mapped at 0x400C0000
```

오! 전의 로그와 함께 다시 보면, 전의 로그에서 `I (149) esp_image: segment 1: paddr=0002bc8c vaddr=3ffb0000 size=03a54h` 라는 부분이 있다. ESP32 펌웨어가 부팅될 때, 플래시 메모리에 있던 segment1 영역을 정확히 `0x3FFB0000` 메모리 주소에 올려놓은 것이다.

이 주소는 통상적으로 펌웨어에 하드코딩된 전역 변수나 평문 문자열이 저장되는 데이터 영역이다.

근데 딱 4개인 WIRE만 나온 이유는, 메모리 영역의 첫 4바이트만 입력해서 그런 것 같다. 그래서 각각 더 입력해보았다.

```bash
//input
mem read 0x3FFB0100 256
mem read 0x3FFB0200 256
mem read 0x3FFB0300 256
mem read 0x3FFB0400 256
```

```bash
//output
WIRED-MED> 3FFB0100: 4D 41 43 3A 20 61 34 3A  63 66 3A 31 32 3A 64 38   |MAC: a4:cf:12:d8|
3FFB0110: 3A 33 65 3A 39 63 00 00  00 00 00 00 00 00 00 00   |:3e:9c..........|
3FFB0120: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0130: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0140: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0150: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0160: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0170: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0180: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0190: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB01A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB01B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB01C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB01D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB01E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB01F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
WIRED-MED> 3FFB0200: 45 53 50 2D 49 44 46 20  76 35 2E 33 2E 32 20 28   |ESP-IDF v5.3.2 (|
3FFB0210: 62 75 69 6C 64 20 32 30  32 34 2D 30 31 2D 31 35   |build 2024-01-15|
3FFB0220: 54 30 38 3A 33 30 3A 30  30 5A 29 00 00 00 00 00   |T08:30:00Z).....|
3FFB0230: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0240: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0250: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0260: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0270: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0280: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0290: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB02A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB02B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB02C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB02D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB02E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB02F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
WIRED-MED> 3FFB0300: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0310: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0320: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0330: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0340: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0350: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0360: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0370: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0380: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0390: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB03A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB03B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB03C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB03D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB03E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB03F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
WIRED-MED> 3FFB0400: 58 6E 42 17 33 9A 1D FB  98 C8 D8 1C 4D 4B 6D 0E   |XnB.3.......MKm.|
3FFB0410: 3E A8 C6 41 E5 7E 69 22  E1 33 22 95 E8 B5 56 BC   |>..A.~i".3"...V.|
3FFB0420: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0430: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0440: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0450: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0460: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0470: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0480: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0490: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB04A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB04B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB04C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB04D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB04E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB04F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
```

여기서 차례대로,
- `0x3FFB0100`: 기기의 MAC 주소가 저장
- `0x3FFB0200`: ESP-IDF 빌드 버전 정보
- `0x3FFB0300`: Null 공간
- `0x3FFB0400`
    - `3FFB0400: 58 6E 42 17 33 9A 1D FB 98 C8 D8 1C 4D 4B 6D 0E`
    - `3FFB0410: 3E A8 C6 41 E5 7E 69 22 E1 33 22 95 E8 B5 56 BC`

이 부분이 이상해서 해시인가? 키인가? 싶어서 토큰으로 사용해봤는데 안 되는 것 같다. 그래서 메모리를 더 깊이 파봤다.

```bash
WIRED-MED> 3FFB0500: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0510: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0520: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0530: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0540: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0550: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0560: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0570: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0580: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0590: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB05A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB05B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB05C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB05D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB05E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB05F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
WIRED-MED> 3FFB0600: 48 45 41 50 3A 20 66 72  65 65 3D 31 32 34 38 33   |HEAP: free=12483|
3FFB0610: 32 20 61 6C 6C 6F 63 3D  38 39 32 31 36 20 74 6F   |2 alloc=89216 to|
3FFB0620: 74 61 6C 3D 32 31 34 30  34 38 00 00 00 00 00 00   |tal=214048......|
3FFB0630: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0640: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0650: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0660: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0670: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0680: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0690: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB06A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB06B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB06C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB06D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB06E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB06F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
WIRED-MED> 3FFB0700: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0710: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0720: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0730: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0740: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0750: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0760: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0770: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0780: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0790: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB07A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB07B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB07C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB07D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB07E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB07F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
WIRED-MED> 3FFB0800: 57 49 52 45 44 2D 4D 45  44 00 00 00 00 00 00 00   |WIRED-MED.......|
3FFB0810: 64 47 67 7A 63 6D 46 77  65 56 39 74 4D 47 52 31   |dGgzcmFweV9tMGR1|
3FFB0820: 62 47 55 39 00 00 00 00  00 00 00 00 00 00 00 00   |bGU9............|
3FFB0830: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0840: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0850: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0860: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0870: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0880: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB0890: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB08A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB08B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB08C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB08D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB08E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
3FFB08F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   |................|
```

이 두 줄의 아스키 값을 이어서 보면 `dGgzcmFweV9tMGR1bGU9`가 되는데, 이건 Base64 인코딩이 된 것 같다.

디코딩을 하면 `th3rapy_m0dule=`가 된다.

`debug auth th3rapy_m0dule=`을 입력하니까, 아래와 같이 디버그 모드 진입에 성공할 수 있었다.

![light_mode_only](./assets/img/2026-03-31/image4.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark_mode_only](./assets/img/2026-03-31/image4.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }

이제 NVS를 직접 까볼 차례다.

nvs를 입력하니 usage가 나왔고, 각 명령어를 입력하니 아래와 같이 출력되었다.

```bash
WIRED-MED> NVS operations:
  nvs list               - List all NVS entries
  nvs read <key>         - Read NVS entry value
  nvs namespaces         - List namespaces
WIRED-MED> Namespace: wired_med
  Key: device_id        Type: str   Value: WM-0042
  Key: wifi_ssid        Type: str   Value: WIRED-MED-AP
  Key: wifi_pass        Type: str   Value: SainteMika2024!
  Key: mqtt_broker      Type: str   Value: wired-med.local
  Key: mqtt_port        Type: u16   Value: 1883
  Key: fw_version       Type: str   Value: v2.3
  Key: patient_data     Type: blob  Size: 64 bytes
  Key: crypto_flag      Type: blob  Size: 33 bytes
WIRED-MED> NVS Namespaces:
  - wired_med
```

이 중 누가 봐도 flag인 crypto_flag가 보인다.

nvs read crypto_flag을 입력하니 아래와 같이 나왔다.

```bash
WIRED-MED> NVS [wired_med] crypto_flag (blob, 33 bytes):
00000000: 12 1A 02 0C 08 18 07 29  30 70 25 3D 0D 2B 32 24   |.......)0p%=.+2$|
00000010: 16 34 29 70 24 21 0D 21  75 24 2A 62 33 77 25 30   |.4)p$!.!u$*b3w%0|
00000020: 2F                                                |/|
```

먼저 flag가 ESPILON~ 으로 시작하니 이걸 16진수로 바꾸면 `45 53 50 49 4C 4F 4E 7B`와 같다.

내가 찾은 암호문의 첫 8바이트는 `12 1A 02 0C 08 18 07 29`와 같다.

이걸 각각 XOR 연산을 하면, (LLM 시켰다 ㅎㅎ)
- `0x12 ^ 0x45 = 0x57` ('**W**')
- `0x1A ^ 0x53 = 0x49` ('**I**')
- `0x02 ^ 0x50 = 0x52` ('**R**')
- `0x0C ^ 0x49 = 0x45` ('**E**')
- `0x08 ^ 0x4C = 0x44` ('**D**')
- `0x18 ^ 0x4F = 0x57` ('**W**')
- `0x07 ^ 0x4E = 0x49` ('**I**')
- `0x29 ^ 0x7B = 0x52` ('**R**')

이렇게 계속 반복된다. 바로 WIRED가 5글자의 key인 것이다.

이제 암호문 전체 33바이트를 WIRED로 차례대로 XOR 하면 다음과 같이 나온다. (이것도 귀찮아서 LLM 돌림 ㅎㅎ)

- **Cipher:** `12 1A 02 0C 08 18 07 29 30 70 25 3D 0D 2B 32 24 16 34 29 70 ...`
- **Key:** `W I R E D W I R E D W I R E D W I R E D ...`
- **Plain:** `E S P I L O N { u 4 r t _ n v s _ f l 4 ...`

최종 Flag가 나왔다. 

**ESPILON{u4rt_nvs_fl4sh_d1sc0v3ry}**