# HTML 설계서 템플릿 가이드

이 문서는 모듈 설계서 HTML 생성 시 따라야 할 구조, 스타일, 컴포넌트 규격을 정의한다.

## 1. 전체 HTML 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{프로젝트명} — 계층적 모듈 설계서</title>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700;900&family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>/* 스타일 */</style>
</head>
<body>
  <div class="container">
    <!-- 1. 헤더 -->
    <!-- 2. 목차 (TOC) -->
    <!-- 3. 기술스택 -->
    <!-- 4. 폴더 구조 -->
    <!-- 5. 전체 메인 파이프라인 -->
    <!-- 6. 공통 모듈 (C0) -->
    <!-- 7. 기능 모듈 (F1~FN) -->
    <!-- 8. 의존성 매트릭스 -->
    <!-- 9. 구현 체크리스트 -->
    <!-- 10. 푸터 -->
  </div>
  <script>/* 인터랙션 */</script>
</body>
</html>
```

## 2. 디자인 시스템

### 2.1 컬러 팔레트 (CSS Variables)

```css
:root {
  /* 배경 계층 */
  --bg-base: #0a0c12;        /* 페이지 배경 */
  --bg-surface-1: #10131e;   /* 카드 배경 */
  --bg-surface-2: #171c2a;   /* 모듈 블록 헤더 */
  --bg-surface-3: #1e2438;   /* 호버 상태 */
  
  /* 테두리 */
  --border-1: #252b45;       /* 기본 테두리 */
  --border-2: #313858;       /* 강조 테두리 */
  
  /* 텍스트 */
  --text-primary: #dfe2ef;   /* 본문 */
  --text-secondary: #8f94b5; /* 보조 텍스트 */
  --text-muted: #565b78;     /* 흐린 텍스트 */
  
  /* 시맨틱 컬러 */
  --color-in: #4ecdc4;       /* IN 표시 (녹색 계열) */
  --color-out: #f7a44c;      /* OUT 표시 (주황색) */
  --color-api: #56d4e0;      /* API 표시 (시안) */
  --color-event: #efd060;    /* 이벤트 (노란색) */
  --color-type: #5b8def;     /* 타입 표시 (파란색) */
  --color-warning: #ef6868;  /* 경고 (빨간색) */
  --color-deprecated: #565b78; /* 폐기 */
  
  /* Feature별 구분 컬러 (최대 16개) */
  --f-common: #56d4e0;    /* C0 시안 */
  --f1: #5b8def;           /* F1 블루 */
  --f2: #9b7fed;           /* F2 퍼플 */
  --f3: #f7a44c;           /* F3 오렌지 */
  --f4: #4ecdc4;           /* F4 그린 */
  --f5: #e87fc4;           /* F5 핑크 */
  --f6: #ef6868;           /* F6 레드 */
  --f7: #efd060;           /* F7 옐로 */
  --f8: #8fd460;           /* F8 라임 */
  --f9: #5b8def;           /* F9 블루 (밝은) */
  --f10: #d4a0f0;          /* F10 라벤더 */
}
```

### 2.2 타이포그래피

```css
body {
  font-family: 'Noto Sans KR', sans-serif;
  line-height: 1.7;
}

code, .mono, .type, .param, .module-id {
  font-family: 'JetBrains Mono', monospace;
}

/* 크기 체계 */
.project-title { font-size: 28px; font-weight: 900; }
.section-title { font-size: 16px; font-weight: 800; }
.module-title  { font-size: 13px; font-weight: 700; }
.body-text     { font-size: 11.5px; }
.code-text     { font-size: 10.5px; }
.label-text    { font-size: 9px; font-weight: 700; letter-spacing: 1.5px; text-transform: uppercase; }
```

## 3. 필수 컴포넌트

### 3.1 프로젝트 헤더

```html
<header class="project-header">
  <span class="version-badge">Hierarchical Module Design v1.0</span>
  <h1 class="project-title">{프로젝트명} 계층적 모듈 설계서</h1>
  <p class="project-desc">{프로젝트 한 줄 설명}</p>
  <div class="meta-info">
    <span>생성일: {날짜}</span>
    <span>총 모듈 수: {N}개</span>
    <span>기술스택: {핵심 기술}</span>
  </div>
