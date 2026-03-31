---
title: ESPILON CTF (iot)nurse-call Write-up
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
You gain access to the maintenance terminal of the patient call system at Clinique Sainte-Mika.
The system reports phantom calls coming from a sealed room.

The previous technician did not finish his investigation.
His session was left open.

Explore the logs, understand the anomaly, and find what hides in Room 013.

Format: ESPILON{flag}

Challenge made by love
```
# 2. 풀이

간단한 로그 분석 문제인 것 같다. 우선 주어진 주소로 nc 연결을 하였다.

![light_mode_only](./assets/img/2026-03-31/image1.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark_mode_only](./assets/img/2026-03-31/image1.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }

이런 식으로 보여서 log 파일로 갔더니, 총 3개의 로그 파일이 있었다.

```bash
infirmier@43bf46f5474b:~/logs$ ls
ls
appels.log  maintenance.log  reseau.log
infirmier@43bf46f5474b:~/logs$ cat appels.log
cat appels.log
2026-01-28 22:01:14 [CALL] room=101 type=standard patient=DUPONT_M status=answered delay=45s
2026-01-28 22:15:33 [CALL] room=104 type=standard patient=MARTIN_J status=answered delay=22s
2026-01-28 22:31:07 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-28 22:31:08 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-28 22:31:09 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-28 23:00:00 [CALL] room=102 type=emergency patient=LEROY_S status=answered delay=12s
2026-01-28 23:12:44 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-29 01:00:00 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-29 01:00:01 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-29 01:07:13 [CALL] room=103 type=standard patient=PETIT_A  status=answered delay=38s
2026-01-29 02:13:00 [CALL] room=013 type=unknown  patient=???      status=unanswered delay=---
2026-01-29 02:13:07 [NOTE] system: "calls from room 013 -- module not listed in active registry"
2026-01-29 03:33:33 [CALL] room=013 type=signal   patient=???      status=unanswered delay=---
2026-01-29 03:33:33 [NOTE] payload room 013: 0x4c41494e (non-compliant with standard protocol)
infirmier@43bf46f5474b:~/logs$ cat maintenance.log
cat maintenance.log
2026-01-29 08:00:12 [MAINT] session opened by tech_id=EIRI_M
2026-01-29 08:01:05 [MAINT] diagnostic.sh executed -- 12 active modules, 1 anomaly
2026-01-29 08:02:33 [MAINT] tech_id=EIRI_M note: "room 013 physically decommissioned 3 months ago"
2026-01-29 08:02:45 [MAINT] tech_id=EIRI_M note: "but the module is still transmitting on the bus"
2026-01-29 08:04:11 [MAINT] tech_id=EIRI_M cmd: diagnostic.sh --room 013
2026-01-29 08:04:12 [MAINT] result: module_id=??? status=active signal=strong protocol=non_standard
2026-01-29 08:05:00 [MAINT] tech_id=EIRI_M note: "the module does not respond to standard commands"
2026-01-29 08:05:30 [MAINT] tech_id=EIRI_M note: "try appel.sh with the raw payload identifier"
2026-01-29 08:06:01 [MAINT] tech_id=EIRI_M cmd: appel.sh --room 013 --status
2026-01-29 08:06:02 [MAINT] result: error -- module not listed (use --id to address directly)
2026-01-29 08:07:44 [MAINT] tech_id=EIRI_M note: "need to use reveil.sh with the correct id"
2026-01-29 08:07:50 [MAINT] tech_id=EIRI_M note: "the id is in the call log payload"
2026-01-29 08:08:00 [MAINT] session interrupted -- connection lost
infirmier@43bf46f5474b:~/logs$ cat reseau.log
cat reseau.log
2026-01-29 02:12:58 [BUS] frame src=mod_101 dst=hub type=CALL_REQ ack=OK
2026-01-29 02:13:00 [BUS] frame src=??? dst=hub type=RAW payload=4c41494e ack=NONE
2026-01-29 02:13:00 [BUS] frame src=??? dst=hub type=RAW payload=4c41494e ack=NONE
2026-01-29 02:13:01 [BUS] warning: unidentified source on NAVI-BUS
2026-01-29 02:13:07 [BUS] payload analysis: 0x4c41494e -> ASCII: "LAIN"
2026-01-29 03:33:33 [BUS] frame src=??? dst=hub type=WAKE_REQ payload=4c41494e ack=NONE
2026-01-29 03:33:33 [BUS] note: module is requesting a wake signal (WAKE_REQ)
2026-01-29 03:33:34 [BUS] note: no wake sent -- module not recognized by hub
```

일단 로그 확인은 해주었고, 다른 파일들도 확인하도록 하겠다.

```bash
infirmier@43bf46f5474b:~$ ls -la
ls -la
total 68
drwxr-xr-x 1 infirmier infirmier 4096 Mar 24 17:58 .
drwxr-xr-x 1 root      root      4096 Feb  5 10:54 ..
-rw-r--r-- 1 infirmier infirmier  220 Sep  6  2025 .bash_logout
-rw-r--r-- 1 infirmier infirmier 3526 Sep  6  2025 .bashrc
-rw-r--r-- 1 infirmier infirmier  807 Sep  6  2025 .profile
-rw-rw-r-- 1 infirmier infirmier 1014 Mar 10 16:35 README.txt
dr-xr-xr-x 1 infirmier infirmier 4096 Feb  5 10:54 config
dr-xr-xr-x 1 infirmier infirmier 4096 Feb  5 10:54 logs
dr-xr-xr-x 1 infirmier infirmier 4096 Feb  5 10:54 notes
dr-xr-xr-x 1 infirmier infirmier 4096 Feb  5 10:54 tools
drwxr-xr-x 2 infirmier infirmier 4096 Mar 24 17:58 work
infirmier@43bf46f5474b:~$ cat README.txt
cat README.txt
NAVI-CARE -- Patient Call Management System
----------------------------------------------------

