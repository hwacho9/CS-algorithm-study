# 5주차: HTTP/HTTPS, REST API, CDN·Caching·Reverse Proxy
---

## 1. 전체 요청 흐름 (Big Picture)

### 표준 트래픽 경로
```
[DNS Resolution Phase]
User
 ↓
Local DNS Resolver (ISP / 회사 / OS)
 ↓
Root DNS Server (.)
 ↓
TLD DNS Server (.com)
 ↓
Route 53 Authoritative DNS
 ↓
CloudFront IP 반환

[HTTP/HTTPS Request Phase]
User
 ↓
CloudFront (CDN)
 ↓
AWS WAF(Web Application Firewall)
 ↓
ALB                           | NLB (Ingress용)
 ↓                            | ↓
ECS Service (Target Group)    | Kubernetes Ingress Controller
 ↓                            | ↓
Task ((Container: App)        | Service (ClusterIP)
 ↓                            | ↓
DB or Cache                   | DB or Cache
```

![image](./Screenshot.png)

### 계층별 책임
| 계층 | 주요 역할 |
|---|---|
| Client | 캐시 재검증, 쿠키/토큰 보관 |
| CDN | 엣지 캐시, TLS, HTTP/2·3, Bot 차단 |
| WAF | OWASP Top10, Rate Limit | <- attach - Bot protection
| Reverse Proxy | 라우팅, 인증 위임, retry/timeout |
| Application | 비즈니스 로직 |
| DB/Cache | 영속성 / 고속 처리 |

---

## 2. HTTP 핵심
#### 정의
**HTTP(HyperText Transfer Protocol)**는
클라이언트와 서버가 **요청(Request) / 응답(Response)**을 주고받는 애플리케이션 계층 프로토콜

#### 특징
- ❌ 암호화 없음 (평문)
- ❌ 도청/위조 가능
- TCP 위에서 동작 (보통 80번 포트)

#### 상태 코드
| 코드 | 의미 |
|---|---|
| 200 | 정상 처리 |
| 201 | 리소스 생성 (`Location` 헤더 권장) |
| 204 | 응답 바디 없음 |
| 304 | 캐시 재사용 |
| 400 | 입력 검증 실패 |
| 401 | 인증 실패 |
| 403 | 권한 없음 |
| 404 | 리소스 없음 |
| 409 | 중복/충돌 |
| 429 | Rate limit |
| 5xx | 서버/업스트림 장애 |

---

## 3. HTTPS / TLS

#### 정의
HTTPS = HTTP + TLS(암호화)

#### handshake
https://dkswhdgur246.tistory.com/54

