---
title:  "[ft_irc] 소켓의 우아한 연결 종료"
excerpt: "윤성우 열혈 TCP/IP 소켓 프로그래밍 챕터 07"

categories:
  - 42seoul
tags:
  - [42seoul, ft_irc, socket, network]

toc: true
toc_label: "42seoul"
# toc_icon: "bars"
toc_sticky: true
 
date: 2024-06-06 15:32:00 +0900
# last_modified_at: 2024-06-05 11:00:01 +0900

---

## 07-1 TCP 기반의 Half-close

### 일방적인 연결종료 문제점

`close()`를 통해 소켓을 닫아버리면, 필수적으로 받아야할 데이터를 받지 못하거나, 보내지 못하는 경우가 생긴다.    
-> `Half-close`하여 수신 또는 송신만 닫아서 문제 발생을 줄인다.

### 소켓과 스트림
두 소켓이 연결되면 데이터의 송 수신이 가능한 두 개의 스트림이 생성된다.   
스트림은 단 방향이므로 A -> B / B -> A로 각각 존재한다.

Half-close는 결국 **두 스트림 중 하나의 스트림만 먼저 종료하는 상태**를 의미한다.

```c
#include <sys/socket.h>
int shutdown(int sock, int howto);
// 성공 시 0, 실패 시 -1 반환
```

- 첫 번째 인자 : Half-close할 소켓   
- 두 번째 인자 : `SHUT_RD` 입력 스트림 종료, `SHUT_WR` 출력 스트림 종료, `SHUT_RDWR` 입출력 스트림 종료   

### Half-close의 필요성
두 소켓이 파일을 주고 받는다고 할 때, 파일을 다 보냈다면 EOF를 날려서 더이상 보낼 파일이 없음을 알려야함.
이를 소켓을 닫는 방법으로 EOF를 전달할 수 있는데, `close`로 냅다 닫으면 파일을 받고 있던 상대 소켓도 닫혀서 데이터를 다 받지 못하는 에러가 발생할 수 있음.    

이를 위해 Half-close를 하여 소켓의 출력 스트림을 닫으면, 상대 소켓에 EOF가 들어가게 된다.   
이후 상대 소켓이 모든 데이터를 받고 ACK를 보내면 이 데이터는 입력 스트림으로 들어오게 되어 데이터 흐름 제어에 문제가 없게 된다.

