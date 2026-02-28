# 모듈 설계 규칙 상세 가이드

## 1. 모듈 정의

모듈이란 **단일 책임을 가진 독립적인 처리 단위**이다. 
모든 모듈은 다음 구조를 따른다:

```
┌──────────────────────────────────────────┐
│  [모듈 ID]  모듈명                         │
│                                          │
│  IN (메인) ──→ [내부 처리 로직] ──→ OUT    │
│  IN (보조1) ↗                             │
│  IN (보조2) ↗                             │
│  IN (보조N) ↗                             │
└──────────────────────────────────────────┘
```

## 2. IN/OUT 규칙

### 2.1 OUT 규칙 (절대 원칙)

- **하나의 모듈은 반드시 정확히 1개의 OUT만 가진다**
- OUT은 해당 모듈의 책임에 맞는 단일 결과물이다
- OUT의 형태: 데이터 객체, 리스트, API 응답, 이벤트, 함수 반환값 등 다양하지만 **1개**
- 여러 결과가 필요하면? → 모듈을 여러 번 재사용하거나 별도 모듈로 분리

```
# 올바른 예시
모듈A → OUT: UserProfile (name, email, role)

# 잘못된 예시 (금지!)
모듈A → OUT1: UserProfile
        OUT2: UserSettings  ← 이건 별도 모듈로!
```

### 2.2 IN 규칙

- **메인 IN**: 메인 파이프라인에서 이전 모듈의 OUT으로부터 받는 핵심 입력 (1개)
- **보조 IN**: 공통모듈(C0), 외부 API, 설정값 등에서 받는 부가 입력 (0~N개)
- 모든 IN에는 다음을 명시한다:
  - 파라미터명 (snake_case)
  - 타입 (구체적: str, int, list[ArticleDTO], Optional[float] 등)
  - 출처 (어떤 모듈의 OUT인지, 예: C0.1.get_secret)
  - 설명 (이 데이터가 무엇인지 한 줄 설명)

### 2.3 IN/OUT 명세 작성 형식

```
IN (메인):
  - raw_articles: list[RawArticle]
    출처: F1.2.crawl() 반환값
    설명: 크롤링된 미가공 기사 목록

IN (보조):
  - db_session: AsyncSession
    출처: C0.2.get_session()
    설명: PostgreSQL 비동기 DB 세션
  - cache_client: CacheClient
    출처: C0.3
    설명: Redis 캐시 클라이언트

OUT:
  - DeduplicationResult
    필드:
      - is_new: bool          # 신규 기사 여부
      - article_hash: str     # SHA-256 해시값
      - duplicate_id: Optional[int]  # 중복 기사 ID (있을 경우)
```

## 3. 모듈 계층 구조

### 3.1 계층 분류

```
Level 0 (최상위): 프로젝트 전체 오케스트레이션
  └─ Level 1 (Feature): 기능 단위 (F1, F2, F3...)
       └─ Level 2 (Module): Feature 내 모듈 (F1.1, F1.2...)
            └─ Level 3 (Sub-Module): 모듈 내부 함수/클래스 (구현 단계에서)

Side (공통): C0 공통 모듈 (모든 Level에서 참조 가능)
```

### 3.2 명명 규칙

```
공통 모듈: C0.{번호}  예) C0.1 SecretVault
기능 모듈: F{기능번호}.{모듈번호}  예) F1.3 CrawlVerifier
특수 기능: F{문자}.{모듈번호}  예) FW.1 WebSocketManager (W=WebSocket)
```

### 3.3 Feature 분해 기준

하나의 Feature로 묶이려면:
- 같은 도메인의 데이터를 다루거나
- 동일한 비즈니스 목적을 위한 처리이거나
- 변경 시 함께 변경되는 모듈들의 집합

Feature가 너무 크면(모듈 15개 이상) 분할을 검토한다.

## 4. 메인 파이프라인 규칙

### 4.1 전체 메인 파이프라인

프로젝트 전체를 관통하는 **단일 직선 흐름**이다:

```
[시작] → F1 → F2 → F3 → ... → FN → [종료]
```

- 분기(if/else)는 파이프라인 레벨에서 금지
- 조건 분기는 모듈 내부에서 처리
- 파이프라인의 각 노드는 Feature의 오케스트레이터 모듈

### 4.2 Feature 내부 파이프라인

각 Feature 내부도 동일하게 직선 파이프라인:

```
F1: [F1.1] → [F1.2] → [F1.3] → [F1.4 (오케스트레이터)]
```

- Feature의 마지막 모듈은 해당 Feature의 오케스트레이터
- 오케스트레이터가 내부 모듈을 호출하고 결과를 조합하여 Feature OUT 생성

### 4.3 파이프라인 시각화 규칙

HTML 설계서에서:
```
전체 파이프라인:
[시작 버튼] → [C0 초기화] → [F1 데이터수집] → [F2 분석] → [F3 실행] → [종료]

F1 내부 파이프라인:
[F1.1 스케줄러] → [F1.2 크롤러] → [F1.3 검증기] → [F1.4 중복제거] → [F1.5 저장] → [F1.6 엔진]
                                                       ↑ C0.3 (캐시)    ↑ C0.2 (DB)
```

보조 IN은 파이프라인 아래쪽에서 화살표(↑)로 표현한다.