ex)
```shell
curl --header "Authorization: Bearer $TOKEN" -v https://bookinfo.local/productpage -L -k

# === TCP 연결 단계 ===
# *   Trying 127.0.0.1:443...
# * Connected to bookinfo.local (127.0.0.1) port 443 (#0)

# === ALPN 협상 (Application Layer Protocol Negotiation) ===
# * ALPN, offering h2                    # HTTP/2 지원 제안
# * ALPN, offering http/1.1              # HTTP/1.1 지원 제안

# === TLS 핸드셰이크 시작 ===
# * TLSv1.0 (OUT), TLS header, Certificate Status (22):
# * TLSv1.3 (OUT), TLS handshake, Client hello (1):    # 1단계: 클라이언트가 지원 암호화 방식 전송

# === 서버 응답 ===
# * TLSv1.2 (IN), TLS header, Certificate Status (22):
# * TLSv1.3 (IN), TLS handshake, Server hello (2):     # 2단계: 서버가 선택한 암호화 방식 응답
# * TLSv1.2 (IN), TLS header, Finished (20):
# * TLSv1.2 (IN), TLS header, Supplemental data (23):
# * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
# * TLSv1.3 (IN), TLS handshake, Certificate (11):     # 3단계: 서버 인증서 전송
# * TLSv1.3 (IN), TLS handshake, CERT verify (15):     # 서버 인증서 검증
# * TLSv1.3 (IN), TLS handshake, Finished (20):        # 서버 핸드셰이크 완료

# === 클라이언트 핸드셰이크 완료 ===
# * TLSv1.2 (OUT), TLS header, Finished (20):
# * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
# * TLSv1.2 (OUT), TLS header, Supplemental data (23):
# * TLSv1.3 (OUT), TLS handshake, Finished (20):       # 4단계: 클라이언트 핸드셰이크 완료

# === SSL 연결 확립 ===
# * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384  # 최종 선택된 암호화 방식
# * ALPN, server accepted to use h2                         # HTTP/2 프로토콜 선택됨

# === 인증서 정보 ===
# * Server certificate:
# *  subject: CN=*.local                 # 인증서 주체
# *  start date: Aug 26 17:09:23 2025 GMT
# *  expire date: Aug 26 17:09:23 2026 GMT
# *  issuer: CN=*.local                  # 발급자 (자체 서명)
# *  SSL certificate verify result: self-signed certificate (18), continuing anyway.  # -k 옵션으로 무시

# === HTTP/2 연결 확립 ===
# * Using HTTP2, server supports multiplexing
# * Connection state changed (HTTP/2 confirmed)
# * Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0

# === HTTP 요청 전송 ===
# * Using Stream ID: 1 (easy handle 0x55a1f0d189f0)
# > GET /productpage HTTP/2             # HTTP/2 프로토콜로 GET 요청
# > Host: bookinfo.local
# > user-agent: curl/7.81.0
# > accept: */*
# > authorization: Bearer eyJhbGciOiJSUzI1NiIs...  # JWT 토큰

# === 세션 관리 ===
# * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):    # 세션 재사용을 위한 티켓
# * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
# * old SSL session ID is stale, removing
# * Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!

# === HTTP 응답 ===
# < HTTP/2 200                          # 성공 응답
```
---
## REST API 리소스 중심 URI = 엔티티 관계 표현

### 핵심 개념
REST API의 리소스 중심 URI는 데이터베이스 엔티티 간의 관계를 HTTP 경로로 표현한 것

### 강엔티티 vs 약엔티티 관점

강엔티티 (Strong Entity)
- 독립적 존재, 자체 기본키 보유
- URI: /users, /products, /categories

약엔티티 (Weak Entity)
- 강엔티티에 의존적 존재
- URI: /users/{id}/orders, /orders/{id}/items

### 실제 예시
GET /users/123              # 강엔티티: 사용자
GET /users/123/orders       # 약엔티티: 사용자의 주문들
GET /orders/456/items       # 약엔티티: 주문의 상품들


### 관계 표현 방식
- **소유 관계**: /users/123/orders (사용자가 주문을 소유)
- **포함 관계**: /orders/456/items (주문이 상품들을 포함)
- **분류 관계**: /categories/789/products (카테고리가 상품들을 분류)

### 결론
REST URI의 계층 구조 = 데이터베이스 엔티티의 의존성 관계를 웹 API로 표현한 것

예외)
- 복잡한 비즈니스 로직은 /search, /calculate 같은 동사형 엔드포인트 사용
- 성능상 이유로 중첩을 피하고 플랫 구조 선택하기도 함

## 엔티티 관계 기반 URI 설계의 장점

### 1. 직관적 이해
```
/users/123/orders     # "123번 사용자의 주문들"
/orders/456/items     # "456번 주문의 상품들"
```

- URL만 봐도 데이터 관계를 바로 이해 가능
- 개발자 간 소통 비용 감소

### 2. 일관성 있는 설계
- 데이터베이스 스키마와 API 구조가 일치
- 백엔드-프론트엔드 개발자 모두 같은 멘탈 모델 공유
- 새로운 엔드포인트 추가 시 예측 가능한 패턴

### 3. 권한 관리 용이
/users/123/orders     # 123번 사용자만 접근 가능
/orders/456/items     # 456번 주문 소유자만 접근 가능

- 계층 구조로 자연스러운 권한 체크
- 보안 로직이 단순해짐

