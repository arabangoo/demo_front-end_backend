# 📄 기존 HTML/CSS/JS 프론트엔드 구조 (arabangoo.com)

> **프로젝트**: arabangoo.com AI 서비스  
> **주제**: 기존 HTML/CSS/JavaScript 아키텍처 심층 분석  
> **분석 대상**: index.html (316줄) / styles.css (619줄) / script.js (624줄)

---

## 📑 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [파일 구조 분석](#2-파일-구조-분석)
3. [HTML 구조 상세 분석](#3-html-구조-상세-분석)
4. [CSS 스타일 상세 분석](#4-css-스타일-상세-분석)
5. [JavaScript 로직 상세 분석](#5-javascript-로직-상세-분석)
6. [API 통신 패턴 분석](#6-api-통신-패턴-분석)
7. [사용자 경험 (UX) 분석](#7-사용자-경험-ux-분석)
8. [보안 및 성능 분석](#8-보안-및-성능-분석)
9. [문제점 및 개선 방향](#9-문제점-및-개선-방향)

---

## 1. 프로젝트 개요

### 1.1 프로젝트 정보

```
프로젝트명: arabangoo.com AI 서비스
도메인: https://arabangoo.com
개발 방식: 전통적인 웹 개발 (HTML/CSS/JavaScript)
배포 방식: AWS S3 + CloudFront
```

### 1.2 서비스 구성

총 **12개의 AI 기반 서비스** 제공:

1. **PDF 요약** - CloudFront를 통한 파일 업로드 및 AI 요약
2. **맛집 추천** - 위치 기반 맞춤 추천
3. **영화 추천** - 플랫폼별 장르별 추천 (넷플릭스, 디즈니+, 쿠팡플레이)
4. **외국어 학습** - 영어, 일본어, 중국어 학습 문장 생성
5. **코인 분석** - 업비트 거래대금 기준 AI 분석
6. **논문 추천** - arXiv 기반 최신/고전 논문 추천
7. **나스닥 분석** - 실시간 거래대금 기준 AI 분석
8. **IT 뉴스 요약** - Google News 기반 최신 기사 AI 요약
9. **도서 추천** - 알라딘 베스트셀러 기반 추천
10. **주변 시설** - 위치 기반 시설 찾기 (약국, 병원, 편의점 등)
11. **AI Chat** - 다양한 GPTs 링크 제공
12. **HOME** - 메인 화면

### 1.3 기술 스택

```yaml
Frontend:
  - HTML5
  - CSS3
  - Vanilla JavaScript (ES6+)

Backend:
  - AWS Lambda (Python)
  - AWS API Gateway (REST API)
  - AWS Bedrock (Claude 3.5 Sonnet)

Infrastructure:
  - AWS S3 (정적 웹 호스팅)
  - AWS CloudFront (CDN + PDF 업로드)
  - AWS Lambda (서버리스 컴퓨팅)

External APIs:
  - Google Maps API (위치 기반 서비스)
  - TMDb API (영화 추천)
  - arXiv API (논문 검색)
  - Google News API (뉴스 요약)
  - Aladin API (도서 추천)
  - Upbit API (코인 시세)
  - Yahoo Finance API (나스닥 시세)
```

---

## 2. 파일 구조 분석

### 2.1 전체 구조

```
aws_ai/
├── index.html                      (316줄) - HTML 구조
├── styles.css                      (619줄) - 스타일시트
├── script.js                       (624줄) - JavaScript 로직
│
├── Lambda Functions (백엔드)
│   ├── ai_pdf_summary_lambda.zip
│   ├── ai_restaurant_menu_lambda.zip
│   ├── ai_choice_movie_lambda.zip
│   ├── ai_foreign_language_lambda.zip
│   ├── ai_coin_analysis_lambda.zip
│   ├── ai_arxiv_paper_lambda.zip
│   ├── ai_nasdaq_analysis_lambda.zip
│   ├── ai_news_summary_lambda.zip
│   ├── ai_choice_book_lambda.zip
│   └── ai_choice_place_lambda.zip
│
├── SEO 및 광고
│   ├── robots.txt
│   ├── sitemap.xml
│   ├── ads.txt
│   ├── clickmon.txt
│   └── naver4dd5c5235e41257998c4e07aacb271f3.html
│
└── react-app/                      (React 마이그레이션 버전)
```

### 2.2 코드 라인 분포

| 파일 | 줄 수 | 역할 | 비중 |
|------|-------|------|------|
| **index.html** | 316줄 | HTML 구조, 마크업 | 20.3% |
| **styles.css** | 619줄 | 스타일, 레이아웃, 반응형 | 39.8% |
| **script.js** | 624줄 | 이벤트 처리, API 통신, 로직 | 40.1% |
| **합계** | 1,559줄 | 전체 프론트엔드 | 100% |

---

## 3. HTML 구조 상세 분석

### 3.1 문서 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <!-- 메타 정보 -->
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>아라방구 AI 서비스</title>
  <link rel="stylesheet" href="styles.css"/>
</head>
<body>
  <header>...</header>   <!-- 헤더 및 네비게이션 -->
  <main>...</main>       <!-- 12개 섹션 -->
  <footer>...</footer>   <!-- 푸터 및 광고 -->
  <script src="script.js"></script>
</body>
</html>
```

### 3.2 헤더 구조 (48줄)

```html
<header>
  <h1>arabangoo.com</h1>
  <div class="nav-buttons">
    <!-- 4개의 nav-row로 구성 -->
    <div class="nav-row">
      <button class="nav-btn" data-target="home" id="home-btn">HOME</button>
      <button class="nav-btn" data-target="it-news-section">IT News</button>
      <button class="nav-btn" data-target="paper-recommendation-section">논문 추천</button>
    </div>
    <div class="nav-row">
      <button class="nav-btn" data-target="movie-section">영화 추천</button>
      <button class="nav-btn" data-target="place-section">주변 시설</button>
      <button class="nav-btn" data-target="restaurant-section">맛집 추천</button>
    </div>
    <div class="nav-row">
      <button class="nav-btn" data-target="coin-analysis-section">코인 분석</button>
      <button class="nav-btn" data-target="nasdaq-analysis-section">나스닥 분석</button>
      <button class="nav-btn" data-target="pdf-section">PDF 요약</button>
    </div>
    <div class="nav-row">
      <button class="nav-btn" data-target="language-section">외국어 학습</button>
      <button class="nav-btn" data-target="book-section">도서 추천</button>
      <button class="nav-btn" data-target="talk-ai-section">AI Chat</button>
    </div>
  </div>
</header>
```

**특징:**
- ✅ 시맨틱 태그 사용 (`<header>`)
- ✅ `data-target` 속성으로 섹션 연결
- ✅ 4x3 그리드 레이아웃
- ❌ 하드코딩된 네비게이션 버튼
- ❌ 재사용성 부족

### 3.3 메인 섹션 구조 (246줄)

#### 3.3.1 HOME 섹션

```html
<section id="home" class="content-section active">
  <h2>Welcome to arabangoo.com</h2>
  <p>이곳에서 다양한 AI 기반 서비스를 경험해보세요!</p>
</section>
```

**특징:**
- ✅ 기본 활성화 (`active` 클래스)
- ✅ 간결한 구조

#### 3.3.2 PDF 요약 섹션

```html
<section id="pdf-section" class="content-section">
  <div class="service-section">
    <h2>PDF 요약 AI 서비스</h2>
    <form id="uploadForm">
      <input type="file" id="pdfInput" accept="application/pdf"/>
      <button type="submit">업로드하기</button>
    </form>
    <div id="status"></div>
    <div id="summary"></div>
  </div>
</section>
```

**특징:**
- ✅ 파일 업로드 폼
- ✅ 상태 표시 영역 (`#status`)
- ✅ 결과 표시 영역 (`#summary`)
- ❌ 로딩 인디케이터 없음
- ❌ 에러 처리 UI 부족

**JavaScript 연동:**
```javascript
document.getElementById('uploadForm')
  .addEventListener('submit', async (e) => {
    // 90줄의 업로드 및 요약 로직
  });
```

#### 3.3.3 맛집 추천 섹션

```html
<section id="restaurant-section" class="content-section">
  <div class="service-section">
    <h2>맛집 추천 AI 서비스</h2>
    <p class="recommendation-info">(위치 기반 추천)</p>
    <div id="restaurant-buttons">
      <button class="category-btn" data-category="한식">한식</button>
      <button class="category-btn" data-category="양식">양식</button>
      <button class="category-btn" data-category="중식">중식</button>
      <button class="category-btn" data-category="일식">일식</button>
    </div>
    <div id="restaurant-results"></div>
  </div>
</section>
```

**특징:**
- ✅ 카테고리 버튼 그룹
- ✅ `data-category` 속성 활용
- ✅ 결과 표시 영역
- ❌ 하드코딩된 카테고리 버튼
- ❌ 위치 정보 로딩 상태 표시 없음

**JavaScript 연동:**
```javascript
document.querySelectorAll('#restaurant-buttons .category-btn')
  .forEach(button => {
    button.addEventListener('click', async () => {
      // 70줄의 맛집 추천 로직
    });
  });
```

#### 3.3.4 영화 추천 섹션 (가장 복잡한 섹션)

```html
<section id="movie-section" class="content-section">
  <div class="service-section">
    <h2>영화 추천 AI 서비스</h2>
    <p class="movie-info">(3년 기준 추천)</p>
    
    <!-- 플랫폼 선택 -->
    <div id="platform-buttons">
      <button class="platform-btn" data-platform="netflix">넷플릭스</button>
      <button class="platform-btn" data-platform="disneyplus">디즈니플러스</button>
      <button class="platform-btn" data-platform="coupangplay">쿠팡플레이</button>
    </div>
    
    <!-- 장르 선택 (3개 행) -->
    <div id="genre-buttons">
      <div class="genre-row">
        <button class="genre-btn" data-genre="28">액션</button>
        <button class="genre-btn" data-genre="12">모험</button>
        <button class="genre-btn" data-genre="35">코미디</button>
        <button class="genre-btn" data-genre="27">공포</button>
      </div>
      <div class="genre-row">
        <button class="genre-btn" data-genre="14">판타지</button>
        <button class="genre-btn" data-genre="10749">로맨스</button>
        <button class="genre-btn" data-genre="9648">미스테리</button>
        <button class="genre-btn" data-genre="878">SF</button>
      </div>
      <div class="genre-row">
        <button class="genre-btn" data-genre="10752">전쟁</button>
        <button class="genre-btn" data-genre="16">애니메이션</button>
        <button class="genre-btn" data-genre="99">다큐멘터리</button>
        <button class="genre-btn" data-genre="53">스릴러</button>
      </div>
    </div>
    
    <!-- 추천 타입 선택 -->
    <div id="recommendation-buttons">
      <div class="recommendation-row">
        <button id="latest-recommendation-btn">최신 명작 추천</button>
      </div>
      <div class="recommendation-row">
        <button id="classic-recommendation-btn">고전 명작 추천</button>
      </div>
    </div>
    
    <div id="movie-results"></div>
  </div>
</section>
```

**특징:**
- ✅ 3단계 선택 프로세스 (플랫폼 → 장르 → 타입)
- ✅ TMDb API의 장르 ID 활용 (`data-genre="28"`)
- ✅ 12개 장르 지원
- ❌ 복잡한 상태 관리 필요
- ❌ 버튼이 많아 모바일 UI 복잡

**JavaScript 연동:**
```javascript
// 플랫폼 선택
document.querySelectorAll('.platform-btn')
  .forEach(button => { /* ... */ });

// 장르 선택
document.querySelectorAll('.genre-btn')
  .forEach(button => { /* ... */ });

// 최신 명작 추천
document.getElementById('latest-recommendation-btn')
  .addEventListener('click', async () => { /* ... */ });

// 고전 명작 추천
document.getElementById('classic-recommendation-btn')
  .addEventListener('click', async () => { /* ... */ });
```

#### 3.3.5 외국어 학습 섹션

```html
<section id="language-section" class="content-section">
  <div class="service-section">
    <h2>외국어 학습 AI 서비스</h2>
    <p class="update-info">(3시간 주기 갱신)</p>
    <div>
      <button class="language-btn" data-language="english">영어</button>
      <button class="language-btn" data-language="japanese">일본어</button>
      <button class="language-btn" data-language="chinese">중국어</button>
    </div>
    <div id="language-results"></div>
  </div>
</section>
```

**특징:**
- ✅ 간단한 3개 버튼 구조
- ✅ 언어 코드 활용 (`data-language`)
- ✅ 주기적 갱신 정보 제공

#### 3.3.6 코인 분석 섹션

```html
<section id="coin-analysis-section" class="content-section">
  <div class="service-section">
    <h2>코인 분석 AI 서비스</h2>
    <p class="upbit-info">(업비트 기준 분석)</p>
    <div id="coin-analysis-buttons">
      <button class="category-btn" data-range="first_5">거래대금 1위 ~ 5위</button>
      <button class="category-btn" data-range="last_5">거래대금 6위 ~ 10위</button>
    </div>
    <div id="coin-analysis-results"></div>
  </div>
</section>
```

**특징:**
- ✅ 거래대금 기준 순위 분석
- ✅ 2개 그룹 (상위 5개 / 6-10위)
- ✅ Upbit API 연동

#### 3.3.7 논문 추천 섹션

```html
<section id="paper-recommendation-section" class="content-section">
  <div class="service-section">
    <h2>논문 추천 AI 서비스</h2>
    <p class="recommendation-info">(2년 기준 추천)</p>
    
    <!-- 주제 선택 -->
    <div id="topic-buttons">
      <button class="topic-btn" data-topic="AI">AI</button>
      <button class="topic-btn" data-topic="ROBOT">ROBOT</button>
      <button class="topic-btn" data-topic="QUANTUM">QUANTUM</button>
      <button class="topic-btn" data-topic="BLOCKCHAIN">BLOCKCHAIN</button>
    </div>
    
    <!-- 타입 선택 -->
    <div id="paper-type-buttons">
      <button class="paper-type-btn" data-type="latest">최신 논문</button>
      <button class="paper-type-btn" data-type="classic">고전 논문</button>
    </div>
    
    <!-- 검색 버튼 -->
    <div class="search-button-container">
      <button id="search-paper-btn">논문 추천 시작</button>
    </div>
    
    <div id="paper-results"></div>
  </div>
</section>
```

**특징:**
- ✅ 2단계 선택 + 검색 버튼
- ✅ 4개 주제 지원
- ✅ arXiv API 연동
- ❌ 주제와 타입 선택 후 별도 버튼 클릭 필요 (UX 개선 여지)

#### 3.3.8 AI Chat 섹션 (링크 모음)

```html
<section id="talk-ai-section" class="content-section">
  <div class="service-section">
    <h2>AI Chat</h2>
    <div class="ai-links">
      <!-- 8개의 GPTs 링크 -->
      <div class="ai-service">
        <h3>[Cloud AI 무료 컨설팅 GPTs]</h3>
        <p>AWS / Azure / GCP / Nvidia</p>
        <a href="https://bit.ly/cloud-ai-consulting" target="_blank">
          https://bit.ly/cloud-ai-consulting
        </a>
      </div>
      <!-- ... 나머지 7개 서비스 ... -->
    </div>
  </div>
</section>
```

**특징:**
- ✅ 외부 GPTs 링크 제공
- ✅ 8개 전문 컨설팅 서비스
- ✅ 카드 형식의 깔끔한 레이아웃
- ✅ `target="_blank"`로 새 탭 열기

### 3.4 푸터 구조 (22줄)

```html
<footer>
  <!-- 소셜 링크 -->
  <p>GitHub - <a href="https://github.com/arabangoo" target="_blank">
    https://github.com/arabangoo
  </a></p>
  <p>지피터스(GPTers) - <a href="https://www.gpters.org" target="_blank">
    https://www.gpters.org
  </a></p>
  <p>한국인공지능진흥협회(KAIPA) - 
    <a href="https://huggingface.co/openfree" target="_blank">
      https://huggingface.co/openfree
    </a>
  </p>
  
  <!-- 광고 배너 (웹용) -->
  <div id="clickmon-web-banner" class="ad-banner">
    <div class="ad-wrapper">
      <script type="text/javascript">/* 광고 스크립트 */</script>
    </div>
  </div>

  <!-- 광고 배너 (모바일용) -->
  <div id="clickmon-mobile-banner" class="ad-banner">
    <div class="ad-wrapper">
      <script type="text/javascript">/* 광고 스크립트 */</script>
    </div>
  </div>
</footer>
```

**특징:**
- ✅ 소셜 링크 제공
- ✅ 반응형 광고 배너 (웹/모바일)
- ✅ `fixed` 포지셔닝으로 하단 고정
- ❌ 광고 스크립트로 인한 보안 리스크

---

## 4. CSS 스타일 상세 분석

### 4.1 전역 스타일

```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    background: #f5f5f5;
    color: #333;
}
```

**특징:**
- ✅ 브라우저 기본 마진 제거
- ✅ 밝은 배경색 (#f5f5f5)
- ❌ 폰트 폴백 제한적 (Arial만)
- ❌ CSS 변수 미사용

### 4.2 헤더 스타일 (58줄)

```css
header {
    width: 100%;
    background-color: #333;
    color: white;
    text-align: center;
    padding: 20px 0;
}

h1 {
    margin: 0;
    font-size: 2.5rem;
}

/* 네비게이션 버튼 컨테이너 */
.nav-buttons {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
}

.nav-row {
    display: flex;
    justify-content: center;
    gap: 10px;
}

/* 네비게이션 버튼 */
.nav-btn {
    padding: 8px 15px;
    font-size: 1rem;
    color: white;
    background-color: #4CAF50;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

.nav-btn:hover {
    background-color: #45a049;
}

.nav-btn.active {
    background-color: #2e7d32;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}
```

**특징:**
- ✅ Flexbox 레이아웃
- ✅ 호버 효과 (transition)
- ✅ 활성 상태 표시
- ✅ 반응형 gap 활용
- ❌ 색상 하드코딩 (#4CAF50, #2e7d32 등)

### 4.3 메인 컨텐츠 스타일

```css
main {
    width: 90%;
    max-width: 800px;
    margin: 10px auto;
    margin-bottom: 60px;
    padding: 10px;
    box-sizing: border-box;
}

/* 섹션 표시/숨김 */
.content-section {
    display: none;
}

.content-section.active {
    display: block;
}
```

**특징:**
- ✅ 중앙 정렬 (margin: auto)
- ✅ 최대 너비 제한 (800px)
- ✅ 간단한 표시/숨김 로직
- ❌ 애니메이션 효과 없음

### 4.4 버튼 스타일 시스템

```css
/* 카테고리 버튼 (공통) */
.category-btn {
    padding: 8px 12px;
    font-size: 1rem;
    color: #000;
    background-color: #f0f0f0;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
    cursor: pointer;
    transition: all 0.3s ease;
}

.category-btn:hover {
    background-color: #e0e0e0;
}

.category-btn.selected {
    background-color: #4CAF50;
    color: white;
    border-color: #4CAF50;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}
```

**특징:**
- ✅ 일관된 버튼 스타일
- ✅ 선택 상태 시각화
- ✅ 부드러운 트랜지션
- ✅ 접근성 고려 (커서 포인터)
- ❌ 중복 스타일 (platform-btn, genre-btn 등이 유사)

### 4.5 결과 표시 영역 스타일

```css
#summary, #restaurant-results, #movie-results, 
#language-results, #coin-analysis-results, 
#paper-results, #nasdaq-analysis-results, 
#it-news-results, #bestseller-results, 
#place-results {
    background: #fff;
    padding: 15px;
    border-radius: 5px;
    white-space: pre-wrap;
    line-height: 1.4;
    border: 1px solid #ddd;
    margin-top: 10px;
    color: #333;
    font-size: 0.9rem;
}
```

**특징:**
- ✅ 일관된 결과 표시 스타일
- ✅ `white-space: pre-wrap`으로 줄바꿈 유지
- ✅ 카드 형식 디자인
- ❌ 10개 ID를 하나로 그룹핑 (유지보수 어려움)

### 4.6 반응형 디자인 (112줄)

```css
@media (max-width: 768px) {
    h1 {
        font-size: 2rem;
    }

    h2 {
        font-size: 1.3rem;
    }

    button, .category-btn {
        font-size: 0.9rem;
        padding: 8px 10px;
    }

    .platform-btn, .genre-btn {
        min-width: 70px;
        font-size: 0.8rem;
    }

    main {
        width: 95%;
    }

    /* 웹 광고 숨김, 모바일 광고 표시 */
    #clickmon-web-banner {
        display: none;
    }
    
    #clickmon-mobile-banner {
        display: block;
    }
}
```

**특징:**
- ✅ 모바일 최적화
- ✅ 글자 크기 조정
- ✅ 버튼 크기 조정
- ✅ 광고 배너 교체
- ❌ 단일 브레이크포인트 (768px)만 사용

### 4.7 광고 배너 스타일

```css
.ad-banner {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    max-width: 100%;
    z-index: 1000;
    background-color: #fff;
    text-align: center;
    padding: 10px 0;
    box-shadow: 0 -2px 5px rgba(0, 0, 0, 0.1);
}

.ad-banner .ad-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    overflow-x: hidden;
}
```

**특징:**
- ✅ 하단 고정 (position: fixed)
- ✅ 높은 z-index (1000)
- ✅ 그림자 효과
- ❌ 컨텐츠 가림 문제 (margin-bottom: 60px 필요)

---

## 5. JavaScript 로직 상세 분석

### 5.1 초기화 코드 (22줄)

```javascript
document.addEventListener('DOMContentLoaded', function() {
    // 모든 네비게이션 버튼 이벤트 설정
    const navButtons = document.querySelectorAll('.nav-btn');
    navButtons.forEach(button => {
        button.addEventListener('click', () => {
            // 모든 버튼에서 active 클래스 제거
            navButtons.forEach(btn => btn.classList.remove('active'));
            // 클릭된 버튼에 active 클래스 추가
            button.classList.add('active');
            
            // 모든 섹션 숨기기
            const sections = document.querySelectorAll('.content-section');
            sections.forEach(section => section.classList.remove('active'));
            
            // 선택된 섹션 보이기
            const targetId = button.getAttribute('data-target');
            document.getElementById(targetId).classList.add('active');
        });
    });

    // 페이지 로드시 HOME 버튼을 활성화 상태로 설정
    document.getElementById('home-btn').classList.add('active');
});
```

**특징:**
- ✅ DOMContentLoaded 이벤트 활용
- ✅ 이벤트 위임 패턴 사용
- ✅ 명확한 섹션 전환 로직
- ❌ 전역 스코프에 이벤트 리스너 (메모리 관리 문제)
- ❌ SPA처럼 동작하지만 히스토리 관리 없음

### 5.2 위치 정보 유틸리티 (14줄)

```javascript
function getCurrentPosition() {
    return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
            reject(new Error('위치 정보가 지원되지 않습니다.'));
        }

        navigator.geolocation.getCurrentPosition(resolve, reject, {
            enableHighAccuracy: true,
            timeout: 5000,
            maximumAge: 0
        });
    });
}
```

**특징:**
- ✅ Promise 기반 비동기 처리
- ✅ 에러 처리 포함
- ✅ 고정밀 위치 요청
- ✅ 타임아웃 설정 (5초)
- ✅ 재사용 가능한 유틸리티 함수

### 5.3 PDF 업로드 및 요약 (90줄)

```javascript
document.getElementById('uploadForm').addEventListener('submit', async (e) => {
    e.preventDefault();
    const file = document.getElementById('pdfInput').files[0];
    if (!file) {
        document.getElementById('status').textContent = 'PDF 파일을 선택해주세요.';
        return;
    }

    const cloudfrontUrl = `https://arabangoo.com/ai-pdf-folder/${encodeURIComponent(file.name)}`;
    const summaryUrl = `https://arabangoo.com/ai-pdf-foldersummaries/ai-pdf-folder/${encodeURIComponent(file.name.replace('.pdf', '_summary.txt'))}`;

    try {
        document.getElementById('status').textContent = '파일 업로드 중...';

        // CloudFront로 파일 업로드
        const uploadResponse = await fetch(cloudfrontUrl, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/pdf',
                'x-amz-acl': 'bucket-owner-full-control'
            },
            body: file
        });

        if (!uploadResponse.ok) {
            throw new Error(`업로드 실패. 상태 코드: ${uploadResponse.status}`);
        }

        document.getElementById('status').textContent = '업로드 성공! 요약 파일 생성 중...';

        // 요약 파일 확인 (재시도 로직)
        const checkSummary = async (retryCount = 0) => {
            const maxRetries = 30;
            const retryInterval = 5000;

            try {
                const response = await fetch(summaryUrl, { method: 'GET' });

                if (response.ok) {
                    const summaryText = await response.text();
                    document.getElementById('summary').textContent = summaryText;
                    document.getElementById('status').textContent = '요약 완료!';
                } else if (response.status === 404 && retryCount < maxRetries) {
                    setTimeout(() => checkSummary(retryCount + 1), retryInterval);
                } else {
                    throw new Error(`요약 파일 로드 실패. 상태 코드: ${response.status}`);
                }
            } catch (error) {
                if (retryCount < maxRetries) {
                    console.log(`요약 파일 재시도 중 (${retryCount + 1}/${maxRetries})...`);
                    setTimeout(() => checkSummary(retryCount + 1), retryInterval);
                } else {
                    console.error('요약 파일 로드 중 오류 발생:', error);
                    document.getElementById('status').textContent = '요약 파일 로드 실패!';
                }
            }
        };

        checkSummary();
    } catch (error) {
        console.error('업로드 중 오류 발생:', error);
        document.getElementById('status').textContent = '에러 발생: ' + error.message;
    }
});
```

**특징:**
- ✅ Async/Await 패턴
- ✅ 재시도 로직 (최대 30회, 5초 간격)
- ✅ 상태 메시지 업데이트
- ✅ 에러 처리
- ❌ 재귀 함수 사용 (스택 오버플로우 리스크)
- ❌ 파일 크기 검증 없음

### 5.4 맛집 추천 로직 (70줄)

```javascript
const RESTAURANT_API_BASE_URL = 'https://cg5dfxejik.execute-api.ap-northeast-2.amazonaws.com/restaurant';

document.querySelectorAll('#restaurant-buttons .category-btn').forEach(button => {
    button.addEventListener('click', async () => {
        // 버튼 선택 상태 변경
        document.querySelectorAll('#restaurant-buttons .category-btn')
            .forEach(btn => btn.classList.remove('selected'));
        button.classList.add('selected');

        const category = button.getAttribute('data-category');
        const resultArea = document.getElementById('restaurant-results');
        resultArea.textContent = '위치 정보를 확인하는 중입니다...';

        try {
            // 위치 정보 가져오기
            const position = await getCurrentPosition();
            const { latitude, longitude } = position.coords;

            resultArea.textContent = '맛집 추천 데이터를 가져오는 중입니다...';

            // API 호출
            const response = await fetch(`${RESTAURANT_API_BASE_URL}/restaurant`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    category,
                    latitude,
                    longitude
                }),
            });

            if (response.ok) {
                const data = await response.json();
                const formattedRecommendations = data.recommendations
                    .split('\n')
                    .filter(line => line.trim() !== "")
                    .map(line => `<p>${line.trim()}</p>`).join('');

                resultArea.innerHTML = `
                    <div class="recommendations-container">
                        <h3>현재 위치 기준 추천 맛집</h3>
                        ${formattedRecommendations}
                    </div>
                `;
            } else {
                const errorData = await response.json();
                resultArea.textContent = `에러 발생: ${errorData.message}`;
            }
        } catch (error) {
            console.error('맛집 추천 오류:', error);
            if (error.message.includes('위치 정보')) {
                resultArea.textContent = '위치 정보 접근이 거부되었습니다. 위치 정보 접근을 허용해주세요.';
            } else {
                resultArea.textContent = `서버와 통신 중 오류가 발생했습니다: ${error.message}`;
            }
        }
    });
});
```

**특징:**
- ✅ 위치 기반 추천
- ✅ 로딩 상태 표시
- ✅ 에러 처리
- ✅ 사용자 친화적 에러 메시지
- ❌ innerHTML 사용 (XSS 위험)
- ❌ 반복되는 패턴 (다른 섹션과 유사)

### 5.5 영화 추천 로직 (115줄)

```javascript
const MOVIE_API_BASE_URL = 'https://lqwesvse2f.execute-api.ap-northeast-2.amazonaws.com/movie/movie-recommendation';

// 플랫폼 버튼 선택
document.querySelectorAll('.platform-btn').forEach(button => {
    button.addEventListener('click', () => {
        document.querySelectorAll('.platform-btn').forEach(btn => btn.classList.remove('selected'));
        button.classList.add('selected');
    });
});

// 장르 버튼 선택
document.querySelectorAll('.genre-btn').forEach(button => {
    button.addEventListener('click', () => {
        document.querySelectorAll('.genre-btn').forEach(btn => btn.classList.remove('selected'));
        button.classList.add('selected');
    });
});

// 최신 명작 추천 버튼
document.getElementById('latest-recommendation-btn').addEventListener('click', async () => {
    setRecommendationButtonState('latest-recommendation-btn');
    await fetchMovies("latest");
});

// 고전 명작 추천 버튼
document.getElementById('classic-recommendation-btn').addEventListener('click', async () => {
    setRecommendationButtonState('classic-recommendation-btn');
    await fetchMovies("classic");
});

// 추천 버튼 선택 상태 관리
function setRecommendationButtonState(activeButtonId) {
    document.querySelectorAll('#recommendation-buttons button').forEach(button => {
        button.classList.remove('selected');
    });
    document.getElementById(activeButtonId).classList.add('selected');
}

// 영화 추천 데이터 요청 함수
async function fetchMovies(type) {
    const selectedPlatform = document.querySelector('.platform-btn.selected')?.getAttribute('data-platform');
    const selectedGenre = document.querySelector('.genre-btn.selected')?.getAttribute('data-genre');
    const resultArea = document.getElementById('movie-results');

    if (!selectedPlatform || !selectedGenre) {
        resultArea.textContent = '플랫폼과 장르를 모두 선택해주세요.';
        return;
    }

    resultArea.textContent = `${type === "latest" ? "최신 명작" : "고전 명작"} 데이터를 가져오는 중입니다...`;

    try {
        const response = await fetch(MOVIE_API_BASE_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                platform: selectedPlatform,
                genre_id: selectedGenre,
                type: type,
            }),
        });

        if (response.ok) {
            const data = await response.json();
            resultArea.innerHTML = `
                <h3>${type === "latest" ? "최신 명작" : "고전 명작"} 추천 결과</h3>
                <div>${data.recommendations.split('\n').map(line => `<p>${line.trim()}</p>`).join('')}</div>
            `;
        } else {
            const errorData = await response.json();
            resultArea.textContent = `에러 발생: ${errorData.message}`;
        }
    } catch (error) {
        console.error(`${type === "latest" ? "최신 명작" : "고전 명작"} 추천 오류:`, error);
        resultArea.textContent = `서버와 통신 중 오류가 발생했습니다: ${error.message}`;
    }
}
```

**특징:**
- ✅ 3단계 선택 로직
- ✅ 유효성 검사 (플랫폼, 장르 선택 확인)
- ✅ 함수 분리 (`fetchMovies`, `setRecommendationButtonState`)
- ✅ Optional chaining (`?.`)
- ❌ 중복 코드 (버튼 선택 로직)
- ❌ 전역 함수 선언

### 5.6 코드 패턴 분석

**공통 패턴:**

```javascript
// 1. API URL 상수 선언
const API_BASE_URL = 'https://...';

// 2. 버튼 이벤트 리스너 등록
document.querySelectorAll('.some-btn').forEach(button => {
    button.addEventListener('click', async () => {
        // 3. 버튼 선택 상태 변경
        document.querySelectorAll('.some-btn')
            .forEach(btn => btn.classList.remove('selected'));
        button.classList.add('selected');

        // 4. 로딩 상태 표시
        resultArea.textContent = '데이터를 가져오는 중입니다...';

        try {
            // 5. API 호출
            const response = await fetch(API_BASE_URL, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ /* 데이터 */ })
            });

            if (response.ok) {
                // 6. 성공 처리
                const data = await response.json();
                resultArea.innerHTML = /* 결과 표시 */;
            } else {
                // 7. 에러 처리
                const errorData = await response.json();
                resultArea.textContent = `에러 발생: ${errorData.message}`;
            }
        } catch (error) {
            // 8. 예외 처리
            resultArea.textContent = `서버와 통신 중 오류가 발생했습니다: ${error.message}`;
        }
    });
});
```

**이 패턴의 특징:**
- ✅ 일관된 구조
- ✅ 에러 처리 포함
- ❌ 중복 코드 (12개 섹션에서 반복)
- ❌ 재사용 불가능

---

## 6. API 통신 패턴 분석

### 6.1 사용된 API 목록

| API | 엔드포인트 | 메소드 | 용도 |
|-----|-----------|--------|------|
| **PDF 요약** | CloudFront (PUT/GET) | PUT, GET | PDF 업로드 및 요약 |
| **맛집 추천** | .../restaurant/restaurant | POST | Google Maps 기반 추천 |
| **영화 추천** | .../movie/movie-recommendation | POST | TMDb 기반 추천 |
| **외국어 학습** | .../language/genai_foreign_language | POST | AI 문장 생성 |
| **코인 분석** | .../coin/genai-coin-analisys | POST | Upbit 데이터 분석 |
| **논문 추천** | .../paper/genai-arxiv-papaer | POST | arXiv 검색 |
| **나스닥 분석** | .../nasdaq/genai-nasdaq-analisys | POST | Yahoo Finance 분석 |
| **IT 뉴스** | .../news/genai-news-api | POST | Google News 요약 |
| **도서 추천** | .../book/genai-book-api | POST | 알라딘 베스트셀러 |
| **주변 시설** | .../place/genai-place-api | POST | Google Maps 검색 |

### 6.2 요청 패턴

**일반적인 POST 요청:**

```javascript
const response = await fetch(API_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        // 요청 데이터
    })
});
```

**특이 케이스 - PDF 업로드 (PUT):**

```javascript
const response = await fetch(cloudfrontUrl, {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/pdf',
        'x-amz-acl': 'bucket-owner-full-control'
    },
    body: file  // File 객체 직접 전송
});
```

### 6.3 응답 처리 패턴

**성공 응답:**

```javascript
if (response.ok) {
    const data = await response.json();
    // data.recommendations 또는 data.analysis 사용
}
```

**에러 응답:**

```javascript
if (!response.ok) {
    const errorData = await response.json();
    resultArea.textContent = `에러 발생: ${errorData.message}`;
}
```

### 6.4 API 응답 구조

**공통 응답 형식:**

```json
{
  "recommendations": "AI가 생성한 추천 텍스트...",
  "message": "성공 메시지" (선택적)
}
```

또는

```json
{
  "analysis": "AI 분석 결과...",
  "message": "성공 메시지" (선택적)
}
```

**논문 추천 응답 (복잡한 구조):**

```json
{
  "papers": [
    {
      "title": "논문 제목",
      "authors": "저자명",
      "published_date": "2024-01-01",
      "summary": "논문 요약",
      "link": "https://arxiv.org/..."
    }
  ],
  "recommendations": "AI 추천 이유...",
  "source": "출처 정보"
}
```

---

## 7. 사용자 경험 (UX) 분석

### 7.1 장점

1. **직관적인 네비게이션**
   - ✅ 헤더 고정 (항상 접근 가능)
   - ✅ 버튼 색상으로 상태 표시
   - ✅ 12개 서비스를 4x3 그리드로 정리

2. **즉각적인 피드백**
   - ✅ 로딩 상태 표시
   - ✅ 에러 메시지 표시
   - ✅ 버튼 선택 시 시각적 변화

3. **반응형 디자인**
   - ✅ 모바일 최적화
   - ✅ 광고 배너 자동 교체

### 7.2 단점

1. **로딩 인디케이터 부족**
   - ❌ 스피너나 프로그레스 바 없음
   - ❌ 텍스트만으로 로딩 표시

2. **섹션 전환 애니메이션 없음**
   - ❌ 즉시 표시/숨김 (갑작스러움)
   - ❌ 부드러운 전환 효과 부족

3. **히스토리 관리 없음**
   - ❌ 뒤로가기 버튼 작동 안 함
   - ❌ URL 변경 없음 (북마크 불가)

4. **PDF 업로드 UX**
   - ❌ 파일 크기 제한 안내 없음
   - ❌ 진행률 표시 없음
   - ❌ 재시도 30회는 사용자에게 너무 오래 기다림

5. **영화 추천 UX**
   - ❌ 3단계 선택 필요 (플랫폼 → 장르 → 타입)
   - ❌ 초기 선택 상태 없음 (무엇을 먼저 선택해야 할지 모호)

---

## 8. 보안 및 성능 분석

### 8.1 보안 이슈

#### 8.1.1 XSS (Cross-Site Scripting) 취약점

```javascript
// innerHTML 사용으로 인한 XSS 위험
resultArea.innerHTML = `
    <div class="recommendations-container">
        <h3>현재 위치 기준 추천 맛집</h3>
        ${formattedRecommendations}  // 사용자 입력이 포함될 수 있음
    </div>
`;
```

**문제점:**
- ❌ 서버 응답에 악성 스크립트 포함 시 실행 가능
- ❌ DOMPurify 같은 sanitization 라이브러리 미사용

**해결 방안:**
```javascript
// textContent 사용 또는 DOMPurify 적용
resultArea.textContent = data.recommendations;  // 안전
// 또는
import DOMPurify from 'dompurify';
resultArea.innerHTML = DOMPurify.sanitize(html);
```

#### 8.1.2 광고 스크립트 보안

```html
<!-- 외부 광고 스크립트 직접 삽입 -->
<script type="text/javascript">
  (function(cl, i, c, k, m, o, n) {
    /* 광고 스크립트 */
  })(document, 'script', 'https://tab2.clickmon.co.kr/pop/wp_ad_728_js.php?...', ...);
