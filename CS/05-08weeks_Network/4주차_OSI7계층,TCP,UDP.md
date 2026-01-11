# OSI Model
OSI 모델은 1984년 ISO에서 제안한 모델로, 컴퓨터 간에 데이터가 송수신되는 과정을 계층 구조로 설명 계층을 세분화함으로써 네트워크 통신이 거치는 과정을 이해 가능
현실의 네트워크 구현과 완전히 대응되지는 않으나, 많은 프로토콜과 기술들이 OSI 모델의 개념을 기반으로 디자인됨


## OSI Model - 7 Layers


![][osi_layer_overview.png]

- **Physical Layer**: 장치 간 물리적 연결과 하드웨어 전송 기술을 다루는 최하위 계층
  - Protocol Data Unit (PDU): Bits
  - 한 node에서 다른 node로 bit를 송수신
  - 신호 변환 (디지털 데이터 → 전송 가능한 신호 형태), bit synchronization, bit rate control 등을 담당
  - 주요 장비: 케이블, 허브 등

- **Data Link Layer**: 두 장치 간 (peer-to-peer, P2P) 신뢰성 있는 데이터 전송을 처리하는 계층
  - PDU: Frame (네트워크 내부 전송을 위한 데이터 캡슐화 단위)
  - Network layer에서 받아온 packet을 frame 단위로 만들고 header (srt MAC, dst MAC)와 trailer (데이터 에러 감지 bit) 추가
  - MAC (Media Access Control) 주소 기반으로 데이터 전달
  - Error control, flow control 등을 통해 데이터 손실을 방지
  - 주요 장비: switch, bridge
  - 주요 프로토콜: Address Resolution Protocol (ARP), ethernet, bluetooth, wi-fi 등

- **Network Layer**: end-to-end 간 데이터 전달 (packet routing)을 담당하는 계층
  - PDU: Packet
  - 서로 다른 네트워크의 host 간 통신을 가능하게 하고, 목적지까지의 최적 경로를 선택
  - Packet forwarding, routing, logical addressing(Internet Protocol, IP) 관리 수행
    - Packet forwarding: packet header의 src, dst IP 주소를 보고 packet을 목적지까지 전달
    - Routing: network 간 router를 통해 목적지까지의 최적 경로 선택

- **Transport Layer**: end-to-end 간 데이터 전송의 신뢰성을 제공하는 계층
  - PDU: Segment
  - Error check, retransmission, flow control 등을 구현
  - 주요 장비: Gateway (서로 다른 통신 프로토콜이나 네트워크 구조를 사용하는 두 네트워크를 연결)
  - 주요 프로토콜: TCP, UDP

- **Session Layer**: 통신 세션을 관리하는 계층
    - 데이터 통신을 위한 논리적 연결을 담당 
    - Application 간 연결 (e.g., TCP 세션)을 구축/관리/종료
    - 통신 장치 간 동기화를 제공
    - 연결 세션에서 data 교환과 error 발생 시의 복구 및 보안을 관리

- **Presentation Layer**: 데이터 표현 형식을 변환하는 계층
    - 압축, 암호화 등을 수행하여 application layer가 요구하는 형식으로 변환
    - Translation Layer라고도 불림

- **Application Layer**: User에게 실제 application 서비스를 제공하는 계층
    - User 및 application이 네트워크에 접속할 수 있게 해줌
    - End system이 서비스 (e-mail, 파일 전송 등)를 제공받기 위해 주고 받아야하는 메세지 형식에 대한 프로토콜을 정의
    - 주요 프로토콜: DHCP, HTTP, DNS, FTP 등


![][osi_layer_description.png]


---


# TCP/IP (Transmission Control Protocol / Internet Protocol) Model

TCP/IP 모델은 1960년대 미국 국방부에 의해 개발된 실무 중심 네트워크 모델이며, 4계층 또는 5계층 구조로 표현
1983년 공식 표준으로 채택되면서 OSI 모델보다 단순하고 실용적이기 때문에 실제 네트워크 구현 모델로 널리 사용


## Original TCP/IP Model – 4 Layers

