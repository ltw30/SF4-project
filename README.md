# EverNex BMS Smart Factory

배터리 관리 시스템(BMS) 품질검사 대시보드 — Spring Boot 3 + Vue 3 풀스택 프로젝트

https://smartfactory-bms-project.vercel.app/login

---

## 목차

1. [프로젝트 개요](#프로젝트-개요)
2. [주요 기능](#주요-기능)
3. [기술 스택](#기술-스택)
4. [프로젝트 구조](#프로젝트-구조)
5. [로컬 실행](#로컬-실행)
6. [환경 변수](#환경-변수)
7. [API 엔드포인트](#api-엔드포인트)
8. [데이터베이스 스키마](#데이터베이스-스키마)
9. [차량 상태 흐름](#차량-상태-흐름)
10. [LLM 챗봇](#llm-챗봇)
11. [배포](#배포)

---

## 프로젝트 개요

전기차 배터리 품질검사 공정을 시뮬레이션하고 모니터링하는 대시보드 웹 애플리케이션입니다.


---

## 주요 기능

### 실시간 대시보드
- 차량 상태별 통계 (전체 / 검사 중 / 이상 감지 / 출고 완료) — 5초 자동 갱신
- 공정 파이프라인 진행률 시각화 (입고 → 검사 → 출고)
- 최근 미해결 경보 요약
- 공장별 필터링 (복수 선택 가능)

### 차량 관리
- 차량 목록 조회 및 검색 (모델·상태·수출국·날짜 필터, 최대 500건)
- 차량 상세 페이지
  - 배터리 측정값 (SOC, SOH, SOP, 전압, 온도)
  - 배터리 셀 온도 히트맵 시각화
  - 처리 단계 타임라인 (각 검사 시작/종료 시간)
  - 경보 목록 및 상태 변경 이력

### 경보 관리
- 경보 목록 조회 및 필터링 (심각도·타입·모델)
- 경보 상태 변경: 확인(acknowledge) / 해제(resolve) / 일괄 삭제
- 읽지 않은 경보 수 배지 (헤더에 실시간 표시)

### LLM 챗봇
- 우하단 부동형 챗봇 위젯
- **RAG Lite 모드**: 현재 DB 통계를 컨텍스트로 제공 후 LLM이 자연어 답변
- **Text-to-SQL 모드**: 자연어 질문 → LLM이 SQL 자동 생성 → DB 실행 → 결과 해석
- 대화 기반 리포트 자동 생성 및 저장
- LM Studio(로컬) / OpenAI / Google Gemini 지원

### 설정 관리 (관리자 전용)
- 차량 자동 생성 간격 (ms) 조정
- 검사 단계별 소요 시간 (ms) 조정
- 검사 단계별 성공 확률 (%) 조정
- LLM 제공자 / 모델 / API 키 설정

### 사용자 관리 (관리자 전용)
- 사용자 목록 조회
- 역할 변경 (admin / operator)
- 사용자 삭제

### 다국어 & 테마
- 한국어 / 영어 전환 (vue-i18n, 로컬 스토리지에 저장)
- 다크 / 라이트 모드 전환 (Tailwind CSS)

---

## 기술 스택

### 백엔드

| 항목 | 버전 |
|------|------|
| Java | 17 |
| Spring Boot | 3.3.4 |
| Spring MVC + JdbcTemplate | — |
| MySQL Connector/J | 8.0+ |
| JJWT | 0.12.6 |
| Spring Security Crypto (BCrypt) | — |
| Maven | 3.9+ |

### 프론트엔드

| 항목 | 버전 |
|------|------|
| Vue | 3.4.21 |
| Vue Router | 4.3.2 |
| Pinia | 2.2.0 |
| Vue i18n | 9.14.5 |
| Vite | 5.2.8 |
| Tailwind CSS | 3.4.3 |

### 인프라

| 항목 | 서비스 |
|------|--------|
| 프론트엔드 배포 | Vercel Hobby |
| 백엔드 배포 | Render Free Web (Docker) |
| 데이터베이스 | MySQL |
| CI/CD | GitHub Actions |

---

## 프로젝트 구조

```
04_BMS_SF_ver02/
├── backend/
│   ├── Dockerfile                   # 멀티스테이지 Docker 빌드
│   ├── pom.xml
│   └── src/main/java/com/evernex/bms/
│       ├── BmsApplication.java
│       ├── config/                  # CORS 설정 (WebConfig)
│       ├── controller/              # REST API 컨트롤러 9개
│       │   ├── AuthController       # 로그인 / 회원가입 / 내 정보
│       │   ├── DashboardController  # 실시간 통계
│       │   ├── VehiclesController   # 차량 CRUD + 상세 조회
│       │   ├── AlertsController     # 경보 관리
│       │   ├── ChatController       # LLM 챗봇 + 리포트 생성
│       │   ├── ReportsController    # 저장된 리포트 조회/삭제
│       │   ├── SettingsController   # 관리 설정
│       │   ├── UsersController      # 사용자 관리
│       │   └── HealthController     # 헬스 체크
│       ├── service/
│       │   ├── SimulationService    # 500ms 스케줄 상태머신
│       │   ├── VehicleFactoryService # 차량 자동 생성
│       │   ├── ChatService          # RAG Lite / Text-to-SQL 처리
│       │   ├── LlmClient            # OpenAI / Gemini / LM Studio 연동
│       │   ├── SettingsService      # 설정 CRUD
│       │   └── RandomMetricService  # 시뮬레이션용 난수 생성
│       ├── security/
│       │   ├── JwtUtil              # JWT 생성·검증 (HS256)
│       │   ├── JwtAuthFilter        # Spring 필터 체인 인증
│       │   ├── AuthContext          # 스레드로컬 사용자 정보
│       │   ├── AuthPrincipal        # 사용자 권한 모델
│       │   └── GlobalExceptionHandler
│       ├── db/
│       │   ├── SchemaInitializer    # 앱 시작 시 테이블 자동 생성
│       │   ├── DataSeeder           # 샘플 데이터 삽입
│       │   ├── FactoryScope         # 공장 권한 필터링
│       │   └── TimeUtil
│       └── domain/
│           └── Constants            # 차량 상태, 모델명, 공장 상수
│
├── frontend/
│   ├── vercel.json                  # SPA 라우팅 설정
│   ├── src/
│   │   ├── views/                   # 페이지 컴포넌트 9개
│   │   │   ├── Dashboard.vue
│   │   │   ├── VehicleList.vue
│   │   │   ├── VehicleDetail.vue
│   │   │   ├── Alerts.vue
│   │   │   ├── Reports.vue
│   │   │   ├── Settings.vue
│   │   │   ├── Users.vue
│   │   │   ├── Login.vue
│   │   │   └── Signup.vue
│   │   ├── components/
│   │   │   ├── Chatbot.vue          # 부동형 LLM 챗봇 위젯
│   │   │   ├── CellHeatmap.vue      # 배터리 셀 온도 히트맵
│   │   │   ├── Gauge.vue            # SOC/SOH/온도 게이지 차트
│   │   │   ├── ProcessTimeline.vue  # 처리 단계 타임라인
│   │   │   ├── FactoryFilter.vue    # 공장 다중 선택 필터
│   │   │   └── MultiSelect.vue      # 범용 다중 선택 드롭다운
│   │   ├── layouts/
│   │   │   └── AppLayout.vue        # 사이드바 + 헤더 레이아웃
│   │   ├── stores/                  # Pinia 상태 관리
│   │   │   ├── auth.js              # 로그인 / 사용자 정보
│   │   │   ├── filters.js           # 공장 필터 상태
│   │   │   ├── alerts.js            # 미해결 경보 카운트
│   │   │   └── theme.js             # 다크 / 라이트 모드
│   │   ├── composables/
│   │   │   ├── api.js               # HTTP 클라이언트 (JWT 자동 첨부, 401 자동 로그아웃)
│   │   │   ├── status.js            # 상태별 색상·라벨 매핑
│   │   │   └── labels.js            # 모델명·공장·국가 라벨
│   │   ├── router/index.js          # Vue Router (인증 가드)
│   │   └── i18n/
│   │       └── locales/
│   │           ├── ko.js            # 한국어 메시지
│   │           └── en.js            # 영어 메시지
│
├── sql/
│   ├── schema.sql                   # MySQL DDL (프로덕션 마이그레이션 참고용)
│   └── seed.sql                     # 초기 데이터
│
├── .github/workflows/
│   ├── backend-ci.yml               # Maven 빌드 + Docker 빌드 검증
│   └── frontend-ci.yml              # npm ci + Vite 빌드 검증
│
├── render.yaml                      # Render Blueprint 설정
├── DEPLOYMENT.md                    # Vercel + Render 배포 단계별 가이드
└── README.md
```

---

## 로컬 실행

### 사전 요구사항

- Java 17+
- Maven 3.9+
- Node.js 20+
- (선택) LM Studio — 챗봇 기능 로컬 테스트 시

---

## 환경 변수

### 백엔드

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `PORT` | `3000` | HTTP 서버 포트 |
| `DB_URL` | `jdbc:mysql://localhost:3306/bms` | MySQL 연결 URL |
| `DB_USERNAME` | `root`                            | DB 사용자명 |
| `DB_PASSWORD` |  | DB 비밀번호
| `JWT_SECRET` | (개발용 기본값) | JWT 서명 키 — 운영 시 32자+ 랜덤 문자열로 교체 |
| `LLM_BASE_URL` | `http://127.0.0.1:1234` | LM Studio 또는 OpenAI 엔드포인트 |
| `LLM_MODEL` | `local-model` | 사용할 LLM 모델명 |

운영 배포 시 추가로 필요한 변수:

| 변수 | 예시 | 설명 |
|------|------|------|
| `LLM_API_KEY` | `sk-...` | OpenAI / Gemini API 키 (챗봇 사용 시) |

### 프론트엔드 (`.env`)

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `VITE_API_BASE_URL` | `http://localhost:3000/api/v1` | 백엔드 API 베이스 URL |

---

## API 엔드포인트

모든 엔드포인트는 `/api/v1` 접두사를 사용합니다.  
`/auth/login`, `/auth/signup`을 제외한 모든 요청에 `Authorization: Bearer <token>` 헤더가 필요합니다.

에러 응답 형식: `{ "error": "<message>", "detail": "<optional>" }`

### 인증

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/auth/login` | 로그인 → JWT + 사용자 정보 반환 |
| POST | `/auth/signup` | 회원가입 (operator 역할로 생성) |
| GET | `/auth/me` | 현재 로그인 사용자 정보 |

### 대시보드

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/dashboard/stats` | 실시간 차량 상태 집계, 경보 요약 (`?factory_ids=1,2`) |

### 차량

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/vehicles` | 목록 조회 (모델·상태·국가·날짜 필터) |
| POST | `/vehicles` | 차량 수동 생성 |
| GET | `/vehicles/:carId` | 상세 조회 (배터리, 셀, 이력, 경보) |
| PUT | `/vehicles/:carId` | 차량 정보 수정 |
| DELETE | `/vehicles/:carId` | 삭제 |
| POST | `/vehicles/:carId/resolve` | 차량 경보 일괄 해제 |

### 경보

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/alerts` | 목록 조회 (심각도·타입·모델·상태 필터) |
| GET | `/alerts/facets` | 필터 선택지 반환 |
| GET | `/alerts/unresolved-count` | 미해결 경보 수 |
| POST | `/alerts/:id/acknowledge` | 경보 확인 처리 |
| POST | `/alerts/:id/resolve` | 경보 해제 |
| POST | `/alerts/bulk-delete` | 다중 삭제 |

### 챗봇 & 리포트

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/chat` | 자연어 질문 → LLM 답변 (RAG Lite / Text-to-SQL) |
| POST | `/chat/report` | 대화 내용 기반 리포트 생성 및 DB 저장 |
| GET | `/reports` | 저장된 리포트 목록 |
| GET | `/reports/:id` | 리포트 상세 (메시지, SQL, 데이터 포함) |
| DELETE | `/reports/:id` | 리포트 삭제 |

### 설정 & 사용자 (관리자 전용)

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/settings` | 전체 설정 조회 |
| PUT | `/settings/:key` | 설정값 변경 |
| GET | `/settings/factories` | 공장 목록 |
| GET | `/settings/factories/all` | 전체 공장 목록 (비활성 포함) |
| GET | `/settings/countries` | 수출국 목록 |
| PUT | `/settings/countries/:id` | 수출국 허용 여부 변경 |
| GET | `/users` | 사용자 목록 |
| PUT | `/users/:id` | 역할 변경 |
| DELETE | `/users/:id` | 사용자 삭제 |

### 헬스 체크

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/health` | 서버 상태 확인 |

---

## 데이터베이스 스키마

MySQL을 사용하며 sql/schema.sql의 DDL로 테이블을 생성합니다.

### 주요 테이블 (16개)

| 테이블 | 설명 |
|--------|------|
| `users` | 사용자 (user_id, email, password_hash, role, name) |
| `factories` | 6개 공장 정보 (청림·은하·백운·단풍·태양·한빛) |
| `countries` | 수출 대상국 (10개국, 허용 여부 관리) |
| `cars` | 차량 (모델명, 공장, 목적지 국가, 현재 상태) |
| `batteries` | 차량당 1개 배터리 (시리얼, 제조일) |
| `battery_measurements` | 배터리 측정값 (SOC, SOH, SOP, 전압, 온도) |
| `battery_cells` | 배터리 셀 목록 (셀 번호) |
| `battery_cell_measurements` | 셀별 온도·전압 측정값 |
| `car_status_histories` | 차량 상태 변경 이력 |
| `alerts` | 경보 (타입, 심각도, 현재 상태) |
| `alert_status_histories` | 경보 상태 변경 이력 |
| `process_step_histories` | 검사 단계별 진행 기록 (시작/종료 시간) |
| `inspection_results` | 검사 결과 요약 |
| `admin_settings` | 운영 설정 (검사 시간, 확률, LLM 설정 등) |
| `llm_chat_logs` | LLM 대화 기록 |
| `reports` | 생성된 리포트 |



## 차량 상태 흐름

시뮬레이션 서비스(`SimulationService`)가 `@Scheduled(fixedDelay=500)` 으로 500ms마다 상태를 전이시킵니다.

```
ARRIVAL (입고)
    │
    ▼  6단계 검사
BATTERY_INSPECTION ─── 이상 감지 ──► ANOMALY_DETECTED
CELL_INSPECTION                              │
    │ 통과                           QA_MAINTENANCE
    │                                        │
    │                          RE_INSPECTION_WAITING
    │                          RE_INSPECTION
    │◄──────────────────────────────── 재검사 통과
    │
BATTERY_QC_COMPLETE
    │
SHIPMENT_WAITING
    │
SHIPMENT_COMPLETE (출고 완료)
```

### 배터리 검사 6단계

| 단계 | 항목 | 설명 |
|------|------|------|
| 1 | SOC_CHECK | 충전 상태 (State of Charge) |
| 2 | SOH_CHECK | 배터리 건강도 (State of Health) |
| 3 | SOP_CHECK | 전력 상태 (State of Power) |
| 4 | PACK_VOLTAGE_CHECK | 팩 전압 |
| 5 | CELL_TEMPERATURE_CHECK | 셀 온도 |
| 6 | CELL_VOLTAGE_CHECK | 셀 전압 |

각 단계의 소요 시간(ms)과 성공 확률(%)은 Settings 페이지에서 실시간 조정 가능합니다.

---

## LLM 챗봇

### 지원 제공자

| 제공자 | `llm_provider` 값 | 비고 |
|--------|-------------------|------|
| LM Studio (로컬) | `lm_studio` | 인터넷 불필요, 기본값 |
| OpenAI | `openai` | API 키 필요 |
| Google Gemini | `gemini` | API 키 필요 |

### 동작 모드

**RAG Lite** (`rag_lite`)
- DB에서 전체 통계, 미해결 경보, 키워드 기반 차량 목록을 JSON으로 추출
- LLM에 컨텍스트와 함께 질문을 전달하여 자연어 답변 생성

**Text-to-SQL** (`text_to_sql`)
1. 자연어 질문 → LLM이 MySQL SELECT 문 생성
2. SQL 검증 (SELECT만 허용, 테이블 화이트리스트, 공장 권한 필터 자동 삽입)
3. DB 실행 → 결과를 LLM이 다시 자연어로 해석

### 리포트 생성

챗봇 대화창의 **리포트 생성** 버튼 클릭 시:
- 대화 내용 전체를 LLM에 전달하여 구조화된 리포트 생성
- 저장 항목: 제목, 요약, 핵심 발견사항, 권장 조치, 데이터 근거
- Reports 페이지에서 저장된 리포트 조회·삭제 가능

### 챗봇 설정 방법

1. Settings 페이지(관리자) → LLM 섹션에서 제공자/모델/API키 설정
2. 또는 Render 환경 변수로 직접 설정:
   ```
   LLM_BASE_URL=https://api.openai.com
   LLM_MODEL=gpt-4o-mini
   LLM_API_KEY=sk-...
   ```

---

## 배포

Vercel(프론트엔드) + Render(백엔드) 사용

### 빠른 요약

```
1. Render → New → Blueprint → 이 저장소 연결
   → render.yaml 자동 감지 → bms-backend 배포

2. Vercel → New Project → 이 저장소 Import
   → Root Directory: frontend
   → 환경 변수: VITE_API_BASE_URL = https://<render-url>/api/v1
   → Deploy
```

---

## 권한 체계

| 역할 | 접근 범위 |
|------|-----------|
| **admin** | 모든 공장 데이터 + 사용자·설정 관리 |
| **operator** | 할당된 공장 데이터만 조회 가능 (관리 기능 접근 불가) |

---


