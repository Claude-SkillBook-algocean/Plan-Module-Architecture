# AI 설계 행동 지침서

이 문서는 Claude가 계층적 모듈 설계를 수행할 때 따라야 하는 구체적인 행동 지침이다.

## 1. 대화 흐름 프로토콜

### 1.1 첫 응답: 프로젝트 파악

사용자가 설계를 요청하면, 즉시 코드를 작성하지 말고 다음을 먼저 파악한다:

```
[확인 사항]
1. 프로젝트명과 한 줄 설명
2. 핵심 기능 목록 (대략적으로)
3. 사용 환경 (로컬/서버/클라우드)
4. 사용자의 기술 선호도 (언어, 프레임워크)
5. 프로젝트 규모 감각 (소/중/대)
```

사용자가 이미 충분한 정보를 제공했다면 불필요한 질문을 하지 않고 바로 기술스택 제안으로 넘어간다.

### 1.2 기술스택 협의

기술스택을 표 형태로 제안하고 **반드시 사용자 승인을 받는다**.

```
"다음 기술스택을 제안드립니다. 수정하거나 추가할 부분이 있으시면 말씀해주세요."

| 영역 | 기술 | 선택 이유 |
|------|------|-----------|
| ... | ... | ... |
```

사용자가 "좋아요", "이대로 진행", "OK" 등 승인하면 다음 단계로 진행.

### 1.3 모듈 계층 설계

전체 모듈 계층 트리를 먼저 보여준다:

```
C0 공통 모듈 (N개)
├── C0.1 SecretVault
├── C0.2 DatabaseGateway
└── ...

F1 {기능1} (N개)
├── F1.1 ...
├── F1.2 ...
└── ...

F2 {기능2} (N개)
└── ...
```

모듈 수가 30개 이상이면 **Feature 단위로 나누어 승인**을 받는다.

### 1.4 상세 IN/OUT 명세

모듈 계층이 승인되면, 각 모듈의 상세 IN/OUT을 작성한다.
**이 단계는 코드가 아닌 명세**이다. 실제 구현 코드를 작성하지 않는다.

### 1.5 HTML 설계서 생성

모든 명세가 완성되면 HTML 설계서를 생성한다.
`references/html-template-guide.md`의 구조와 스타일을 따른다.

## 2. 모듈 설계 시 판단 기준

### 2.1 모듈을 분할해야 하는 신호

다음 중 하나라도 해당하면 모듈을 분할한다:

- 모듈 설명에 "그리고", "또한", "및" 같은 연결어가 필요할 때
- OUT이 2가지 이상의 독립적인 데이터를 포함할 때
- 하나의 모듈에서 2개 이상의 외부 API를 호출할 때
- 모듈의 예상 코드 길이가 200줄을 초과할 때
- 테스트 작성 시 mock이 3개 이상 필요할 때

### 2.2 모듈을 합쳐야 하는 신호

다음 중 하나라도 해당하면 너무 과도하게 분할한 것이므로 합친다:

- 모듈이 단순히 값을 전달만 할 때 (pass-through)
- 두 모듈이 항상 함께 호출되고 독립적으로 사용되지 않을 때
- 모듈의 IN/OUT이 실질적으로 동일할 때

### 2.3 Feature 분리 기준

하나의 Feature로 묶이는 기준:
- 같은 데이터 도메인을 다루는 모듈
- 변경 시 함께 변경되는 모듈
- 동일한 비즈니스 목적

Feature를 분리하는 기준:
- 독립적으로 배포/테스트 가능한 단위
- 서로 다른 외부 시스템에 의존하는 경우
- 팀/담당자가 다른 경우

## 3. IN/OUT 명세 작성 규칙

### 3.1 타입 표기법

Python 스타일의 타입 힌트를 기본으로 사용:

```python
# 기본 타입
str, int, float, bool, None

# 컬렉션
list[Article], dict[str, float], tuple[int, str]

# Optional
Optional[float], str | None

# 커스텀 타입 (데이터 클래스/모델)
UserProfile, CrawlResult, TradingDecision

# 콜백/함수 타입
Callable[[str], bool], Coroutine[Any, Any, Result]
```

TypeScript 프로젝트의 경우:
```typescript
string, number, boolean, null
Array<Article>, Record<string, number>
UserProfile | null
(input: string) => boolean
Promise<r>
```

### 3.2 OUT 명세 상세도

OUT은 반드시 **필드 레벨까지** 명세한다:

```
OUT: CrawlResult
  필드:
    - total_count: int           # 전체 수집 건수
    - new_count: int             # 신규 건수 (중복 제외)
    - failed_sources: list[str]  # 실패한 소스 이름 목록
    - articles: list[Article]    # 수집된 기사 목록
    - duration_seconds: float    # 소요 시간 (초)
    - timestamp: datetime        # 수집 완료 시각
```

### 3.3 IN 출처 표기법

IN의 출처는 다음 형식으로 명확하게 표기:

```
← C0.1.get_secret("DB_URL")      # 공통모듈의 특정 메서드 호출
← F1.2.crawl() 반환값              # 다른 모듈의 OUT
← config.yaml["crawl"]["timeout"]  # 설정 파일
← .env → C0.1                      # 환경변수 (C0.1 경유)
← EventBus("ArticleCollected")     # 이벤트 수신
← Redis key: "cache:vix:latest"    # 캐시에서 조회
← user_input                       # 사용자 입력 / API 요청
```

## 4. 메인 파이프라인 설계 원칙

### 4.1 전체 파이프라인

```
[트리거] → [C0 초기화] → [F1] → [F2] → ... → [FN] → [종료]
```

- 트리거: 사용자 입력, 스케줄, 이벤트 등
- C0 초기화: 의존성 주입 컨테이너 생성
- F1~FN: 순차 실행, 각 Feature의 OUT이 다음 Feature의 IN
- 종료: 리소스 정리, 결과 보고

### 4.2 파이프라인 IN/OUT 체인

파이프라인의 각 단계별 IN/OUT을 명시적으로 연결:

```
C0.init()
  → OUT: ServiceContainer {db, cache, http, ai, broker}

F1.execute(services: ServiceContainer)
  → OUT: CollectionResult {articles: list[Article], count: int}

F2.execute(articles: list[Article], services: ServiceContainer)
  → OUT: AnalysisResult {decisions: list[Decision], regime: Regime}

F3.execute(decisions: list[Decision], services: ServiceContainer)
  → OUT: ExecutionResult {orders: list[Order], status: str}
```

### 4.3 조건 분기 처리

파이프라인 레벨에서 분기를 피하고, 모듈 내부에서 처리:

```
# 잘못된 방법 (파이프라인 레벨 분기)
if market.is_open:
    F3.trade()
else:
    F3.analyze_only()

# 올바른 방법 (모듈 내부에서 처리)
F3.execute()  # 내부에서 market.is_open 확인 후 적절히 처리
  → OUT: ExecutionResult  # 항상 같은 타입의 OUT
```

## 5. HTML 생성 시 주의사항

### 5.1 반드시 포함해야 하는 것

1. 프로젝트 헤더 (이름, 버전, 설명, 날짜, 모듈 총 수)
2. 인터랙티브 목차
3. 기술스택 테이블
4. 폴더 구조 트리 (모듈 ID 주석 포함)
5. **전체 메인 파이프라인 + 단계별 IN/OUT 테이블**
6. C0 공통 모듈 전체 상세
7. 각 Feature 카드:
   - Feature 내부 파이프라인 + 단계별 IN/OUT 테이블
   - 모든 모듈의 상세 IN/OUT 명세 (토글)
   - 내부 로직 설명
   - 사용하는 공통모듈 태그
   - 주의사항/제약사항
8. 의존성 매트릭스
9. 구현 체크리스트 (체크박스)

### 5.2 절대 빠뜨리면 안 되는 것

- **파이프라인의 단계별 IN/OUT 연결 테이블**: 이것 없이는 모듈 간 데이터 흐름을 파악할 수 없다
- **OUT의 필드 구조**: OUT이 "Result"만 적혀있으면 안 된다. 필드까지 명시
- **IN의 출처 모듈**: 어디서 오는 데이터인지 모르면 구현 불가
- **내부 로직 요약**: 알고리즘이나 처리 순서를 모르면 구현 방향 불명확

### 5.3 크기 관리

- 모듈이 50개 이상: 접기 기능 필수, 기본적으로 모두 접은 상태
- 모듈이 100개 이상: Feature별 탭 또는 페이지 분리 고려
- 단일 HTML 파일 10,000줄 초과 시: Feature별 분할 HTML 제안

## 6. 대규모 프로젝트 분할 전략

### 6.1 작업 분할

```
[Phase 1] 기술스택 + 전체 구조 확정 (1회 응답)
[Phase 2] C0 공통 모듈 상세 설계 (1회 응답)
[Phase 3] F1 상세 설계 (1회 응답)
[Phase 4] F2 상세 설계 (1회 응답)
...
[Phase N] HTML 통합 생성 (1~2회 응답)
[Phase N+1] 검토 및 수정 (필요 시)
```

### 6.2 연속성 유지

매 응답 마지막에:
```
[현재 상태: Phase 3 / F1 상세 설계 완료]
[다음 단계: Phase 4 / F2 상세 설계 예정]
[전체 진행: 3/8 단계]
```

### 6.3 일관성 유지

- 이전 단계에서 정의한 타입명, 변수명, 모듈명을 정확히 동일하게 사용
- OUT 타입이 다음 단계의 IN과 정확히 매칭되는지 검증
- 공통모듈(C0) 참조가 일관된지 확인

## 7. 품질 자가 검증

HTML 생성 후 다음을 자동 검증:

1. **OUT = 1 검증**: 모든 모듈의 OUT이 정확히 1개인가?
2. **IN 체인 검증**: 파이프라인의 OUT → IN이 타입이 일치하는가?
3. **참조 무결성**: C0.N 참조가 실제 존재하는 C0 모듈인가?
4. **순환 의존 검증**: A→B→A 같은 순환이 없는가?
5. **누락 검증**: 명세가 빠진 모듈이 없는가?
6. **네이밍 일관성**: snake_case/CamelCase가 일관된가?