- **Application Layer**: OSI의 Session, Presentation, Application layer를 통합한 계층
- **Transport Layer**: TCP/UDP 기반 end-to-end 데이터 전송 제공
- **Internet Layer**: IP 기반 packet routing
- **Network Access Layer**: OSI의 Physical Layer + Data Link Layer를 통합해 표현한 계층


## TCP/IP Updated Model – 5 Layers

- **Application Layer**: OSI의 Session, Presentation, Application layer를 통합한 계층
- **Transport Layer**: TCP/UDP 기반 end-to-end 데이터 전송 제공
- **Internet Layer**: IP 기반 packet routing
- **Data Link Layer**: MAC 주소 기반 frame 전송
- **Physical Layer**: 신호 기반 bit 전송


---


# TCP (Transmission Control Protocol)

TCP는 transport layer에서 사용하는 프로토콜로, 신뢰성을 보장하는 연결형 서비스
Network에 연결된 host에서 실행되는 application 간 데이터를 순서대로 안정적이고 에러 없이 교환할 수 있게 해줌.


## TCP 특징

- 연결 지향 (Connection-oriented) 방식: data를 보내기 전에 먼저 상대와 연결 설정 필요
- Stream 기반 서비스: 데이터를 연속된 바이트 흐름으로 전달하며, 메시지 경계를 구분하지 않음


![][connection_oriented.png]


- 3-way handshaking 과정을 통해 연결 설정
- 4-way handshaking 과정을 통해 연결 해제
- 흐름 제어 (flow control), 혼잡 제어 (congestion control), 그리고 재전송 (Retransmission)을 통한 높은 신뢰성 보장
    - Flow control: sliding window 방식으로 데이터 전송 속도를 조절 -> 수신자의 buffer overflow 방지
    - Sliding window: 송신자는 수신자가 알려준 window size 범위만큼 packet을 보내고 ACK마다 window를 늘려가며 전송함
    - Congestion control: 네트워크에서 돌아다니는 packt 수가 과도하게 증가하는 현상 방지
    - Retransmission: sequence와 ACK를 사용해 segment에서 손실된 데이터 범위를 식별하고 재전송 시작 위치를 결정
- UDP보다 속도가 느림
- 전이중(Full-Duplex), 점대점(Point-to-Point) 방식
    - Full-Duplex: 송신과 수신이 동시에 가능한 통신 방식
    - Point-to-Point: 하나의 TCP 연결은 한 명의 송신자와 한 명의 수신자만 존재 (1:1 연결)


![][tcp_sliding_window.png] TCP Sliding Window


## TCP Header


![][tcp_header.png]


- Source Port (16 bit)
    - 출발지 port 번호
    - 전송하는 측에서 임의의 번호를 사용 (application에 따라 port 번호가 정해져 있을 수 있음)
 
- Destination Port (16 bit)
    - 목적지 port 번호
    - Application에 따라 port 번호가 정해져 있음
 
- Sequence Number (32 bit)
    - TCP 세그먼트에 포함된 데이터의 시작 바이트 번호
    - 통신 시작 시 0이 아니라 임의의 초기 값(ISN, Initial Sequence Number)에서 시작해 보안성과 충돌을 방지
 
- Acknowledgement Number (32 bit)
    - 상대방이 보낸 segment를 잘 받았음을 알리기 위한 번호
 
- Offset (4 bit)
    - TCP header 길이를 4바이트 단위로 표시
    - TCP header는 최소 20, 최대 60 byte
 
- Reserved (4 bit)
    - 사용하지 않는 필드이며, 모두 0으로 표시
 
- Flags (8 bit)
    - 제어 비트(control bits) 라고도 부르며, segment의 종류를 표시하는 필드
 
- Window size (16 bit)
    - Client의 ACK 없이 현재 전송 가능한 최대 byte 수
 
- Checksum (16 bit)
    - Header와 data의 에러를 확인하기 위한 필드
 
- Urgent Pointer (16 bit)
    - 현재 sequence number부터 긴급 데이터의 끝 위치까지의 바이트 오프셋을 표시
    - 긴급 data를 우선 처리하도록 수신 측에 알려주는 역할
 
- Option (0~40 byte)
    - 최대 segment 사이즈 지정 등 추가적인 옵션이 있을 경우 표시