### 4. 캐싱 효율성
- 리소스 간 의존성이 명확해 캐시 무효화 전략 수립 용이
- CDN에서도 패턴 기반 캐싱 규칙 적용 가능

### 5. 확장성
- 새로운 관계 추가 시 기존 패턴 재사용
- API 버전 관리가 체계적

결론: 비즈니스 도메인의 자연스러운 구조를 API에 반영해서 이해하기 쉽고 유지보수하기 좋은 API가 됩니다.

### 다른 방법틀

#### GraphQL
개념
클라이언트가 필요한 데이터 구조를 직접 정의해서 요청
```
단일 요청으로 복합 데이터 조회(Semi-Structured Data:반구조화 데이터):
query {
  user(id: 123) {
    name
    email
    orders {
      id
      date
      items {
        name
        price
        quantity
      }
    }
    profile {
      avatar
      bio
    }
  }
}
```

단일 Endpoint (/graphql)


언제 쓰나

- 프론트엔드(BFF) 중심 서비스
- 화면별로 필요한 필드가 크게 다른 경우
- 모바일/웹 공통 API

장점

- Over-fetch / Under-fetch 해결
- 프론트 주도 개발에 유리
- Schema 기반 (타입 안정성)

단점

- 캐싱 어려움 (CDN 캐시 거의 불가)
- 복잡한 쿼리로 서버 부하 위험
- Rate limit / 권한 설계 난이도 높음
- GET(x)
- 로그 관리

link
- https://graphql.org/
- https://hasura.io/

#### gRPC(Google Remote Procedure Call))

### 핵심 개념
Google이 개발한 고성능 RPC 프레임워크로, HTTP/2 + Protocol Buffers 기반의 바이너리 통신 프로토콜

### 주요 구성 요소

1. Protocol Buffers (직렬화)
protobuf
service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
}

- 바이너리 데이터 포맷 (JSON 대비 3-10배 작음)
- 타입 안전성, 하위 호환성

2. HTTP/2 (전송 계층)
- 멀티플렉싱: 하나의 연결로 동시 요청 처리
- 바이너리 프로토콜: 빠른 파싱
- 스트리밍: 실시간 양방향 통신

3. 4가지 통신 방식
- Unary: 일반 요청-응답
- Server/Client/Bidirectional Streaming

### REST vs gRPC
| 항목 | REST | gRPC |
|------|------|------|
| 데이터 | JSON (텍스트) | Protobuf (바이너리) |
| 프로토콜 | HTTP/1.1 | HTTP/2 |
| 성능 | 느림 | 빠름 |
| 브라우저 | 완전 지원 | 제한적 |
| 사용처 | 외부 API | 내부 서비스 간 통신 |

### 장단점
장점: 고성능, 스트리밍, 타입 안전성
단점: 브라우저 제한, 가독성 낮음, 디버깅 어려움

결론: 마이크로서비스 간 고성능 내부 통신에 최적화된 프레임워크

### OSI 7계층의 소프트웨어/하드웨어 구분

```
┌─────────────────────────┐
│ 7. Application Layer    │ ← 사용자 애플리케이션 (소프트웨어)
├─────────────────────────┤
│ 6. Presentation Layer   │ ← 사용자 애플리케이션 (소프트웨어)
├─────────────────────────┤
│ 5. Session Layer        │ ← 사용자 애플리케이션 (소프트웨어)
├─────────────────────────┤ ---------------------------------------------(User Mode / Kernel Mode) system call을 이용한 통신
│ 4. Transport Layer      │ ← 운영체제 커널 (소프트웨어)
├─────────────────────────┤
│ 3. Network Layer        │ ← 운영체제 커널 (소프트웨어)
├─────────────────────────┤
│ 2. Data Link Layer      │ ← 네트워크 드라이버 + NIC (소프트웨어+하드웨어)
├─────────────────────────┤
│ 1. Physical Layer       │ ← 네트워크 카드, 케이블 (하드웨어)
└─────────────────────────┘

```
---

