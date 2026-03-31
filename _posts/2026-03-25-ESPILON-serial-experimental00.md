---
title: ESPILON CTF (hardware)Serial Experimental 00 write-up
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
**Serial Experimental 00 -- Lain Maintenance Node**

You gained access to a split UART debug interface from a WIRED-MED prototype.

- TX (read): `tcp/<host>:1111`
- RX (write): `tcp/<host>:2222`

Investigate serial diagnostics, recover the maintenance token, then unlock the node.

Format: `ESPILON{...}`
```

# 2. 풀이

우선 주어진 주소로 각각 연결해보았다.

문제를 보니 숨겨진 유지보수 토큰(Maintenance Token)을 찾아내고, 이를 이용해 노드를 잠금 해제해서 플래그를 얻으면 되는 문제 같다.

먼저 RX창에 help, diag, status라고 입력했을 때 TX 창의 출력을 확인하도록 하겠다.

```bash
> Commands:
  help
  whoami
  diag.uart
  diag.eeprom
  diag.order
  unlock <token>

> node: serial-experimental-00
> [UART] boot trace restored
[UART] frag_a_hex=4c41494e
[UART] frag_c_hex=3030
[UART] note: hex => ASCII
> [EEPROM] page 0x13
[EEPROM] frag_b_xor_hex=4056415a525f
[EEPROM] xor_key=0x13
> [ORDER] token format: <A>-<B>-<C>
[ORDER] A from diag.uart frag_a_hex
[ORDER] B from diag.eeprom frag_b_xor_hex (decode XOR)
[ORDER] C from diag.uart frag_c_hex
```

이 출력에서 나온 hex 값들을 분석하도록 하겠다.

1. `4c41494e` = LAIN
2. `3030` = 00
3. `4056415a525f` = SERIAL
- 이 부분은 키 값인 0x13을 각각 XOR 연산해야 값이 나온다. 

최종적으로, 위의 로그에서 알려준 형식에 대입하면 `LAIN-SERIAL-00`와 같다.

이제 RX 창에서 `unlock LAIN-SERIAL-00`를 입력했더니 Flag가 출력됐다.

**ESPILON{l41n_s3r14l_3xp_00}**

![light_mode_only](./assets/img/2026-03-31/image6.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark_mode_only](./assets/img/2026-03-31/image6.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }