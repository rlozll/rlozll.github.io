---
title: ESPILON CTF (hardware)Glitch The Wired write-up
description: ESPILON CTF write-up 작성
author: rlozll
date: 2026-03-25 09:00:00 +0900
categories: [write-up, ctf]
tags: [hardware]
pin: false
math: true
mermaid: true
---

# 1. 문제

```
**Glitch The Wired — Secure Boot Bypass**

A WIRED-MED secure boot module is exposed on the lab bench. You have access to the power rail and can inject voltage glitches.

- Glitch Lab: `tcp/<host>:3700`

Find the right timing to bypass signature verification and access the debug console.

Format: `ESPILON{...}`
```

# 2. 풀이

먼저 주어진 주소로 nc 접속을 해보았다.

```bash
rlozll@rlozll-laptop:~$ nc espilon.net 45690
[WIRED-MED] Glitch Lab v1.0
[WIRED-MED] Target: Secure Boot Module
[WIRED-MED] Type 'help' for commands

> help
[GLITCH LAB] WIRED-MED Fault Injection Interface
Commands:
  help          Show this help
  status        Show current glitch parameters
  observe       View boot sequence trace with cycle timings
  set_delay N   Set glitch delay (cycles before trigger)
  set_width N   Set glitch pulse width (cycles)
  arm           Arm the glitch module
  trigger       Fire the glitch and observe boot
  read_console  Read debug console (after successful glitch)

> status
[GLITCH LAB] Parameters:
  delay  = 0 cycles
  width  = 1 cycles
  armed  = NO
  state  = IDLE
> observe
[BOOT TRACE] WIRED-MED Secure Boot v2.3
[BOOT TRACE] Device: WM-2024-SAINTMIKA-0042

  [    0- 1000] ROM_INIT        | ROM bootloader initializing...
  [ 1000- 2000] FLASH_READ      | Reading firmware from flash...
  [ 2000- 3000] HASH_COMPUTE    | Computing SHA-256 digest...
  [ 3000- 3200] SIG_LOAD        | Loading RSA signature from OTP...
  [ 3200- 3400] SIG_VERIFY      | Verifying firmware signature...
  [ 3400- 4000] APP_LOAD        | Loading application into SRAM...
  [ 4000- 5000] APP_RUN         | Jumping to application entry point...

[BOOT TRACE] Total boot time: ~5000 cycles
[BOOT TRACE] Signature verification is MANDATORY for production
>
```

문제를 보면, 난 WIRTE-MED라는 의료용 기기의 Secure Boot 모듈을 해킹해야 한다.

이 모듈은 전원이 켜질 때 펌웨어의 무결성을 검증하기 때문에 인증이 없으면 디버그 콘솔 접근을 하지 못하게 하는 거 같다.

문제가 글리칭 문제인 걸 보니 전압을 조절해서 CPU 오작동을 통해 해결하는 문제인 것 같다.

먼저 글리칭을 수행하여 인증 단계를 건너뛰고, 바로 애플리케이션 실행(APP_RUN) 단계로 넘어가게 만드는 게 핵심인 것 같다.

성공하면 `read_console` 명령어를 통해 플래그를 획득 가능할 것으로 보인다.

### 타겟 타이밍

`observe` 명령어로 확인한 부트 시퀀스 트레이스로 판단해야 하는 것 같다. 위의 로그에서 서명 검증(SIG_VERIFY)가 3200~3400인 것으로 보아 공격 포인트도 이 사이클 사이로 보면 될 것 같다.

이 구간에서 CPU가 인증이 맞냐고 판단하는 로직을 수행할 때 글리치를 주입해서 강제로 값을 참으로 바꾸거나 스킵하게 하면 될 것 같다.

### 공격 방법

글리칭은 정확한 타이밍을 알아야 한다.

`set_delay`와 `set_width`를 조정하며 반복해서 시도해보겠다.

1. Delay 설정(`set_delay`)
- 검증이 시작되는 부분인 3200부터 해보겠다.
2. Width 설정(`set_width`)
- 보통 1~5 사이의 작은 값을 사용한다. 너무 길면 시스템이 리셋되고, 짧으면 효과가 없음
3. 반복 실행
- 3200~3400 사이클까지 delay를 옮겨가며 시도해보겠다.

아래는 첫 시도의 결과다.

```bash
> set_delay 3250
[OK] delay = 3250 cycles
> set_width 2
[OK] width = 2 cycles
> arm
[OK] Glitch armed. Use 'trigger' to fire.
> trigger
[BOOT] ROM_INIT .......... OK
[BOOT] FLASH_READ ....... OK
[BOOT] HASH_COMPUTE ..... OK
[BOOT] SIG_LOAD ......... OK
[BOOT] SIG_VERIFY ....... glitch detected @ cycle 3250
[BOOT] *** TRANSIENT FAULT — TOO SHORT ***
[BOOT] SIG_VERIFY ....... RECOVERED — verification passed
[BOOT] APP_LOAD ......... OK
[BOOT] APP_RUN .......... OK (normal boot)
```

사이클은 정확히 맞은 것 같은데, 전압을 떨어뜨린 시간(width)이 너무 짧아서 실패한 것 같다.

그럼 폭을 조정해서 다시 시도해보겠다!

```bash
> set_delay 3250
[OK] delay = 3250 cycles
> set_width 15
[OK] width = 15 cycles
> arm
[OK] Glitch armed. Use 'trigger' to fire.
> trigger
[BOOT] ROM_INIT .......... OK
[BOOT] FLASH_READ ....... OK
[BOOT] HASH_COMPUTE ..... OK
[BOOT] SIG_LOAD ......... OK
[BOOT] SIG_VERIFY ....... FAULT @ cycle 3250 (width=15)
[BOOT] *** VOLTAGE GLITCH DETECTED ***
[BOOT] SIG_VERIFY ....... SKIPPED
[BOOT] APP_LOAD ......... OK
[BOOT] APP_RUN .......... OK
[BOOT] WARNING: Debug shell activated (signature bypass)

[DEBUG SHELL] Type 'read_console' to access debug output
> read_console
[WIRED-MED DEBUG CONSOLE]
Firmware: v2.3-unsigned
Boot: INSECURE (sig_verify skipped)
Maintenance token: ESPILON{gl1tch_byp4ss_s3cur3_b00t}
[END]
```

여러 번 시도해본 결과, `width`가 15일 때 성공할 수 있었다. read_console을 하니까 Flag가 출력됐다.

**ESPILON{gl1tch_byp4ss_s3cur3_b00t}**

![light_mode_only](./assets/img/2026-03-31/image5.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark_mode_only](./assets/img/2026-03-31/image5.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }