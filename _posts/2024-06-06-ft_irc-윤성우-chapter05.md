---
title:  "[ft_irc] TCP 기반 서버 & 클라이언트 2"
excerpt: "윤성우 열혈 TCP/IP 소켓 프로그래밍 챕터 05"

categories:
  - 42seoul
tags:
  - [42seoul, ft_irc, socket, network]

toc: true
toc_label: "42seoul"
# toc_icon: "bars"
toc_sticky: true
 
date: 2024-06-06 13:40:00 +0900
# last_modified_at: 2024-06-05 11:00:01 +0900

---


## 05-1 에코 클라이언트 완벽 구현
 TCP의 전송 특성에 맞게 클라이언트를 재구성해야한다.

### 에코 클라이언트만 문제가 있나요?
앞선 에코 클라이언트에서는 1 write - 1 read 를 통해 데이터를 수신하려고 한다. 하지만 이는 데이터 전송 방법 상 옳지 않은 방법이다. 

에코 클라이언트는 자신의 말을 서버로 보내고 그 말이 그대로 자신에게 돌아오는 시스템이다.
즉, 클라이언트는 자신이 보낸 데이터의 바이트를 알고 있다.
-> 자신이 보낸 바이트 모두 수신될때까지 반복문으로 `read()` 하여 문제를 해결 !

### 계산기 서버 / 클라이언트 구현해보기
```c
// op_server.c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 1024
#define RLT_SIZE 4
#define OPSZ 4

int main(int argc, char **argv)
{
	int serv_sock, clnt_sock;
	char opinfo[BUF_SIZE];
	int result, opnd_cnt, i;
	int recv_cnt, recv_len;
	struct sockaddr_in serv_adr, clnt_adr;
	socklen_t clnt_adr_sz;
	
	if (argc != 2)
	{
		printf("not a correct usage\n");
		exit(1);
	}
	
	serv_sock = socket(PF_INET, SOCK_STREAM, 0);
	if (serv_sock == -1)
		error_handling("socket function error\n");
	
	memset(&serv_adr, 0, sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
	serv_adr.sin_port = htons(atoi(argv[1]));
	
	if (bind(serv_sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
		error_handling("bind function error\n");
	if (listen(serv_sock, 5) == -1)
		error_handling("listen function error\n");
	
	clnt_adr_sz = sizeof(clnt_adr);
	
	for (i = 0; i < 5; i++)
	{
		opnd_cnt = 0;
		clnt_sock = accept(serv_sock, (struct sockaddr *)&clnt_adr, clnt_adr_sz);
		read(clnt_sock, &opnd_cnt, 1);
		recv_len = 0;
		while ((opnd_cnt * OPSZ + 1) > recv_len)
		{
			recv_cnt = read(clnt_sock, &opinfo[recv_len], BUF_SIZE - 1);
			recv_len += recv_cnt;
		}
		result = calc(opnd_cnt, (int*)opinfo, opinfo[recv_len - 1]);
		write(clnt_sock, (char*)&result, sizeof(result));
		close(clnt_sock);
	}
	
	close(serv_sock);
	return 0;
}
```

위  코드에서 for문이 5 번의 연결요청을 `accept()` 한다.
하나의 클라이언트가 보낸 데이터에 대한 결과를 `calc()` 로 계산하여 송신한다.
결과를 한 번의 `write()`로 다시 클라이언트에 전달해준다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

#define BUF_SIZE 1024
#define RLT_SIZE 4
#define OPSZ 4