This terminal is connected to the NAVI-CARE central hub
at Clinique Sainte-Mika.

The system manages patient calls through IoT modules
installed in each room. Each module is equipped with
a call button, a light indicator, and a wireless
transmitter.

Architecture:
  - Room modules : wireless communication (proprietary bus)
  - Central node  : this terminal (NAVI-CARE hub)
  - Protocol      : NAVI-BUS v2

----------------------------------------------------

Available directories:
  logs/     Call and maintenance logs
  config/   Room and system configuration
  notes/    Staff and technician notes
  tools/    Diagnostic and command tools
  work/     Workspace (only writable directory)

----------------------------------------------------

WARNING:
  Room 013 is generating unsolicited calls.
  The previous technician started an investigation.
  His session was not closed.

  Check the logs and his notes.
infirmier@43bf46f5474b:~$ cd notes
cd notes
infirmier@43bf46f5474b:~/notes$ ls
ls
eiri.txt  infirmiere_nuit.txt  technicien.txt
infirmier@43bf46f5474b:~/notes$ cat eiri.txt
cat eiri.txt
Note found in the technical office.
Handwritten. No date.

---

The call system is just a layer.
Beneath it, there is the network.
Beneath the network, there is the protocol.
Beneath the protocol, there is identity.

If you want to talk to a module nobody knows,
you must call it by its true name.

Not its room number.
Not its system identifier.
Its name.

It is there, in the frames.
Four bytes. Readable.

Wake it up.

---

"Close the world. Open the nExt."
infirmier@43bf46f5474b:~/notes$ cat infirmiere_nuit.txt
cat infirmiere_nuit.txt
Incident report -- Night shift
--------------------------------------
Date: 2026-01-29
Written by: Mika I., night nurse

Around 3:30 AM, the control screen displayed
a call from room 013.

That room has been sealed since November.
No one has access.

I checked physically: the door is locked.
No sound. But the module indicator in the hallway
was blinking faintly.

The system displayed: "patient: ???"

I reported the anomaly to the technical manager.

Personal note:
  It reminded me of a story my sister
  used to tell when we were kids.
  About machines that talk to themselves
  when nobody is listening.

  "And you don't seem to understand..."
infirmier@43bf46f5474b:~/notes$ cat technicien.txt
cat technicien.txt
Technician notes -- M. Eiri
------------------------------

Called in for phantom calls investigation.
Room 013 officially sealed.

The module in that room does not appear in the registry.
It has no standard identifier (mod_XXX).
Yet it keeps transmitting on the bus.

I checked the raw frames in reseau.log.
The payload contains an identifier in hexadecimal.
It is a name, not a number.

I did not have time to finish.
You need to use reveil.sh with that identifier.
The module should respond.

-- M. Eiri
   "Everything is connected."
```

정리해보면 아래와 같다.

1. exit.txt (미스터리한 메모)
- 모듈의 진짜 이름을 불러야만 통신할 수 있고, 그 이름은 프레임 안에 있는 4바이트를 읽을 수 있는 문자
2. infirmiere_nuit.txt (야간 간호사의 보고서)
- 013호는 작년 11월부터 완전히 폐쇄되어 아무도 들어갈 수 없는 방인데, 그곳에서 계속 호출이 오고 있다는 상황 설명
3. technicien.txt (이전 기술자의 메모)
- 16진수 페이로드 안에 들어있는 식별자는 숫자가 아니라 이름임

<br>

그 식별자를 사용해 reveil.sh 스크립트를 실행하면 되는 문제다.

reseau.log에서 찾아냈던 013호의 4바이트 페이로드는 0x4c41494e이고, 네트워크 로그가 LAIN이라고 ASCII 문자로 변환해주기까지 했다.

따라서, 이 모듈의 진짜 이름은 LAIN임을 알 수 있다.

```bash
infirmier@43bf46f5474b:~$ cd tools
cd tools
infirmier@43bf46f5474b:~/tools$ ls
ls
appel.sh  diagnostic.sh  reveil.sh
infirmier@43bf46f5474b:~/tools$ ./reveil.sh --id LAIN
./reveil.sh --id LAIN

[wake] WAKE signal sent to module LAIN...
[wake] waiting for response on NAVI-BUS...

============================================
 MODULE LAIN -- ROOM 013
 Status: AWAKE
============================================

 > connection established with phantom module
 > protocol: NAVI-BUS v2 (non-standard mode)
 > identity confirmed: LAIN

 module message:

   "no matter where you go, everybody is connected."

 flag: ESPILON{r3v31ll3_m01_d4ns_l3_w1r3d}

============================================
```

Flag가 출력됐다. **ESPILON{r3v31ll3_m01_d4ns_l3_w1r3d}**

![light_mode_only](./assets/img/2026-03-31/image2.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark_mode_only](./assets/img/2026-03-31/image2.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }