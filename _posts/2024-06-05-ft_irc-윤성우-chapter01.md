---
title:  "[ft_irc] 네트워크 프로그래밍과 소켓 이해"
excerpt: "윤성우 열혈 TCP/IP 소켓 프로그래밍 챕터 01"

categories:
  - 42seoul
tags:
  - [42seoul, ft_irc, socket, network]

toc: true
toc_label: "42seoul"
# toc_icon: "bars"
toc_sticky: true
 
date: 2024-06-05 15:26:00 +0900
# last_modified_at: 2024-06-05 11:00:01 +0900

---

## 01-1 네트워크 프로그래밍과 소켓의 이해
### 네트워크 프로그래밍과 소켓에 대한 매우 간단한 이해
* 네트워크 프로그래밍 : 서로 다른 두 컴퓨터가 정보를 주고받을 수 있도록 하는 것.
	-> 기기 간 연결 + 데이터 송수신
	기기 간의 연결은 인터넷이라는 거대한 연결망으로 되어있음.
	데이터 송 수신을 어떻게 할지가 관건.
		전화기를 통해 목소리를 주고받듯, `소켓`을 통해 데이터를 주고받는다.

### 전화 받는 소켓의 구현 (서버 소켓 or 리스닝 소켓)
- `socket` : 데이터를 주고받을 **소켓 생성 함수**.
- `bind` : 갓 만든 소켓을 **IP주소와 port번호를 할당**하여 연결망에 __연결__. 
- `listen` : 소켓으로의 **연결 요청을 받아들이도록** 기다리도록 한다.
- `accept` : 연결 요청받은 소켓이 **연결을 수락**한다.

### 전화 거는 소켓의 구현(클라이언트 소켓)
동일하게 소켓을 생성하지만, 위와는 달리 `connect()` 함수를 통해 전화를 걸 클라이언트 소켓으로 구현한다.

## 01-2 리눅스 기반 파일 조작하기
**_ 리눅스에서의 소켓 조작 == 파일 조작_**
-> 파일 입출력 함수를 소켓 입출력, **데이터 송수신** 에 사용할 수 있다. (윈도우는 별개)

### 저 수준 파일 입출력과 FD
- File Descriptor : 할당받은 파일 또는 소켓에 부여된 정수 번호
	 소켓의 경우 생성될 때 FD 값을 부여받는다.

### 파일 열기 / 닫기 / 쓰기 / 읽기
`#include <unistd.h>` 필요

```c
int open(const char *path, int flag);
```
* `path` : 파일 이름
* `flag` : 파일의 오픈 모드 설정(비트 연산자 or로 묶어서 전달 가능)

```c
int close(int fd);
// fd는 닫을 FD

ssize_t write(int fd, const void * buf, size_t nbytes);
ssize_t read(int fd, void *buf, size_t nbytes);

```