int main(int argc, char **argv)
{
	int sock;
	char opmsg[BUF_SIZE];
	int result, opnd_cnt, i;
	struct sockaddr_in serv_adr;
	
	if (argc != 3)
	{
		printf("not a correct usage\n");
		exit(1);
	}
	
	sock = socket(PF_INET, SOCK_STREAM, 0);
	if (sock == -1)
		error_handling("socket function error\n");

	memset(&serv_adr, 0, sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
	serv_adr.sin_port = htons(atoi(argv[2]));
  
	if (connect(sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
		error_handling("connect function error\n");
	else
		puts("Connected..............");

	fputs("Operand count: ", stdout);
	scanf("%d", &opnd_cnt);
	opmsg[0] = (char)opnd_cnt;
	for (i = 0; i < opnd_cnt; i++)
	{
		printf("Operand %d: " , i + 1);
		scanf("%d", (int*)&opmsg[i * OPSZ + 1]);
	}
	fgetc(stdin); // delete remained \n in buffer.
	fputs("Operator: ", stdout);
	scanf("%c", &opmsg[opnd_cnt * OPaSZ + 1]);
	write(sock, opmsg, opnd_cnt * OPSZ + 2);
	read(sock, &result, RLT_SIZE);
	printf("Operation result: %d \n", result);
	close(sock);
	return 0;
}
```

 데이터를 누적하여 송수신할 것이기 때문에 배열을 통해 연속적인 메모리공간으로 생성하는 것이 좋다.   
또한 수신할 데이터 크기가 4 바이트이므로 한 번의 `read()`로 충분히 받을 수 있다.   

하나의 배열에 다양한 종류의 데이터를 저장해서 전송하려면, char형 배열을 선언하여야함. 플랫폼에 따라 데이터 타입 호환성이 좋고, 메모리 레이아웃이 8비트를 사용하기 때문에 일관되고 데이터 관리가 편하다.

## TCP의 이론적인 이야기
### TCP 소켓에 존재하는 입출력 버퍼    

- 입출력 버퍼는 TCP 소켓 각각에 대해 별도로 존재한다.
- 입출력 버퍼는 소켓생성시 자동으로 생성된다.
- 소켓을 닫아도 출력버퍼에 남아있는 데이터는 계속해서 전송이 이뤄진다.
- 소켓을 닫으면 입력버퍼에 남아있는 데이터는 소멸된다.

슬라이딩 윈도우(sliding window) : 소켓 간 데이터 송수신에 대해 확인을 진행한다는 프로토콜.
TCP에는 슬라이딩 윈도우가 존재해서 버퍼가 차고 넘쳐서 데이터가 소멸되는 일이 없다. 정확히 받을 수 있는 데이터만큼을 수신한다.

> `write()`가 반환디는 시점
> : 전송할 데이터가 출력버퍼로 이동이 완료되는 시점.    

TCP 의 경우 출력버퍼로 이동된 데이터의 전송을 보장.   
-> "write 함수는 데이터의 전송이 완료되어야 반환된다." 라고 표현함.

### TCP의 내부 동작원리 1: 상대 소켓과의 연결
소켓은 **전 이중(Full-duplex) 방식**으로 동작하므로 양뱡향으로 데이터를 주고 받는다.
연결 과정에서 TCP 소켓은 3 번의 대화를 주고받음. (**Three-way handshaking**)
> SYN : 데이터 전송을 위한 동기화 메시지 전송(A -> B)   
> SYN + ACK : 동기화 메시지 + 응답 메시지.   
> ACK : 응답 메시지 전달

이렇게 총 3 회의 주고받음을 통해 데이터 송수신의 준비완료를 확인함.   

### TCP의 내부 동작원리 2: 상대 소켓과의 데이터 송수신
마찬가지로 SEQ와 ACK에 패킷 번호를 부여해서 올바르게 송수신이 이뤄지는지를 체크함   
> ACK num = SEQ num + sended bytes + 1

이렇게 하면 데이터가 보낸만큼 잘 받았는지 확인할 수 있다.   
ACK 응답을 요구하는 패킷 전송 시, 타이머가 작동한다. 제한 시간 내 ACK가 오지 않으면 타임아웃이 발생하여 다시한번 패킷 재전송.   

### TCP의 내부 동작원리 3: 상대 소켓과의 연결종료

종료를 알리는 `FIN`메시지를 주면 그에 대한 ACK를 답한다.
각 소켓은 서로에게 `FIN`메시지를 전달하고 그에 대한 확인 패킷을 전달하므로 **총 4회**의 교류가 발생. (**Four-way handshaking**)