</header>
```

### 3.2 목차 (TOC) - 인터랙티브 네비게이션

```html
<nav class="toc-nav">
  <h2>목차</h2>
  <div class="toc-grid">
    <a href="#tech-stack" class="toc-item">1. 기술스택</a>
    <a href="#folder-structure" class="toc-item">2. 폴더 구조</a>
    <a href="#main-pipeline" class="toc-item">3. 전체 파이프라인</a>
    <a href="#c0" class="toc-item" style="border-color: var(--f-common)">C0. 공통 모듈 ({N}개)</a>
    <a href="#f1" class="toc-item" style="border-color: var(--f1)">F1. {기능명} ({N}개)</a>
    <!-- ... -->
    <a href="#dependencies" class="toc-item">의존성 매트릭스</a>
    <a href="#checklist" class="toc-item">구현 체크리스트</a>
  </div>
</nav>
```

### 3.3 기술스택 테이블

```html
<section id="tech-stack" class="card">
  <div class="card-header"><h2>1. 기술스택</h2></div>
  <div class="card-body">
    <table class="tech-table">
      <thead>
        <tr><th>영역</th><th>기술</th><th>역할</th><th>배포환경</th></tr>
      </thead>
      <tbody>
        <tr>
          <td><b style="color: var(--f1)">백엔드</b></td>
          <td><span class="tech-tag backend">Python 3.12</span> <span class="tech-tag backend">FastAPI</span></td>
          <td>API 서버, 비즈니스 로직</td>
          <td>Docker</td>
        </tr>
        <!-- ... -->
      </tbody>
    </table>
  </div>
</section>
```

### 3.4 폴더 구조 (트리)

```html
<section id="folder-structure" class="card">
  <div class="card-header"><h2>2. 프로젝트 폴더 구조</h2></div>
  <div class="card-body">
    <pre class="folder-tree"><span class="folder">project-root/</span>
│
├── <span class="folder">src/</span>
│   ├── <span class="folder">common/</span>                    <span class="comment">── C0 공통 모듈 ({N}개)</span>
│   │   ├── secret_vault.py            <span class="comment">── C0.1 SecretVault</span>
│   │   ├── database_gateway.py        <span class="comment">── C0.2 DatabaseGateway</span>
│   │   └── ...
│   │
│   ├── <span class="folder">f1_{feature_name}/</span>          <span class="comment">── F1 {기능명} ({N}개)</span>
│   │   ├── scheduler.py               <span class="comment">── F1.1 Scheduler</span>
│   │   └── ...
│   │
│   └── main.py                        <span class="comment">── 엔트리포인트</span>
│
├── .env                               <span class="comment">── 환경변수</span>
└── README.md</pre>
  </div>
</section>
```

### 3.5 메인 파이프라인 (★핵심★)

전체 메인 파이프라인과 각 Feature 내부 파이프라인을 시각화한다.

```html
<!-- 전체 메인 파이프라인 -->
<section id="main-pipeline" class="card">
  <div class="card-header"><h2>3. 전체 메인 파이프라인</h2></div>
  <div class="card-body">
    <!-- 파이프라인 플로우 -->
    <div class="pipeline-flow">
      <div class="pipeline-node" style="--node-color: var(--f-common)">
        <span class="node-id">C0</span>
        <span class="node-name">초기화</span>
      </div>
      <div class="pipeline-arrow">→</div>
      <div class="pipeline-node" style="--node-color: var(--f1)">
        <span class="node-id">F1</span>
        <span class="node-name">데이터수집</span>
      </div>
      <div class="pipeline-arrow">→</div>
      <!-- ... -->
    </div>
    
    <!-- 파이프라인 상세 IN/OUT -->
    <div class="pipeline-detail">
      <table class="pipeline-io-table">
        <thead>
          <tr>
            <th>단계</th>
            <th>IN (이전 단계의 OUT)</th>
            <th>OUT (다음 단계의 IN)</th>
            <th>설명</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td><code>C0 → F1</code></td>
            <td><code class="in-type">InitResult {services: ServiceContainer}</code></td>
            <td><code class="out-type">CrawlResult {articles: list[Article], count: int}</code></td>
            <td>초기화 완료 후 수집된 서비스를 F1에 전달</td>
          </tr>
          <!-- ... -->
        </tbody>
      </table>
    </div>
  </div>