</script>
```

**문제점:**
- ❌ 외부 스크립트 직접 실행
- ❌ 스크립트 무결성 검증 없음 (SRI 미사용)

#### 8.1.3 API 키 노출

**현재 상태:**
- ✅ API Gateway URL만 노출 (키는 서버 측)
- ✅ Lambda 함수에서 API 키 관리

**권장 사항:**
- ✅ API Gateway에 IAM 인증 추가
- ✅ CloudFront 서명된 URL 사용

### 8.2 성능 이슈

#### 8.2.1 불필요한 DOM 조작

```javascript
// 매번 모든 버튼 순회
document.querySelectorAll('.platform-btn')
    .forEach(btn => btn.classList.remove('selected'));
```

**문제점:**
- ❌ 12개 섹션마다 반복
- ❌ 이벤트 리스너 중복 등록

**해결 방안:**
```javascript
// 이벤트 위임 패턴
document.querySelector('.platform-buttons')
    .addEventListener('click', (e) => {
        if (e.target.classList.contains('platform-btn')) {
            // 처리
        }
    });
```

#### 8.2.2 이미지 최적화 부족

**현재 상태:**
- ❌ 이미지 lazy loading 없음
- ❌ WebP 포맷 미사용

#### 8.2.3 JavaScript 번들링 없음

**현재 상태:**
- ❌ 624줄이 하나의 파일
- ❌ 코드 스플리팅 없음
- ❌ Tree shaking 없음

#### 8.2.4 캐싱 전략

**현재 상태:**
- ✅ CloudFront CDN 사용
- ❌ Service Worker 없음
- ❌ 오프라인 지원 없음

---

## 9. 문제점 및 개선 방향

### 9.1 구조적 문제점

| 문제점 | 영향 | 심각도 |
|--------|------|--------|
| **평면적 구조** | 유지보수 어려움 | 🔴 높음 |
| **코드 중복** | 버그 증가, 수정 비용 증가 | 🔴 높음 |
| **전역 스코프 오염** | 네임스페이스 충돌 | 🟡 중간 |
| **재사용 불가능** | 다른 프로젝트 적용 어려움 | 🟡 중간 |

### 9.2 기능적 문제점

| 문제점 | 영향 | 심각도 |
|--------|------|--------|
| **XSS 취약점** | 보안 위험 | 🔴 높음 |
| **히스토리 관리 없음** | SEO 불리, UX 저하 | 🟡 중간 |
| **에러 처리 불완전** | 사용자 혼란 | 🟡 중간 |
| **로딩 상태 부족** | UX 저하 | 🟢 낮음 |

### 9.3 개선 방향

#### 9.3.1 React 마이그레이션 (권장)

**장점:**
- ✅ 컴포넌트 재사용
- ✅ 상태 관리 명확화
- ✅ Virtual DOM 성능 최적화
- ✅ 풍부한 생태계

**마이그레이션 전략:**
1. 공통 패턴 추출 (커스텀 훅)
2. 섹션별 컴포넌트화
3. API 레이어 분리
4. 상태 관리 라이브러리 도입

#### 9.3.2 보안 강화

```javascript
// 1. XSS 방어
import DOMPurify from 'dompurify';
resultArea.innerHTML = DOMPurify.sanitize(html);