## TCP 세션 연결


![][tcp_3way.png]


- Connection을 위해 3-way handshaking 방법을 사용 -> 양쪽 모두 data 전송 준비가 되었다는 것을 보장
- Step 1) Client는 server에 접속을 요청하는 SYN packet을 전송 -> Client는 ACK을 기다리는 SYN_SENT 상태로 전환
- Step 2) Server는 요청을 정상적으로 받았다는 대답(ACK)과 client도 포트를 열어달라는 SYN 을 같이 전송 -> Server는 ACK을 기다리는 SYN_RECEIVED 상태로 전환
- Step 3) Client가 SYN+ACK 을 받으면 ESTABLISHED로 상태를 변경하고 서버에 ACK 을 전송 -> ACK을 받은 Server는 ESTABLSHED 상태가 됨


## TCP 세션 종료


![][tcp_4way.png]


- Connection을 끊기 위해 4-way handshaking 방법을 사용
- Step 1) Client가 server에게 FIN을 보냄 -> Client는 FIN_WAIT 1 상태로 대기
- Step 2) FIN을 받은 server는 상태를 CLOSE_WAIT로 변경하고 ACK을 보냄 -> ACK을 받은 client는 상태를 FIN_WAIT 2로 변경
    - 이 시점에 server는 해당 port에 연결되어 있는 application에게 CLOSE()를 요청
- Step 3) Application이 close 준비가 되면 server는 FIN을 client에게 보낸 후 상태를 LAST_ACK으로 변경
- Step 4) Client가 FIN을 받으면 이에 대한 응답으로 ACK을 보낸 후 TIME_WAIT 상태로 변경
    - ACK을 받은 server는 CLOSED로 상태를 변경하고 client도 일정 시간이 지나면 CLOSED로 상태를 변경


---


# UDP (User Datagram Protocol)

UDP는 transport layer에서 사용하는 비연결형 프로토콜로, 빠른 데이터 전송을 목표로 하지만 신뢰성을 보장하지 않음.  
Network에 연결된 host에서 실행되는 application 간 데이터를 별도의 연결 과정 없이 데이터그램 단위로 전송함.


## UDP 특징

- 비연결형 방식: 데이터를 보내기 전에 상대와 연결 설정 과정이 없음
    - 각 패킷이 독립적으로 전송되며 서로 다른 경로를 통해 전달될 수 있음
- Datagram 기반 서비스: 메시지 단위로 독립적으로 보내며, 보낸 메시지의 경계가 그대로 유지되어 전달
- 재전송, 흐름 제어, 혼잡 제어 기능 없음 → 데이터 신뢰성 보장 X
- 순서를 보장하지 않음 → 먼저 전송한 패킷이 더 늦게 도착하거나 유실될 수 있음
- Packet overhead가 적고 네트워크 부하 감소 → TCP보다 전송 속도 빠름
- 속도가 중요한 서비스에서 사용 (e.g., 실시간 스트리밍, 음성/영상 서비스 등)


![][datagram.png] Datagram


## UDP 헤더


![][udp_header.png]


- Source Port (16 bit)
    - 송신 application을 식별하기 위한 port 번호
    - 일반적으로 송신 측에서 임의의 port를 사용

- Destination Port (16 bit)
    - 수신 application을 식별하기 위한 port 번호
    - 서비스에 따라 정해진 port 번호 사용 (e.g., DNS 53, DHCP 67/68)

- Length (16 bit)
    - UDP header와 data를 합한 전체 크기를 byte 단위로 표시

- Checksum (16 bit)
    - Header와 data의 오류를 검출하기 위한 필드
    - 에러 복구 기능은 없으며, UDP header는 TCP보다 단순함

> 데이터 신뢰성 보장을 위한 Sequence Number, ACK Number 등은 존재하지 않음.


---


# TCP vs. UDP

- 연결 방식
  - TCP: 연결형(Connection-oriented), 통신 전에 세션을 설정
  - UDP: 비연결형(Connectionless), 세션 설정 없이 데이터 전송

- 신뢰성
  - TCP: 데이터 전달 신뢰성 보장, 재전송, 오류 검출, 순서 보장
  - UDP: 신뢰성 없음, 손실·순서 뒤바뀜 허용, 재전송 없음