</section>
```

### 3.6 모듈 블록 (★핵심★)

각 모듈의 상세 정보를 접기/펴기 가능한 블록으로 표시한다.

```html
<div class="module-block" data-module-id="F1.3">
  <!-- 모듈 헤더 (클릭으로 토글) -->
  <div class="module-header" onclick="toggleModule(this)">
    <span class="module-id" style="background: rgba(91,141,239,.12); color: var(--f1)">F1.3</span>
    <h3 class="module-name">CrawlVerifier — 기사 품질 검증</h3>
    <span class="module-toggle">▶</span>
  </div>
  
  <!-- 모듈 상세 (토글 대상) -->
  <div class="module-body">
    
    <!-- IN 명세 -->
    <div class="io-block io-in">
      <div class="io-label in-label">▼ IN (메인)</div>
      <div class="io-spec">
        <code class="param-name">raw_article</code>: 
        <code class="param-type">RawArticle</code>
        <span class="param-source">← F1.2.crawl()</span>
        <p class="param-desc">크롤링된 미가공 기사 1건</p>
        <div class="type-detail">
          <code>RawArticle {
  source: str,        <span class="comment"># 소스명 (reuters, bloomberg...)</span>
  title: str,         <span class="comment"># 기사 제목</span>
  content: str,       <span class="comment"># 기사 본문</span>
  url: str,           <span class="comment"># 원문 URL</span>
  published_at: datetime,  <span class="comment"># 발행일시</span>
  language: str       <span class="comment"># 언어 코드 (en, ko)</span>
}</code>
        </div>
      </div>
    </div>
    
    <!-- 보조 IN -->
    <div class="io-block io-in-aux">
      <div class="io-label in-aux-label">▼ IN (보조)</div>
      <div class="io-spec">
        <code class="param-name">min_quality_score</code>: 
        <code class="param-type">float</code>
        <span class="param-source">← config.yaml</span>
        <p class="param-desc">최소 품질 점수 임계값 (기본: 0.3)</p>
      </div>
    </div>
    
    <!-- OUT 명세 -->
    <div class="io-block io-out">
      <div class="io-label out-label">▲ OUT</div>
      <div class="io-spec">
        <code class="param-name">반환</code>: 
        <code class="param-type">VerifiedArticle | None</code>
        <p class="param-desc">검증 통과 시 VerifiedArticle, 실패 시 None</p>
        <div class="type-detail">
          <code>VerifiedArticle {
  source: str,
  title: str,
  content: str,
  url: str,
  published_at: datetime,
  language: str,
  quality_score: float,   <span class="comment"># 품질 점수 (0.0~1.0)</span>
  word_count: int          <span class="comment"># 단어 수</span>
}</code>
        </div>
      </div>
    </div>
    
    <!-- 내부 로직 -->
    <div class="logic-block">
      <div class="io-label logic-label">⚙ 내부 로직</div>
      <ol class="logic-steps">
        <li>필드 완전성 검사: title, content, url 필수</li>
        <li>언어 판별: content의 언어가 en 또는 ko인지 확인</li>
        <li>시간 검증: published_at이 24시간 이내인지 확인</li>
        <li>품질 점수 계산: content 길이, 키워드 밀도 기반</li>
        <li>임계값 비교: quality_score >= min_quality_score</li>
      </ol>
    </div>
    
    <!-- 사용하는 공통 모듈 -->
    <div class="deps-block">
      <span class="dep-label">사용 공통모듈:</span>
      <span class="dep-tag">C0.8 Logger</span>
    </div>
    
    <!-- 주의사항 -->
    <div class="note-block warning">
      <strong>주의:</strong> None 반환 시 파이프라인에서 해당 기사를 조용히 드랍한다. 에러를 발생시키지 않는다.
    </div>
  </div>
</div>
```

### 3.7 Feature 카드 레이아웃

```html
<section id="f1" class="card feature-card" style="--feature-color: var(--f1)">
  <div class="card-header">
    <div class="feature-icon">F1</div>
    <h2>데이터 수집 (Data Collection) — {N}개 모듈</h2>
  </div>
  <div class="card-body">
    <!-- Feature 설명 -->
    <p class="feature-desc">
      위치: <code>src/f1_collection/</code> · 
      사용 Common: <code>C0.1</code>, <code>C0.2</code>, <code>C0.4</code>, <code>C0.8</code>
    </p>
    
    <!-- Feature 내부 파이프라인 -->
    <div class="feature-pipeline">
      <div class="pipeline-flow">
        <!-- Feature 내부의 직선 파이프라인 노드들 -->
      </div>
      <!-- Feature 파이프라인의 단계별 IN/OUT 테이블 -->
      <table class="pipeline-io-table">...</table>
    </div>
    
    <!-- 전체 펼치기/접기 컨트롤 -->
    <div class="feature-controls">
      <button onclick="toggleFeatureModules(this, true)">▼ 모두 펼치기</button>
      <button onclick="toggleFeatureModules(this, false)">▲ 모두 접기</button>
    </div>
    
    <!-- 모듈 블록들 -->
    <div class="module-block">...</div>
    <div class="module-block">...</div>
    <!-- ... -->
  </div>