// 2. CSP 헤더 추가
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline';

// 3. API 인증
fetch(API_URL, {
    headers: {
        'Authorization': `Bearer ${token}`
    }
});
```

#### 9.3.3 성능 최적화

```javascript
// 1. 이벤트 위임
document.addEventListener('click', (e) => {
    if (e.target.classList.contains('category-btn')) {
        handleCategoryClick(e.target);
    }
});

// 2. Debouncing
const debouncedSearch = debounce(searchFunction, 300);

// 3. Lazy Loading
if ('IntersectionObserver' in window) {
    const observer = new IntersectionObserver(entries => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                loadSection(entry.target);
            }
        });
    });
}
```

#### 9.3.4 UX 개선

```javascript
// 1. 로딩 스피너
<div class="spinner"></div>

// 2. 섹션 전환 애니메이션
.content-section {
    transition: opacity 0.3s ease-in-out;
}

// 3. 히스토리 관리
window.history.pushState({ section: 'movie-section' }, '', '/movie');
```

---

## 10. 결론

### 10.1 현재 상태 요약

**강점:**
- ✅ 다양한 AI 서비스 제공
- ✅ AWS 서버리스 아키텍처 활용
- ✅ 반응형 디자인 구현
- ✅ 일관된 코드 패턴

**약점:**
- ❌ 유지보수성 낮음 
- ❌ 코드 중복 심각
- ❌ XSS 취약점 존재
- ❌ 확장성 부족
