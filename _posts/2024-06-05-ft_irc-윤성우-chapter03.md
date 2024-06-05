---
title:  "[ft_irc] 주소체계와 데이터 정렬"
excerpt: "윤성우 열혈 TCP/IP 소켓 프로그래밍 챕터 03"

categories:
  - 42seoul
tags:
  - [42seoul, ft_irc, socket, network]

toc: true
toc_label: "42seoul"
# toc_icon: "bars"
toc_sticky: true
 
date: 2024-06-05 16:12:00 +0900
# last_modified_at: 2024-06-05 11:00:01 +0900

---

## 03-1 소켓에 할당되는 IP주소와 PORT 번호
### 인터넷 주소 (Internet Address)
> IP 주소: 데이터 송수신 목적으로 컴퓨터에 부여되는 값.
> 	- IPv4 : 4바이트 IP 주소체계
> 	- IPv6 : 16바이트 IP 주소체계

IPv4 기준으로 했을 때, 4 바이트의 주소는 네트워크 주소와 호스트(컴퓨터) 주소로 나뉜다.    

데이터가 전송될 때, 우선적으로 네트워크 주소로 찾아가게 된다. 데이터가 네트워크 주소에 도착하면 라우터 또는 스위치를 만나게되고, 해당 기기들이 호스트 주소에 맞는 컴퓨터로 데이터를 전송해준다.

### 소켓 구분에 활용되는 PORT 번호
 IP 주소를 통해 컴퓨터까지 도착했다고 치자. 여기서 끝나면 안된다. 왜냐하면 해당 데이터를 사용하는 응용 프로그램으로 전송해줘야하기 때문이다. 따라서 각 프로그램들은 데이터를 제공받을 소켓을 가지게 된다.
-> 이 **소켓을 구분해주는 것**이 PORT 번호이다.

운영체제가 포트 번호를 활용하여 수신된 데이터를 알맞게 응용프로그램으로 보내준다.

- 범위 : 0 ~ 65535 (16비트)
	+ 0 ~ 1023 -> 특정 프로그램에 할당 예약되어 임의로 사용 불가. 이 외의 포트번호 사용해야함.
	+ 동일한 소켓 간 동일 포트 번호로 중복되어선 안됨.(단, TCP 소켓과 UDP 소켓은 상관없음)

데이터 전송의 목적지 주소: IP주소 + Port 번호로 구성.
## 03-2 주소정보의 표현
### IPv4 기반 주소표현을 위한 구조체
```c
struct sockaddr_in
{
	sa_family_t sin_family;
	uint16_t sin_port;
	struct in_addr sin_addr;
	char sin_zero[8];
};

struct in_addr
{
	in_addr_t s_addr; // 32 비트 IPv4 인터넷주소. uint32_t 로 정의됨
};
```
- `sin_family` : 주소체계
- `sin_port` : 16비트 TCP/UDP PORT 번호
- `sin_addr` : 32비트 IP주소
- `sin_zero[8]` : 사용되지 않음.

*_t  는 추후 자료형의 확장성을 고려한 자료형.

### 구조체 `sockaddr_in` 의 멤버에 대한 분석
> 멤버 `sin_family`

프로토콜 체계마다 적용하는 주소체계가 다르다. ex) IPv4 : 4바이트, IPv6 : 16바이트

> 멤버 `sin_port`

16비트 포트번호 저장
**네트워크 바이트 순서**로 저장해야함.

> 멤버 `sin_addr`

32비트 IP주소정보를 저장
역시 **네트워크 바이트 순서**로 저장.

> 멤버 `sin_zero`

단순히 구조체 `sockaddr_in`의 크기를 구조체 `sockaddr`와 일치시키기 위해 삽입된 멤버.
** 반드시 `0` 으로 채워야한다 (주의) **

`bind()` 는 기존 존재하던 구조체인 `struct sockaddr` 에 대해 IP주소와 포트번호를 포함시킨 뒤 남은 정보를 0으로 채우도록 지시하는데, 이는 매우 불편한 요구.
-> `sockaddr_in` 의 `sin_zero`를 통해 0을 채우는 상황을 구현한 것임.

## 03-3 네트워크 바이트 순서와 인터넷 주소 변환
`호스트 바이트 순서`: CPU마다 데이터의 바이트를 저장하는 순서가 다르다(리틀엔디안, 빅엔디안)

### 바이트 순서와 네트워크 바이트 순서
`네트워크 바이트 순서`: 네트워크를 통해 데이터를 전송할 때 지켜져야할 데이터 전송 순서
> 네트워크 상으로 데이터를 전송할 때는 데이터 배열을 "빅 엔디안" 기준으로 송수신하기로 약속.

### 바이트 순서의 변환(Endian Conversions)
앞선 이유로 `sockaddr_in`구조체 변수에 값을 채우기 앞서 네트워크 바이트 순서로 변환하여 저장해야한다.

