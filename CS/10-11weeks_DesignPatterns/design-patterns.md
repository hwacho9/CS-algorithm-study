# 디자인 패턴 & 아키텍처 패턴 정리

---

## 목차

1. [옵저버 패턴 (Observer Pattern)](#1-옵저버-패턴-observer-pattern)
2. [프록시 패턴 (Proxy Pattern)](#2-프록시-패턴-proxy-pattern)
3. [MVC 패턴 (Model-View-Controller)](#3-mvc-패턴-model-view-controller)
4. [MVVM 패턴 (Model-View-ViewModel)](#4-mvvm-패턴-model-view-viewmodel)
5. [클린 아키텍처 (Clean Architecture)](#5-클린-아키텍처-clean-architecture)
6. [패턴 비교 요약](#6-패턴-비교-요약)

---

## 1. 옵저버 패턴 (Observer Pattern)

### 등장 배경

GUI 프로그래밍이 발전하던 1980~90년대, **한 객체의 상태 변화를 다른 여러 객체에게 알려야 하는 상황**이 반복적으로 등장했다.

**해결 전 — 폴링(Polling) 방식의 문제:**
```
// 변화가 생겼는지 직접 반복 확인 → 낭비
while (true) {
  if (stock.getPrice() !== lastPrice) {
    chart.update();
    alertUI.update();
    logger.update();
  }
}
```
- 변화가 없어도 계속 확인해야 하므로 **CPU 낭비**
- 알림을 받아야 할 객체가 늘어날수록 **발행자 코드가 수신자를 직접 알아야 함**
- 수신자가 추가/제거될 때마다 **발행자 코드를 수정**해야 함

**핵심 문제:** 발행자와 수신자가 강하게 결합되어 있어, 수신자가 바뀌면 발행자도 바뀌어야 한다.

→ 이 문제를 해결하기 위해 "수신자가 스스로 등록하고, 변화가 생기면 자동으로 통보받는" 구조가 정립되었다.

### 개요

**옵저버 패턴**은 객체의 상태 변화가 있을 때, 그 객체에 의존하는 다른 객체들에게 자동으로 알림을 보내는 **행위(Behavioral) 패턴**이다.

> "구독자가 발행자의 상태 변화를 자동으로 통보받는다."

### 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **Subject (발행자)** | 상태를 갖고, 옵저버를 등록/해제/알림하는 객체 |
| **Observer (구독자)** | Subject의 변화를 통보받아 반응하는 객체 |
| **ConcreteSubject** | Subject의 실제 구현체 |
| **ConcreteObserver** | Observer의 실제 구현체 |

### 동작 흐름

```
Subject 상태 변경
    → notify() 호출
        → Observer1.update()
        → Observer2.update()
        → Observer3.update()
```

### 예시 코드 (TypeScript)

```typescript
// Observer 인터페이스
interface Observer {
  update(data: any): void;
}

// Subject 클래스
class EventEmitter {
  private observers: Observer[] = [];

  subscribe(observer: Observer) {
    this.observers.push(observer);
  }

  unsubscribe(observer: Observer) {
    this.observers = this.observers.filter(o => o !== observer);
  }

  notify(data: any) {
    this.observers.forEach(o => o.update(data));
  }
}

// ConcreteObserver
class Logger implements Observer {
  update(data: any) {
    console.log('[LOG]', data);
  }
}

// 사용
const emitter = new EventEmitter();
const logger = new Logger();

emitter.subscribe(logger);
emitter.notify({ event: 'login', user: 'Alice' });
// [LOG] { event: 'login', user: 'Alice' }
```

### 장단점

**장점**
- 느슨한 결합(Loose Coupling): Subject와 Observer가 서로 독립적으로 변경 가능
- 브로드캐스트 방식의 통신 지원
- 새로운 Observer 추가가 쉬움

**단점**
- 옵저버가 많아지면 알림 순서 예측이 어려움
- 순환 참조 시 무한 루프 위험
- 메모리 누수 주의 (구독 해제 필수)

### 활용 사례
- JavaScript `EventEmitter`, DOM `addEventListener`
- Vue.js의 반응형 데이터 시스템 (Reactivity)
- RxJS의 Observable/Subject
- 뉴스레터 구독 시스템

---

## 2. 프록시 패턴 (Proxy Pattern)

### 등장 배경

소프트웨어가 복잡해지면서 세 가지 반복적인 문제가 나타났다.

**문제 1 — 비용이 큰 객체를 미리 생성하는 낭비:**
```
// 앱 시작 시 수백 MB 이미지를 모두 메모리에 올림
const images = allImagePaths.map(p => new HeavyImage(p)); // 실제로 안 보는 것도 포함
```

**문제 2 — 핵심 로직에 부가 기능(로깅, 인증)이 뒤섞임:**
```
class UserService {
  getUser(id) {
    // 인증 코드 ← 비즈니스 로직과 무관
    if (!isAuthenticated()) throw new Error('Unauthorized');
    // 로깅 코드 ← 비즈니스 로직과 무관
    console.log(`getUser called: ${id}`);
    // 실제 비즈니스 로직
    return db.find(id);
  }
}
```

**문제 3 — 동일 연산을 반복 수행하는 성능 낭비:**
```
// 같은 데이터를 요청할 때마다 DB를 다시 조회
const user = await db.findById(1); // 매번 DB 히트
```

**핵심 문제:** 실제 객체를 직접 사용하면 생성 비용·부가 관심사·반복 연산을 제어할 수 없다.

→ "실제 객체와 동일한 인터페이스를 가진 대리 객체를 앞에 두어, 접근을 가로채고 제어한다"는 구조가 정립되었다.

### 개요

**프록시 패턴**은 어떤 객체에 대한 접근을 제어하기 위해 **대리인(Proxy) 객체**를 앞에 두는 **구조(Structural) 패턴**이다.

> "실제 객체 대신 프록시가 먼저 요청을 받아 처리하거나 전달한다."

### 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **Subject (인터페이스)** | RealSubject와 Proxy가 공유하는 인터페이스 |
| **RealSubject** | 실제 비즈니스 로직을 수행하는 객체 |
| **Proxy** | RealSubject에 대한 접근을 제어하는 대리 객체 |

### 프록시의 종류

| 종류 | 설명 |
|---|---|
| **가상 프록시 (Virtual Proxy)** | 비용이 큰 객체의 생성을 지연 (Lazy Loading) |
| **보호 프록시 (Protection Proxy)** | 접근 권한을 제어 (인증/인가) |
| **원격 프록시 (Remote Proxy)** | 원격 객체에 대한 로컬 인터페이스 제공 |
| **캐싱 프록시 (Caching Proxy)** | 요청 결과를 캐시하여 반복 연산 방지 |

### 동작 흐름

```
Client → Proxy.request()
            → (전처리: 권한 확인, 캐시 조회 등)
            → RealSubject.request()
            → (후처리: 로깅, 캐시 저장 등)
         ← 결과 반환
```

### 예시 코드 (TypeScript)

```typescript
// Subject 인터페이스
interface DataService {
  getData(id: number): string;
}

// RealSubject
class RealDataService implements DataService {
  getData(id: number): string {
    console.log(`DB에서 데이터 조회: ${id}`);
    return `Data(${id})`;
  }
}

// Caching Proxy
class CachedDataService implements DataService {
  private cache: Map<number, string> = new Map();
  private real = new RealDataService();

  getData(id: number): string {
    if (this.cache.has(id)) {
      console.log(`캐시에서 반환: ${id}`);
      return this.cache.get(id)!;
    }
    const result = this.real.getData(id);
    this.cache.set(id, result);
    return result;
  }
}

// 사용
const service: DataService = new CachedDataService();
service.getData(1); // DB에서 데이터 조회: 1
service.getData(1); // 캐시에서 반환: 1
```

### 장단점

**장점**
- 실제 객체를 수정하지 않고 추가 기능(캐싱, 로깅, 접근제어) 부여
- 단일 책임 원칙(SRP) 유지
- 지연 초기화로 성능 최적화 가능

**단점**
- 프록시 계층이 늘어날수록 복잡도 증가
- 응답 지연 발생 가능 (간접 호출)

### 활용 사례
- JavaScript `Proxy` 객체 (ES6+)
- Vue 3의 반응형 시스템 (`reactive()`)
- Spring AOP (트랜잭션, 로깅 프록시)
- Nginx / API Gateway (리버스 프록시)

---

## 3. MVC 패턴 (Model-View-Controller)

### 등장 배경

1970년대 Smalltalk 개발 과정에서, 초기 GUI 애플리케이션은 **데이터·화면·로직이 한 곳에 뒤섞인 구조**로 만들어졌다.

**해결 전 — 스파게티 코드:**
```
// 데이터 조회, 화면 표시, 입력 처리가 한 함수에 혼재
function onButtonClick() {
  const data = db.query('SELECT * FROM users'); // 데이터
  document.getElementById('list').innerHTML = data.map(...).join(''); // 화면
  if (data.length > 100) sendAlert(); // 비즈니스 로직
}
```

**발생한 문제들:**
- 화면 디자인이 바뀌면 데이터 로직까지 건드려야 함
- DB 구조가 바뀌면 UI 코드도 함께 수정
- 한 사람이 화면을 고치다가 데이터 처리 버그를 유발
- **테스트 불가**: 화면 없이 비즈니스 로직만 테스트할 수 없음

**핵심 문제:** 역할의 경계가 없어서 한 곳의 변경이 전체에 영향을 미친다.

→ Trygve Reenskaug(1979)가 Smalltalk에서 "데이터(Model), 화면(View), 입력 처리(Controller)를 분리"하는 MVC를 고안했다.

### 개요

**MVC**는 애플리케이션을 세 가지 역할로 분리하여 **관심사의 분리(Separation of Concerns)**를 달성하는 아키텍처 패턴이다.

### 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **Model** | 데이터 및 비즈니스 로직. DB 접근, 상태 관리 |
| **View** | 사용자에게 보여지는 UI. Model의 데이터를 표시 |
| **Controller** | 사용자 입력을 받아 Model 업데이트 후 View에 반영 |

### 동작 흐름

```
사용자 입력
    → Controller
        → Model 업데이트
            → View 갱신 (Model → View 직접 or Controller 경유)
                → 사용자에게 화면 표시
```

### 예시 구조 (웹 백엔드 기준)

```
사용자: GET /users/1
    → UserController.show(id=1)
        → UserModel.findById(1)  ← DB 조회
        → user_show.html 렌더링  ← View
    → HTML 응답 반환
```

### 예시 코드 (간단한 구현)

```typescript
// Model
class UserModel {
  private users = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ];

  findById(id: number) {
    return this.users.find(u => u.id === id);
  }
}

// View
class UserView {
  render(user: { id: number; name: string } | undefined) {
    if (!user) return console.log('사용자를 찾을 수 없습니다.');
    console.log(`[화면] ID: ${user.id}, 이름: ${user.name}`);
  }
}

// Controller
class UserController {
  constructor(private model: UserModel, private view: UserView) {}

  showUser(id: number) {
    const user = this.model.findById(id);
    this.view.render(user);
  }
}

// 사용
const controller = new UserController(new UserModel(), new UserView());
controller.showUser(1); // [화면] ID: 1, 이름: Alice
```

### 장단점

**장점**
- 역할이 명확하여 유지보수 용이
- Model과 View의 독립적 개발/테스트 가능
- 코드 재사용성 향상

**단점**
- Controller가 비대해지는 **Massive Controller** 문제 발생 가능
- View와 Model 간 의존성이 생길 수 있음
- 소규모 프로젝트에는 과도한 구조일 수 있음

### 활용 사례
- Spring MVC (Java 백엔드)
- Ruby on Rails
- Django (Python)
- ASP.NET MVC

---

## 4. MVVM 패턴 (Model-View-ViewModel)

### 등장 배경

MVC가 서버 사이드에서는 잘 동작했지만, **복잡한 클라이언트 사이드 UI**에서는 새로운 문제가 드러났다.

**MVC의 한계 — Controller가 UI 동기화를 직접 관리:**
```
// Controller가 UI 상태를 일일이 수동으로 업데이트
class UserController {
  onDataLoaded(user) {
    view.setName(user.name);       // 수동 동기화
    view.setEmail(user.email);     // 수동 동기화
    view.hideLoadingSpinner();     // 수동 동기화
    view.showContent();            // 수동 동기화
    view.updateAvatarUrl(user.avatar); // 수동 동기화 ...
  }
}
```

**발생한 문제들:**
- UI 요소가 많아질수록 Controller가 **수십~수백 줄의 수동 동기화 코드**로 비대해짐 (Massive Controller)
- Controller가 View의 세부 구현(DOM, UI 컴포넌트)을 알아야 하므로 **View와 강하게 결합**
- View 없이는 Controller를 테스트할 수 없음
- 같은 데이터를 여러 화면에 표시할 때 동기화 누락 버그 빈발

**핵심 문제:** Controller가 View를 직접 조작하므로, 상태가 많아질수록 동기화 코드가 폭발적으로 증가한다.

→ 2005년 Microsoft의 Ken Cooper와 Ted Peters가 WPF를 위해 MVVM을 고안했다. "ViewModel이 상태를 갖고 있으면, View가 알아서 바인딩해서 표시한다"는 개념이다.

### 개요

**MVVM**은 MVC에서 발전한 패턴으로, **데이터 바인딩(Data Binding)**을 핵심으로 하여 View와 비즈니스 로직을 완전히 분리하는 아키텍처 패턴이다.

> "ViewModel이 View와 Model 사이의 데이터 변환 및 상태 관리를 담당한다."

### 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **Model** | 데이터와 비즈니스 로직. 서버 API, DB 접근 |
| **View** | UI 레이아웃. ViewModel의 데이터를 바인딩하여 표시 |
| **ViewModel** | View를 위한 상태(State)와 커맨드(Command) 제공. View를 알지 못함 |

### 동작 흐름

```
Model (데이터) ←→ ViewModel (상태/명령) ←[양방향 바인딩]→ View (UI)
                        ↑
                  사용자 입력 처리
```

- View → ViewModel: 사용자 액션 전달 (Command 패턴)
- ViewModel → View: 상태 변화 자동 반영 (Data Binding / 반응형)

### 예시 코드 (Vue 3 Composition API 스타일)

```typescript
// Model (API 레이어)
async function fetchUser(id: number) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}

// ViewModel (Composable)
import { ref, onMounted } from 'vue';

function useUserViewModel(id: number) {
  const user = ref<{ name: string } | null>(null);
  const isLoading = ref(false);
  const error = ref<string | null>(null);

  async function loadUser() {
    isLoading.value = true;
    try {
      user.value = await fetchUser(id);
    } catch (e) {
      error.value = '사용자 로딩 실패';
    } finally {
      isLoading.value = false;
    }
  }

  onMounted(loadUser);
  return { user, isLoading, error };
}

// View (SFC Template)
/*
<template>
  <div v-if="isLoading">로딩 중...</div>
  <div v-else-if="error">{{ error }}</div>
  <div v-else>{{ user?.name }}</div>
</template>

<script setup>
const { user, isLoading, error } = useUserViewModel(1);
</script>
*/
```

### MVC와의 차이점

| 항목 | MVC | MVVM |
|---|---|---|
| **View ↔ Logic 연결** | Controller가 직접 제어 | 데이터 바인딩으로 자동 동기화 |
| **View의 역할** | 수동적 (Controller에 의존) | 능동적 (ViewModel을 구독) |
| **테스트 용이성** | View 없이 테스트 어려움 | ViewModel 단독 테스트 용이 |
| **주요 사용처** | 서버 사이드 렌더링 | 클라이언트 사이드 SPA |

### 장단점

**장점**
- View와 비즈니스 로직의 완전한 분리
- ViewModel 단위 테스트가 쉬움 (View 없이 테스트 가능)
- 양방향 데이터 바인딩으로 UI 동기화 자동화

**단점**
- 데이터 바인딩 디버깅이 어려울 수 있음
- 간단한 UI에는 오버엔지니어링
- ViewModel이 비대해질 수 있음

### 활용 사례
- Vue.js (Reactivity System)
- React + Hooks (useXxx composable 패턴)
- Android Jetpack ViewModel
- WPF (Windows Presentation Foundation)
- SwiftUI

---

## 5. 클린 아키텍처 (Clean Architecture)

### 등장 배경

2000년대 웹 프레임워크(Rails, Django, Spring)가 폭발적으로 보급되면서, 많은 프로젝트가 **프레임워크 중심으로 설계**되기 시작했다. 그 결과 공통적인 고통이 반복됐다.

**전형적인 문제 — 프레임워크에 종속된 비즈니스 로직:**
```
// Spring Entity가 곧 비즈니스 객체 — DB 구조가 도메인을 지배
@Entity
@Table(name = "users")
class User {
  @Id @Column("user_id") val id: Long,
  @Column("user_nm") val name: String,
  // DB 컬럼명, 어노테이션이 핵심 로직에 침투
}

// Service가 HTTP Request 객체를 직접 받음 — Web 프레임워크에 종속
fun createUser(request: HttpServletRequest): User {
  val name = request.getParameter("name") // Spring/Servlet에 묶임
  ...
}
```

**발생한 문제들:**
- **DB를 MySQL → PostgreSQL로 바꾸면** ORM 어노테이션이 퍼진 도메인 객체까지 수정
- **웹 프레임워크를 교체하면** 비즈니스 로직도 함께 재작성
- **비즈니스 로직 단위 테스트** 시 DB·웹 서버를 함께 띄워야 함 → 테스트가 느리고 무거움
- 오래된 프로젝트일수록 **어디가 핵심 로직인지 구분이 안 됨**
- 프레임워크 버전 업그레이드 시 비즈니스 코드도 줄줄이 수정

**핵심 문제:** "세부 구현(DB, 프레임워크)"이 "핵심 가치(비즈니스 규칙)"를 오염시킨다. 바뀌어야 할 것(기술)이 바뀌지 말아야 할 것(규칙)을 지배한다.

→ Robert C. Martin(Uncle Bob)이 2012년 블로그에서 클린 아키텍처를 발표했다. Hexagonal Architecture, Onion Architecture 등 유사한 선행 아이디어들을 통합하여, "의존성은 반드시 안쪽(비즈니스)을 향해야 한다"는 원칙을 체계화했다.

### 개요
https://qiita.com/nrslib/items/a5f902c4defc83bd46b8

**클린 아키텍처**는 Robert C. Martin(Uncle Bob)이 제안한 아키텍처 원칙으로, **의존성 방향을 안쪽으로만 향하게** 하여 비즈니스 로직이 프레임워크·DB·UI 등 외부 요소에 의존하지 않도록 설계하는 아키텍처 패턴이다.

> "세부 구현(DB, 프레임워크, UI)은 언제든 교체 가능해야 한다. 핵심 비즈니스 규칙은 변하지 않는다."

### 계층 구조 (동심원)

```
┌─────────────────────────────────┐
│         Frameworks & Drivers    │  ← 가장 바깥 (Web, DB, UI)
│  ┌───────────────────────────┐  │
│  │    Interface Adapters     │  │  ← Controller, Presenter, Gateway
│  │  ┌─────────────────────┐  │  │
│  │  │   Application       │  │  │  ← Use Cases (응용 비즈니스 규칙)
│  │  │  ┌───────────────┐  │  │  │
│  │  │  │   Entities    │  │  │  │  ← 핵심 비즈니스 규칙 (가장 안쪽)
│  │  │  └───────────────┘  │  │  │
│  │  └─────────────────────┘  │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘

의존성 방향: 바깥 → 안쪽 (단방향)
```

### 계층별 역할

| 계층 | 이름 | 역할 | 예시 |
|---|---|---|---|
| **1층 (최내부)** | Entities | 핵심 비즈니스 규칙, 도메인 객체 | `User`, `Order` 클래스 |
| **2층** | Use Cases | 애플리케이션 고유 비즈니스 로직 | `CreateUserUseCase`, `PlaceOrderUseCase` |
| **3층** | Interface Adapters | 외부와 내부를 연결하는 변환 계층 | Controller, Repository 구현체, Presenter |
| **4층 (최외부)** | Frameworks & Drivers | 프레임워크, DB, UI, 외부 서비스 | Express, TypeORM, React |

### 핵심 원칙: 의존성 규칙 (Dependency Rule)

- **안쪽 계층은 바깥쪽 계층을 절대 알지 못한다.**
- 바깥 계층이 안쪽 계층에 의존한다.
- 경계를 넘을 때는 **인터페이스(추상화)**를 사용한다.

```
❌ 잘못된 의존: Entity → DB (핵심 로직이 DB에 의존)
✅ 올바른 의존: DB → Repository Interface ← Use Case
```

### 예시 코드 (TypeScript)

```typescript
// ── 1층: Entity ──────────────────────────────
class User {
  constructor(
    public readonly id: number,
    public name: string,
    public email: string,
  ) {}

  changeName(name: string) {
    if (!name) throw new Error('이름은 비어있을 수 없습니다.');
    this.name = name;
  }
}

// ── 2층: Use Case ─────────────────────────────
// Repository 인터페이스 (Use Case가 정의, 외부에서 구현)
interface UserRepository {
  findById(id: number): Promise<User | null>;
  save(user: User): Promise<void>;
}

class UpdateUserNameUseCase {
  constructor(private userRepo: UserRepository) {}

  async execute(id: number, newName: string): Promise<void> {
    const user = await this.userRepo.findById(id);
    if (!user) throw new Error('사용자를 찾을 수 없습니다.');
    user.changeName(newName);
    await this.userRepo.save(user);
  }
}

// ── 3층: Interface Adapter ────────────────────
// Repository 구현체 (DB 상세를 여기서 처리)
class InMemoryUserRepository implements UserRepository {
  private store: Map<number, User> = new Map();

  async findById(id: number) {
    return this.store.get(id) ?? null;
  }

  async save(user: User) {
    this.store.set(user.id, user);
  }
}

// Controller (HTTP 요청 → Use Case 호출)
class UserController {
  private useCase = new UpdateUserNameUseCase(new InMemoryUserRepository());

  async handlePatch(req: { id: number; name: string }) {
    await this.useCase.execute(req.id, req.name);
    return { status: 200, message: '업데이트 완료' };
  }
}

// ── 4층: Framework (Express 등) ───────────────
// app.patch('/users/:id', (req, res) => controller.handlePatch(req.body))
```

### 폴더 구조 예시

```
src/
├── domain/                  # 1층: Entities
│   ├── entities/
│   │   └── User.ts
│   └── exceptions/
├── application/             # 2층: Use Cases
│   ├── usecases/
│   │   └── UpdateUserNameUseCase.ts
│   └── repositories/        # Repository 인터페이스 (추상)
│       └── UserRepository.ts
├── infrastructure/          # 3~4층: Adapters & Frameworks
│   ├── persistence/
│   │   └── TypeORMUserRepository.ts   # Repository 구현체
│   └── http/
│       └── UserController.ts
└── main.ts                  # 의존성 조립 (DI)
```

### 의존성 역전 (DIP) 적용 방식

```
Use Case          →  UserRepository (인터페이스)
                          ↑ 구현
TypeORM Repository →  UserRepository (인터페이스)
```

Use Case는 인터페이스에만 의존하므로, DB를 TypeORM → MongoDB로 교체해도 Use Case 코드는 변경되지 않는다.

### 장단점

**장점**
- 비즈니스 로직이 프레임워크·DB와 완전히 독립
- 각 계층을 독립적으로 테스트 가능 (특히 Use Case 단위 테스트)
- 기술 스택 교체가 유연함 (DB, 프레임워크 변경 시 핵심 로직 불변)
- 유지보수성과 확장성이 높음

**단점**
- 초기 설계와 구조 잡기에 많은 시간 소요
- 소규모 프로젝트에는 과도한 복잡도
- 계층 간 데이터 변환(DTO) 코드가 많아질 수 있음
- 팀원 모두가 원칙을 이해해야 효과적

### 활용 사례
- 대규모 엔터프라이즈 백엔드
- NestJS (모듈 기반 DI 구조)
- Android Clean Architecture (Domain / Data / Presentation)
- 도메인 주도 설계(DDD)와 함께 적용

---

## 6. 패턴 비교 요약

### 디자인 패턴 비교 (옵저버 vs 프록시)

| 항목 | 옵저버 패턴 | 프록시 패턴 |
|---|---|---|
| **분류** | 행위(Behavioral) | 구조(Structural) |
| **목적** | 상태 변화 알림 | 접근 제어 및 기능 추가 |
| **관계** | 1:N (발행-구독) | 1:1 (대리) |
| **결합도** | 느슨한 결합 | Subject와 동일 인터페이스 |
| **주 사용처** | 이벤트 시스템, 반응형 | 캐싱, 인증, 지연로딩 |

### 아키텍처 패턴 비교 (MVC vs MVVM)

| 항목 | MVC | MVVM |
|---|---|---|
| **데이터 흐름** | 단방향 (Controller → View) | 양방향 바인딩 |
| **View 업데이트** | Controller가 명시적으로 처리 | ViewModel 상태 변화 시 자동 |
| **의존 방향** | View → Controller → Model | View → ViewModel → Model |
| **테스트** | Controller 테스트 | ViewModel 단독 테스트 |
| **적합한 환경** | 서버 사이드, REST API | 프론트엔드 SPA, 모바일 앱 |

---

> 패턴은 도구다. 문제 상황에 맞는 패턴을 선택하는 것이 핵심이다.