## 5. Reverse Proxy
#### 정의
서버 앞에서 클라이언트 요청을 대신 받아 내부 서버로 전달하는 중계 서버

#### 흐름
Client → Reverse Proxy → Backend Server

#### 주요 역할
- 로드 밸런싱
- TLS(HTTPS) 종료
- 보안 (인증·인가·Rate Limit)
- 캐싱 및 성능 개선

#### 특징
- 클라이언트는 실제 서버를 직접 알 필요 없음
- 서버를 외부로부터 보호

#### ex)
- ALB/NLB
- Broadcast
  - 여러 Application에서 metrics를 pushgateway에 송신한다.
  - pushgateway는 메모리에 메트릭스를 저장한다.
  - prometheus server는 모든pushgateway에서 메모리상의 metrics를 pull한다.
  - 문제!) 10:02에서 app의 실제 cpu사용률은???
    - pushgateway pod(container)冗長化일때 문제점

![image](./Screenshot2.png)
```
- (10:00)app -> svc -> pod(172.1.2.1) => cpu_usage 1000(1core)
                    -> pod(172.1.2.2)
                    -> pod(172.1.2.3)
- (10:01)app -> svc -> pod(172.1.2.1) cpu_usage 1000(memory)
                    -> pod(172.1.2.2) => cpu_usage 1500
                    -> pod(172.1.2.3)
- (10:02)app -> svc -> pod(172.1.2.1) cpu_usage 1000(memory)
                    -> pod(172.1.2.2) cpu_usage 1500(memory)
                    -> pod(172.1.2.3) => cpu_usage 1200

- app -> svc -> reverse proxy -> pod(172.1.2.1) cpu_usage 1000
                              -> pod(172.1.2.2) cpu_usage 1000
                              -> pod(172.1.2.3) cpu_usage 1000
```


link
- https://inpa.tistory.com/entry/NETWORK-%F0%9F%93%A1-Reverse-Proxy-Forward-Proxy-%EC%A0%95%EC%9D%98-%EC%B0%A8%EC%9D%B4-%EC%A0%95%EB%A6%AC
---

## 6. CDN

### 1. CDN이란?
CDN(Content Delivery Network)은
**사용자와 서버(Origin) 사이에 위치한 전 세계 분산된 Edge 서버 네트워크**로,
콘텐츠를 **더 빠르고 안정적으로 전달**하기 위한 인프라이다.


### 2. CDN의 핵심 목적
- 지연 시간(Latency) 감소
- 서버(Origin) 부하 감소
- 트래픽 비용 절감
- 가용성 및 안정성 향상


### 3. CDN의 기본 구조

User
↓
CDN Edge (가장 가까운 위치)
↓ (Cache Miss 시)
Origin Server (ALB / S3 / App)

- Cache Hit → Edge에서 즉시 응답
- Cache Miss → Origin에서 가져와 Edge에 저장 후 응답

### 4. CDN이 제공하는 주요 기능

#### 4.1 엣지 캐싱 (Edge Caching)
- 정적 파일 (JS, CSS, Image)
- 조건부 동적 콘텐츠 (API 응답 일부)


#### Shielding
전 세계 Edge 요청을 “하나의 중앙 캐시(Shield)”로 먼저 모아서 오리진 보호(Shield) 하는 구조

Edge Server는 지리적으로 가까운 지역의 사용자만 이용할 수 있다.

ex)
예를 들어서 us request가 폭증했을때 us edge가 miss인경우 그대로 origin에 리퀘스트가 가서 origin이 죽을 수 있다.
```
US User
 ↓
US Edge (miss)
 ↓
JP Edge (skip) (이용못함)
 ↓
Origin

#해결 방법
US User
 ↓
US Edge (miss)
 ↓
JP Edge (Origin Shielding설정) (이용못함) => edge 아니더라도 공용 캐시도 상관없음[Reverse Proxy Cache(Varnish Cache)]
 ↓
Origin
```
![image](./Screenshot3.png)
