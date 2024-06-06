---
title:  "[ft_irc] UDP 기반 서버 & 클라이언트"
excerpt: "윤성우 열혈 TCP/IP 소켓 프로그래밍 챕터 06"

categories:
  - 42seoul
tags:
  - [42seoul, ft_irc, socket, network]

toc: true
toc_label: "42seoul"
# toc_icon: "bars"
toc_sticky: true
 
date: 2024-06-06 14:10:00 +0900
# last_modified_at: 2024-06-05 11:00:01 +0900

---

## 06-1 UDP에 대한 이해
### UDP 소켓의 특성
`SEQ`와 `ACK`를 통한 흐름 제어가 없다는 점이 TCP와의 큰 차이점이다.
그래서 빠른 속도로 데이터를 전송할 수 있고 빠르게 교류가 일어나야하는 실시간 환경에서 도움이된다.   

### UDP 내부 동작 원리

> UDP의 가장 중요한 역할   
> : 호스트로 수신된 패킷을  PORT 정보를 참조하여 최종 목적지인 호스트 내의 UDP 소켓에 전달하는 것.

참고로 A 호스트에서 B 호스트를 찾아가는 과정은 IP의 역할. 그 이후 호스트에서 알맞은 UDP 소켓으로 전달하는 것이 UDP의 역할이다.


## 06-2 UDP 기반 서버 / 클라이언트 구현

* UDP 에서의 서버와 클라이언트는 연결되어있지 않음.
	UDP 소켓의 생성과 데이터 송수신 과정만 존재
- UDP 에서는 서버이건 클라이언트건, 하나의 소켓만 필요.
	마치 하나의 우체통으로 여러 곳의 우편물을 받듯 하나의 UDP 소켓으로 여러 호스트와 통신 가능.

```c
#include <sys/socket.h>
ssize_t sendto(int sock, void *buff, size_t nbytes, int flags, struct sockaddr *to, socklen_t addrlen);

ssize_t recvfrom(int sock, void *buff, size_t nbytes, int flags, struct sockaddr *from, socklen_t *addrlen);

// 두 함수 모두 성공 시 송수신한 바이트 수, 실패 시 -1
```

앞서 TCP 에서는 read, write를 통해 데이터를 입출력했지만, UDP에서는 `sendto()`, `recvfrom()`을 이용한다.

또한 특이한 점으로 UDP 클라이언트는  `connect()`를 사용하지 않음 -> `sendto()`가 연결과정을 포함한다.

## 06-3 UDP의 데이터 송수신 특성과 UDP에서의 connect() 호출

UDP는 **데이터의 경계성**이 있기때문에 1write - 1read 가 이뤄져야한다.
TCP는 바이트 수 만큼 받았는지를 확인했다면, UDP는 보낸 만큼 받았는지를 확인한다.

### `sendto()`의 특징과 connected / unconnected UDP

`sendto()`의 동작은 세 단계로 구분된다.   
> 1단계 : UDP 소켓에 목적지의 IP , PORT 번호 등록   
> 2단계 : 데이터 전송   
> 3단계 : UDP 소켓에 등록된 목적지 정보 삭제   

sendto()는 목적지의 주소정보가 계속 변하기 때문에 하나의 소켓으로 다양한 목적지로 데이터 전송이 가능하다.
목적지 정보가 등록되지 않은 UDP 소켓을 unconnected UDP 라고 한다.

하지만 매번 동일한 목적지로 보낸다면 계속 목적지를 등록하고 삭제하는 과정이 불필요한 상황이다.   
-> `connect()`를 UDP에 사용하게 된다면 UDP의 목적지를 박아줄 수 있다.    
효율적으로 시간단축이 가능하고, 이렇게되면 `sendto()`, `recvfrom()`을 사용하지 않아도 된다. connected UDP에는 `read() / write()`를 사용하자.