```c
unsigned shrot htons(unsigned short);
unsigned short htonl(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```
`h`는 호스트 바이트 순서, `n`은 네트워크 바이트 순서를 의미한다. `s` 는 short, `l`은 long 을 의미한다.
해당 함수 모두 `ft_irc` 허용 함수이다.

## 03-4 인터넷 주소의 초기화와 할당

### 문자열 정보를 네트워크 바이트 순서의 정수로 변환하기
`sockaddr_in`의 `sin_addr`은 32비트 정수형으로 되어있다. 
일반적으로 `127.0.0.1` 을 문자열로 받는데, 이를 정수형으로 바꿔주는 함수가 있다.
```c
#include <arpa/inet.h>
in_addr_t inet_addr(const char * string);
// 성공 시 빅 엔디안으로 변환된 32비트 정수값, 실패 시 INADDR_NONE 변환
```
유효하지 못한 주소값에 대해서는 `INADDR_NONE` 을 반환해 오류 검출하는 능력도 있다. 또한 자동으로 네트워크 바이트 순서로 변환해주기도 한다. `inet_addr` 를 해준 뒤, 반환값을 `in_addr` 구조체 변수에 대입해주는 과정이 필요하다. 하지만,

```c
#include <arpa/inet.h>
int inet_aton(const char * string, struct in_addr * addr);
```
 `inet_addr` 과 동일한 기능을 하지만 변환된 값을 `in_addr` 구조체에 바로 담기 때문에 활용도가 높다.

```c
#include <arpa/inet.h>
char * inet_ntoa(struct in_addr adr);
// 성공 시 문자열의 주소 값, 실패 시 -1
```
네트워크 바이트 순서로 정렬된 정수형 IP주소 정보를 호스트 바이트 순서로 정렬하여 문자열로 돌려준다.

### 인터넷 주소 초기화
```c
struct sockaddr_in addr;
char *serv_ip = "211.217.168.13"  // IP주소 문자열 선언
char *serv_port = "9190";  // 포트번호 문자열 선언
memset(&addr, 0, sizeof(addr)); // sockaddr_in 초기화
addr.sin_family = AF_INET; // 주소 체계 지정
addr.sin_addr.s_addr = inet_addr(serv_ip); // IP주소 초기화
addr.sin_port = htons(atoi(serv_port)); // 포트번호 초기화
```
일반적으로 위 코드와 같이 초기화를 진행.
하지만 , IP주소나 포트번호는 메인 함수의 인자로 받아서 처리하게 될 것. (주소 서버 프로그램)

### 클라이언트 주소정보 초기화
서버 : `sockaddr_in` 구조체에 IP / PORT 번호로 초기화. -> `bind()`
클라 : `sockaddr_in` 구조체에 IP / PORT 번호로 초기화. -> `connect()`


### `INADDR_ANY`
서버 소켓의 경우, 자신이 속한 컴퓨터의  IP 주소로 초기화되어야한다.
NIC(랜카드) 갯수에 따라 IP주소가 결정되고 여러개가 될 수 있기때문에 서버 소켓도 IP주소의 초기화가 필요하지만 하나의 NIC를 가지는 경우 유일한 서버주소로 정해져있다.
-> `INADDR_ANY` 를 이용하면 소켓이 동작하는 컴퓨터 IP주소가 자동 할당된다.

irc의 경우 한 컴퓨터로 이루어지니 루프백 주소를 사용한다.

### 소켓에 인터넷 주소 할당
`bind()` : 초기화된 주소 정보를 소켓에 할당 !
```c
#include <sys/socket.h>
int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
// 성공 시 0, 실패 시 -1
```
*  `sockfd` : 주소정보 할당받을 소켓의 파일 디스크럽터
* `myaddr` : 할당하고자 하는 주소정보를 지니는 구조체 변수 주소값
* `addrlen` : 두 번째 인자로 전달된 구조체 변수 길이 정보.

#### 서버 프로그램의 기본 코드 구성
```c
int serv_sock;
struct sockaddr_in serv_addr;
char *serv_port = "9190";

/* 서버 소켓(리스닝 소켓) 생성 */
serv_sock = socket(PF_INET, SOCK_STREAM, 0);

/* 주소 정보 초기화 */
memset(&serv_addr, 0, sizeof(serv_addr));
serv_addr.sin_family(AF_INET);
serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
serv_addr.sin_port = htons(atoi(serv_port));

/* 주소 정보 할당 */
bind(serv_sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
```

---
해당 장에서는 ft_irc에서 사용 가능한 함수들을 많이 알게되어 어떻게 활용해야하는지를 배울 수 있었다.
기본적인 서버 프로그램의 뼈대에 대해 보았으니 추후 프로그램 구성에 도움이 될 듯 하다!