## 5. 단일 책임 원칙 (SRP) 검증

### 5.1 SRP 검증 질문

모듈이 SRP를 만족하는지 확인하는 질문:

1. "이 모듈이 변경되어야 하는 이유가 몇 가지인가?"
   → 2가지 이상이면 분할

2. "이 모듈의 이름을 한 단어로 표현할 수 있는가?"
   → "~하고 ~하는 모듈"이면 분할 대상

3. "이 모듈의 OUT을 설명하는데 'and'가 필요한가?"
   → "A and B를 반환"이면 분할 대상

### 5.2 SRP 위반 패턴과 해결

```
# 위반: 크롤링 + 파싱 + 저장을 하나의 모듈에서
DataCollector → OUT: SavedArticles
  (크롤링도 하고, 파싱도 하고, DB에도 저장)

# 해결: 3개 모듈로 분할
Crawler    → OUT: RawHTML
Parser     → OUT: ParsedArticle  
Persister  → OUT: SaveResult
```

### 5.3 최하위 모듈 예시

최하위 모듈은 **원자적(Atomic)** 기능만 수행:

```
좋은 예:
- EnvLoader: .env에서 특정 키의 값을 읽어 반환
- HashCalculator: 텍스트의 SHA-256 해시를 계산
- ExchangeRateFetcher: 환율 API에서 현재 환율을 조회
- JsonParser: JSON 문자열을 파싱하여 객체로 반환
- DateFormatter: 날짜를 지정 포맷으로 변환

나쁜 예:
- DataProcessor: 데이터를 "처리"한다 (→ 뭘 처리하는데?)
- Utility: 여러 유틸리티 기능 (→ 각각 분리해야 함)
- Manager: 여러 것을 "관리"한다 (→ 구체적으로 분해)
```

## 6. 공통 모듈(C0) 설계 원칙

### 6.1 C0에 들어가야 하는 것

- 환경변수/시크릿 관리
- DB/캐시 연결
- HTTP 클라이언트
- 외부 API 게이트웨이
- 로깅/에러 처리
- 이벤트 버스 (Feature 간 통신)
- 시간/스케줄 관리

### 6.2 C0 사용 규칙

- C0은 Feature 폴더 안에 절대 두지 않는다 (`src/common/`)
- Feature는 C0을 직접 import하지 않고, DI(의존성 주입)로 받는다
- C0 모듈끼리는 최소한의 의존만 (C0.1 → C0.2는 OK, 순환 의존 금지)

### 6.3 DI 흐름 패턴

```
C0.1 (시크릿 로드)
  ↓ 키 전달
C0.2~C0.N (인프라 인스턴스 생성)
  ↓ 인스턴스 전달
Feature Manager (인프라를 주입받음)
  ↓ 필요한 것만 전달
Feature Module (파라미터로 전달받음, 직접 접근 금지)
```

## 7. 모듈 재사용 패턴

같은 모듈을 다른 IN으로 여러 번 사용하는 패턴:

```
예: 여러 소스에서 데이터를 크롤링할 때

GenericCrawler (재사용 가능한 범용 크롤러)
  - 사용 1: IN=RSSConfig → OUT=list[RawArticle]  (RSS 크롤링)
  - 사용 2: IN=APIConfig → OUT=list[RawArticle]   (API 크롤링)
  - 사용 3: IN=ScrapConfig → OUT=list[RawArticle]  (스크래핑)

→ 같은 OUT 타입, 다른 IN 설정으로 재사용
```

## 8. 모듈 의존성 규칙

### 8.1 허용되는 의존 방향

```
Feature → C0 (공통 모듈 사용): OK
Feature 내 상위 모듈 → 하위 모듈: OK
Feature 오케스트레이터 → Feature 내 모듈: OK

Feature → 다른 Feature: 금지! (EventBus 사용)
C0 → Feature: 금지!
하위 모듈 → 상위 모듈: 금지! (순환 의존)
```

### 8.2 Feature 간 통신

Feature 간에는 직접 호출 대신 이벤트를 사용:

```
F1(데이터수집) ──이벤트: ArticleCollected──→ F2(분석)
F2(분석) ──이벤트: AnalysisComplete──→ F3(실행)
```

또는 상위 오케스트레이터가 Feature OUT을 다음 Feature IN으로 전달:

```
F9 (오케스트레이터):
  result_f1 = await f1.execute()  # F1 OUT
  result_f2 = await f2.execute(result_f1)  # F1 OUT → F2 IN
  result_f3 = await f3.execute(result_f2)  # F2 OUT → F3 IN
```

## 9. 설계 검증 체크리스트

설계 완료 후 다음을 모두 확인:

- [ ] 모든 모듈의 OUT이 정확히 1개인가?
- [ ] 모든 IN에 파라미터명, 타입, 출처가 명시되어 있는가?
- [ ] 메인 파이프라인이 분기 없는 직선인가?
- [ ] 각 Feature 내부 파이프라인도 직선인가?
- [ ] SRP 위반 모듈이 없는가? (변경 이유 2가지 이상)
- [ ] C0과 Feature가 물리적으로 분리되어 있는가?
- [ ] Feature 간 직접 의존이 없는가?
- [ ] 순환 의존이 없는가?
- [ ] 최하위 모듈이 원자적인가?
- [ ] 모듈 이름이 기능을 명확히 설명하는가?
