# CLAUDE.md

## 프로젝트 개요

주식/자산 포트폴리오와 배당금을 관리하는 웹 애플리케이션.
GitHub Pages로 `index.html` 단일 파일로 서비스한다.

## 기술 스택

- **프론트엔드**: Vanilla HTML/CSS/JavaScript (단일 `index.html` 파일)
- **차트**: Chart.js + chartjs-plugin-datalabels (CDN)
- **폰트**: IBM Plex Sans KR (Google Fonts)
- **인증**: Firebase Authentication (Google 로그인)
- **데이터베이스**: Firebase Firestore (무료 플랜, Spark)
- **배포**: GitHub Pages
- **Firebase SDK**: ES Module 방식으로 CDN에서 직접 import (v11.6.0)

## 프로젝트 구조

```
stock-portfolio-dev/
├── index.html          # 메인 (유일한) 애플리케이션 파일
├── portfolio.json       # 샘플/백업용 포트폴리오 데이터 (gitignore 대상)
├── portfolio.csv        # 샘플/백업용 CSV 데이터 (gitignore 대상)
├── .gitignore           # *.csv, *.json 제외
├── README.md
└── CLAUDE.md
```

## 아키텍처

### 단일 파일 구조
모든 HTML, CSS, JavaScript가 `index.html` 하나에 포함되어 있다. 별도의 빌드 도구나 번들러 없이 브라우저에서 직접 실행된다.

### 스크립트 구성
- **첫 번째 `<script type="module">`**: Firebase 초기화, 인증, Firestore CRUD (ES Module)
- **두 번째 `<script>`**: 일반 스크립트. 포트폴리오 CRUD, 차트, 배당금, 테마, 로컬 저장/불러오기 등 모든 UI 로직
- Firebase 모듈 스코프의 함수는 `window.*` 에 할당하여 HTML onclick에서 접근 가능하게 한다.

### Firestore 데이터 구조
```
dev_users/{uid}/charts/{chartName}
  └── jsonData: string (JSON.stringify된 { items, dividendRecords, exchangeRate })
```

## 주요 기능

### 자산 관리
- 종목 추가/수정/삭제 (이름, 개수, 가격, 통화(KRW/USD), 카테고리, 날짜)
- 환율 입력으로 USD → KRW 자동 환산
- 카테고리별 필터링 (전체 선택/해제, 개별 토글)
- 파이 차트로 카테고리별 자산 비율 시각화

### 배당금 관리
- 배당금 기록 추가/삭제 (모달 UI)
- 보유 종목 이름 자동완성 (datalist)
- 배당금 내역 및 총 배당금 수익 요약 표시

### 데이터 저장/불러오기
- **로컬**: localStorage 저장/불러오기
- **Firebase**: Google 로그인 후 Firestore에 차트 이름으로 저장/불러오기/삭제
- **파일**: JSON/CSV 다운로드 및 업로드
- **Firebase 차트별 내보내기**: 저장된 차트 목록에서 개별 JSON/CSV 다운로드

### UI/테마
- 다크모드/라이트모드 토글 (CSS Custom Properties 기반)
- 테마 설정 localStorage에 자동 저장/복원
- 전환 시 애니메이션 적용 (`transition`)

## 데이터 모델

### Item (자산)
```javascript
{
  name: string,           // 종목 이름
  amount: number,         // 개수 (소수점 가능)
  price: number,          // 가격
  currency: "KRW" | "USD",
  categories: string[],   // 카테고리 목록 (예: ["토스", "배당"])
  date: string            // YYYY-MM-DD
}
```

### DividendRecord (배당금)
```javascript
{
  id: string,             // 고유 ID (Date.now().toString(36))
  stockName: string,      // 종목 이름
  totalAmount: number,    // 배당금액
  currency: "KRW" | "USD",
  paymentDate: string,    // YYYY-MM-DD
  note: string            // 메모 (선택)
}
```

## CSV 포맷

CSV는 두 섹션으로 구분된다. BOM(`\uFEFF`) 포함으로 한글 Excel 호환성을 보장한다.

```
[보유 자산 목록]
이름,개수,가격,통화,카테고리,날짜

[배당금 기록]
종목이름,배당금액,통화,지급일,메모
```

- 카테고리는 `|`로 구분하여 큰따옴표로 감싼다 (예: `"토스|배당|리츠"`)
- 메모는 큰따옴표로 감싸며 내부 `"`는 `""`로 이스케이프한다

## 개발 가이드라인

- 이 프로젝트는 `index.html` 단일 파일로 유지한다. 파일을 분리하지 않는다.
- 외부 라이브러리는 CDN으로만 사용한다. npm/빌드 도구를 도입하지 않는다.
- Firebase config 값은 프론트엔드 전용이므로 코드에 직접 포함되어 있다.
- CSS는 CSS Custom Properties(변수)를 사용하여 다크모드를 지원한다. 새로운 스타일 추가 시 변수를 활용한다.
- 한국어 UI를 기본으로 한다.

## Git 컨벤션

- 커밋 메시지: 한국어 사용
- `.gitignore`: `*.csv`, `*.json` 파일은 커밋하지 않는다 (개인 포트폴리오 데이터)