</section>
```

### 3.8 의존성 매트릭스

```html
<section id="dependencies" class="card">
  <div class="card-header"><h2>의존성 매트릭스</h2></div>
  <div class="card-body">
    <table class="dep-matrix">
      <thead>
        <tr>
          <th></th>
          <th>C0.1</th><th>C0.2</th><th>C0.3</th><th>C0.4</th>
          <!-- ... -->
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><b>F1.1</b></td>
          <td></td><td></td><td></td><td>●</td>
        </tr>
        <tr>
          <td><b>F1.2</b></td>
          <td>●</td><td>●</td><td></td><td>●</td>
        </tr>
        <!-- ... -->
      </tbody>
    </table>
  </div>
</section>
```

### 3.9 구현 체크리스트

```html
<section id="checklist" class="card">
  <div class="card-header"><h2>구현 체크리스트</h2></div>
  <div class="card-body">
    <div class="checklist-group">
      <h3 style="color: var(--f-common)">Phase 1: 공통 모듈 (C0)</h3>
      <ul class="checklist">
        <li><input type="checkbox"> C0.1 SecretVault</li>
        <li><input type="checkbox"> C0.2 DatabaseGateway</li>
        <!-- ... -->
      </ul>
    </div>
    <div class="checklist-group">
      <h3 style="color: var(--f1)">Phase 2: F1 데이터 수집</h3>
      <ul class="checklist">
        <li><input type="checkbox"> F1.1 CrawlScheduler</li>
        <!-- ... -->
      </ul>
    </div>
  </div>
</section>
```

## 4. 인터랙션 (JavaScript)

### 4.1 필수 기능

```javascript
// 1. 모듈 접기/펼치기
function toggleModule(header) {
  const block = header.parentElement;
  block.classList.toggle('open');
}

// 2. Feature 내 모듈 전체 접기/펼치기
function toggleFeatureModules(btn, open) {
  const card = btn.closest('.feature-card');
  card.querySelectorAll('.module-block').forEach(m => {
    m.classList.toggle('open', open);
  });
}

// 3. 전체 모듈 접기/펼치기
function toggleAll(open) {
  document.querySelectorAll('.module-block').forEach(m => {
    m.classList.toggle('open', open);
  });
}

// 4. 목차 스크롤 하이라이트
// IntersectionObserver로 현재 보이는 섹션 감지

// 5. 검색 기능 (선택)
// 모듈 ID나 이름으로 검색하여 해당 모듈만 표시
```

### 4.2 인터랙티브 파이프라인 (선택적 고급 기능)

파이프라인 노드를 클릭하면 해당 모듈로 스크롤하는 기능:

```javascript
document.querySelectorAll('.pipeline-node').forEach(node => {
  node.addEventListener('click', () => {
    const moduleId = node.dataset.moduleId;
    const target = document.querySelector(`[data-module-id="${moduleId}"]`);
    if (target) {
      target.classList.add('open');
      target.scrollIntoView({ behavior: 'smooth', block: 'center' });
      target.classList.add('highlight');
      setTimeout(() => target.classList.remove('highlight'), 2000);
    }
  });
});
```

## 5. 반응형 디자인

```css
@media (max-width: 768px) {
  .container { padding: 12px 8px; }
  .pipeline-flow { flex-wrap: wrap; }
  .tech-table { font-size: 10px; }
  .dep-matrix { font-size: 9px; }
  .module-header h3 { font-size: 11.5px; }
}
```

## 6. HTML 품질 체크리스트

생성된 HTML이 다음을 모두 만족하는지 확인:

- [ ] 모든 모듈에 IN/OUT이 완전히 명세되어 있다
- [ ] IN의 파라미터명, 타입, 출처, 설명이 모두 있다
- [ ] OUT의 반환타입과 필드 구조가 명시되어 있다
- [ ] 메인 파이프라인이 시각적으로 직선 흐름이다
- [ ] 각 Feature 내부 파이프라인도 표시되어 있다
- [ ] 파이프라인의 단계별 IN/OUT 테이블이 있다
- [ ] 모듈 접기/펼치기가 동작한다
- [ ] Feature별 색상 구분이 되어 있다
- [ ] IN=녹색, OUT=주황, API=시안 색상 코딩이 적용되어 있다
- [ ] 모든 타입/변수명이 모노스페이스 폰트이다
- [ ] 폴더 구조에 모듈 ID 주석이 있다
- [ ] 의존성 매트릭스가 있다
- [ ] 구현 체크리스트가 있다
- [ ] 목차에서 각 섹션으로 이동 가능하다