- 데이터 전송 단위
  - TCP: 스트림(Stream) 기반, 연속된 바이트 흐름
  - UDP: 데이터그램(Datagram) 기반, 메시지 단위 독립 전송

- 속도
  - TCP: 상대적으로 느림 (흐름제어, 혼잡제어, 재전송 등)
  - UDP: 빠름, 오버헤드 적음

- 헤더 구조
  - TCP: Sequence Number, ACK Number, Window Size, Urgent Pointer 등 포함
  - UDP: Source Port, Destination Port, Length, Checksum

- 사용 사례
  - TCP: 웹, 이메일, 파일 전송 등 신뢰성이 필요한 서비스
  - UDP: 실시간 스트리밍, VoIP, DNS 조회 등 속도가 중요한 서비스



| 항목 | TCP | UDP |
|------|-----|-----|
| 연결 방식 | 연결형 (Connection-oriented) | 비연결형 (Connectionless) |
| 신뢰성 | 높음 (재전송, 순서 보장) | 낮음 (재전송 X, 순서 X) |
| 데이터 단위 | 스트림(Stream) | 데이터그램(Datagram) |
| 속도 | 느림 | 빠름 |
| 흐름 제어 / 혼잡 제어 | 있음 | 없음 |
| 헤더 필드 | Seq, ACK, Window, Urgent Pointer 등 | Src Port, Dst Port, Length, Checksum |
| 전송 방식 | 점대점(1:1), 전이중 (Full-Duplex) | 독립 메시지 단위 |
| 대표 사용 사례 | 웹, 이메일, 파일 전송 | 실시간 스트리밍, VoIP, DNS 등 |


---


# Remote Direct Memory Access (RDMA)

RDMA는 네트워크를 통해 원격 시스템의 메모리에 직접 접근하여 데이터 전송을 수행하는 기술
CPU 및 Kernel 개입 없이 메모리 간 데이터를 복사할 수 있음


---


## 주요 특징

- 초저지연(Low Latency): 커널 우회를 통한 전송
- 낮은 CPU 사용률: 데이터 복사 과정에서 CPU 개입 최소화
- 고성능 대역폭(High Throughput): 대규모 데이터 처리에 적합


![][rdma_bypass.png]


---


## 핵심 개념

| 항목 | 설명 |
|------|------|
| Zero-Copy | 데이터가 커널 버퍼를 거치지 않고 사용자 메모리 간 직접 복사 |
| Kernel Bypass | 시스템 콜 없이 NIC가 데이터 전송 처리 |
| Queue Pair (QP) | RDMA 통신을 위한 Send/Receive 큐 쌍 |


---


## RDMA 전송 모드

| Operation      | Description               |
|---------------|---------------------------|
| RDMA Write    | 원격 메모리에 직접 쓰기     |
| RDMA Read     | 원격 메모리에서 직접 읽기   |
| Send/Receive  | 일반적인 메시지 기반 통신   |


---


## 사용 예시

- HPC(High Performance Computing)
- 분산 스토리지 / 분산 데이터베이스
- AI/ML 클러스터 통신


---


## 장단점

| 장점 | 단점 |
|------|------|
| 초고성능, 초저지연 | 인프라 구축 비용 높음 |
| CPU 부하 감소 | 구현 복잡도 높음 |
| 확장성 우수 | RDMA 지원 NIC / 스위치 필요 |


---


[osi_layer_overview.png]: ./week04_images/osi_layer_overview.png
[osi_layer_description.png]: ./week04_images/osi_layer_description.png
[connection_oriented.png]: ./week04_images/connection_oriented.png
[datagram.png]: ./week04_images/datagram.png
[tcp_header.png]: ./week04_images/tcp_header.png
[tcp_3way.png]: ./week04_images/tcp_3way.png
[tcp_4way.png]: ./week04_images/tcp_4way.png
[tcp_sliding_window.png]: ./week04_images/tcp_sliding_window.png
[udp_header.png]: ./week04_images/udp_header.png
[rdma_bypass.png]: ./week04_images/rdma_bypass.png
