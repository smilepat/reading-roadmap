# EFL Reading Ladder — App Development PRD

> **목적:** 개발자가 이 문서만으로 앱을 구축할 수 있는 실행 가능한 명세서
> **버전:** 2.0 | **작성일:** 2026-02-28 | **최종 수정:** 2026-02-28
> **선행 문서:** PRD.md (제품 요구사항), TECH_REVIEW.md (기술 검토), READING_TEXT_DB_PLAN.md (콘텐츠 설계)
> **참조 앱:** 5dimension_vocablearning (동일 패턴 적용)
> **변경 이력:** v2.0 — UX_SIMULATION_REVIEW.md 23개 개선사항 반영 (배치 테스트, 온보딩, 학급/과제, 게이미피케이션, Pre/While/Post-reading, 삽화 모델, 5D 핸드오프, 수능 실전 모드 등)

---

## 목차

1. [프로젝트 초기 설정](#1-프로젝트-초기-설정)
2. [디렉토리 구조](#2-디렉토리-구조)
3. [환경 변수](#3-환경-변수)
4. [데이터베이스 스키마 (Prisma)](#4-데이터베이스-스키마-prisma)
5. [Neo4j 연동 모듈](#5-neo4j-연동-모듈)
6. [API Routes 명세](#6-api-routes-명세)
7. [페이지 라우트 & 화면 명세](#7-페이지-라우트--화면-명세)
8. [컴포넌트 설계](#8-컴포넌트-설계)
9. [Lib 모듈 명세](#9-lib-모듈-명세)
10. [콘텐츠 생성 파이프라인 (CLI)](#10-콘텐츠-생성-파이프라인-cli)
11. [인증 & 미들웨어](#11-인증--미들웨어)
12. [스타일링 & 테마](#12-스타일링--테마)
13. [배포 설정](#13-배포-설정)
14. [개발 순서 (Sprint Plan)](#14-개발-순서-sprint-plan)

---

## 1. 프로젝트 초기 설정

### 1.1 기술 스택

| 영역 | 기술 | 버전 | 비고 |
|------|------|------|------|
| Framework | Next.js (App Router) | 15.x | Turbopack dev |
| Language | TypeScript | 5.x | strict mode |
| Runtime | React | 19 | |
| ORM | Prisma | 7.x | PostgreSQL adapter |
| DB (관계형) | Supabase PostgreSQL | — | 인증 + 스토리지 포함 |
| DB (그래프) | Neo4j | 2.1.1 Desktop | 로컬, bolt://localhost:7687 |
| UI | shadcn/ui + Radix UI | — | 5D 앱과 동일 컴포넌트 세트 |
| Styling | Tailwind CSS | v3 | 5D 앱과 동일 (HSL CSS vars) |
| Charts | Recharts | v3 | 레이더, 라인, 바 차트 |
| AI | Vercel AI SDK + OpenAI | GPT-4o | 콘텐츠 생성 + 문항 생성 |
| TTS | OpenAI TTS-1-HD | — | 음성 파일 생성 |
| NLP | compromise | — | 텍스트 분석 (문장 분리, POS) |
| Icons | lucide-react | — | |
| Forms | react-hook-form + zod | — | |
| Deploy | Vercel | — | |

### 1.2 프로젝트 생성

```bash
npx create-next-app@latest reading-ladder \
  --typescript --tailwind --eslint --app --src-dir=false \
  --import-alias="@/*" --turbopack

cd reading-ladder

# Prisma (PostgreSQL)
npm install prisma @prisma/client
npx prisma init --datasource-provider postgresql

# Supabase
npm install @supabase/supabase-js

# AI
npm install ai @ai-sdk/openai

# NLP & Text Analysis
npm install compromise

# UI (shadcn/ui)
npx shadcn@latest init
npx shadcn@latest add button card tabs badge progress \
  dropdown-menu popover tooltip select radio-group \
  input label avatar dialog sheet separator scroll-area

# Charts
npm install recharts

# Forms
npm install react-hook-form @hookform/resolvers zod

# Theme
npm install next-themes

# Icons
npm install lucide-react

# Utilities
npm install clsx tailwind-merge class-variance-authority date-fns
```

### 1.3 package.json scripts

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "prisma generate && next build",
    "start": "next start",
    "lint": "next lint",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "db:seed": "tsx scripts/seed.ts",
    "db:studio": "prisma studio",
    "content:generate": "tsx scripts/generate-content.ts",
    "content:analyze": "tsx scripts/analyze-text.ts",
    "content:tts": "tsx scripts/generate-tts.ts",
    "content:validate": "tsx scripts/validate-texts.ts"
  }
}
```

---

## 2. 디렉토리 구조

```
reading-ladder/
├── app/
│   ├── layout.tsx                    # 루트 레이아웃 (Providers, Header)
│   ├── page.tsx                      # 랜딩 페이지
│   ├── globals.css                   # Tailwind + CSS 변수
│   │
│   ├── (auth)/                       # 인증 그룹 (레이아웃 없음)
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   │
│   ├── (onboarding)/                 # v2.0 온보딩 그룹 (G1, G2)
│   │   ├── layout.tsx                # 미니멀 레이아웃 (사이드바 없음)
│   │   ├── placement/page.tsx        # 배치 테스트 (10문항)
│   │   └── welcome/page.tsx          # 온보딩 3단계 (프로필→배치→첫 텍스트)
│   │
│   ├── (main)/                       # 메인 앱 그룹 (사이드바 레이아웃)
│   │   ├── layout.tsx                # 사이드바 + 메인 영역
│   │   ├── dashboard/page.tsx        # 학생 대시보드
│   │   ├── stage-map/page.tsx        # 7-Stage 시각 맵
│   │   ├── reading/
│   │   │   ├── page.tsx              # 텍스트 브라우저 (목록)
│   │   │   └── [id]/
│   │   │       ├── page.tsx          # Reading Viewer (Pre/While/Post 3탭)
│   │   │       ├── complete/page.tsx # v2.0 완료 축하 화면 (G5)
│   │   │       └── loading.tsx
│   │   ├── vocabulary/page.tsx       # 어휘 학습 현황
│   │   ├── word-bank/page.tsx        # v2.0 개인 단어장 (G16)
│   │   ├── assignments/page.tsx      # v2.0 내 과제 목록 (G4, 학생용)
│   │   └── progress/page.tsx         # 학습 진도 상세
│   │
│   ├── (teacher)/                    # 교사 전용 그룹
│   │   ├── layout.tsx                # 교사 전용 레이아웃
│   │   ├── console/page.tsx          # 교사 콘솔 (학급 관리)
│   │   ├── class/
│   │   │   ├── page.tsx              # v2.0 학급 목록 (G3)
│   │   │   └── [id]/
│   │   │       ├── page.tsx          # v2.0 학급 상세 (학생 목록, 진도)
│   │   │       └── assignments/page.tsx # v2.0 과제 관리 (G4)
│   │   ├── analytics/page.tsx        # 학습 분석 대시보드
│   │   ├── topics/page.tsx           # 토픽 관리 (Stage 6-7)
│   │   └── content/page.tsx          # 콘텐츠 관리 (admin)
│   │
│   └── api/
│       ├── auth/
│       │   ├── session/route.ts      # GET: 세션 확인
│       │   ├── signin/route.ts       # POST: 로그인
│       │   ├── signout/route.ts      # POST: 로그아웃
│       │   └── signup/route.ts       # POST: 회원가입
│       ├── reading/
│       │   ├── route.ts              # GET: 텍스트 목록 (필터)
│       │   ├── [id]/route.ts         # GET: 텍스트 상세
│       │   ├── [id]/vocabulary/route.ts  # GET: 텍스트별 어휘
│       │   ├── [id]/questions/route.ts   # GET: 텍스트별 문제
│       │   └── [id]/exercises/route.ts   # GET: 텍스트별 연습
│       ├── progress/
│       │   ├── route.ts              # POST: 진도 저장 / GET: 내 진도
│       │   └── [textId]/route.ts     # GET/PUT: 특정 텍스트 진도
│       ├── stage/
│       │   ├── route.ts              # GET: Stage별 통계
│       │   └── promotion/route.ts    # GET: 진급 가능 여부 확인
│       ├── vocab/
│       │   ├── [word]/route.ts       # GET: Neo4j 단어 조회 (프록시)
│       │   ├── batch/route.ts        # POST: 일괄 어휘 조회
│       │   └── five-d/[word]/route.ts # GET: 5D 프로필
│       ├── placement/                  # v2.0 배치 테스트 (G1)
│       │   └── route.ts              # POST: 결과 저장 / GET: 결과 조회
│       ├── class/                      # v2.0 학급 관리 (G3)
│       │   ├── route.ts              # POST: 학급 생성 / GET: 내 학급 목록
│       │   ├── [id]/route.ts         # GET: 학급 상세
│       │   ├── [id]/students/route.ts # GET: 학급 학생 목록
│       │   └── join/route.ts         # POST: 초대 코드로 학급 참가
│       ├── assignments/                # v2.0 과제 (G4)
│       │   ├── route.ts              # POST: 과제 생성 / GET: 내 과제
│       │   └── [id]/submit/route.ts  # POST: 과제 제출
│       ├── achievements/               # v2.0 게이미피케이션 (G5)
│       │   └── route.ts              # POST: 뱃지 확인+부여 / GET: 내 뱃지
│       ├── word-bank/                  # v2.0 개인 단어장 (G16)
│       │   └── route.ts              # POST: 단어 추가 / GET: 내 단어장 / DELETE: 삭제
│       ├── analytics/
│       │   ├── student/[id]/route.ts # GET: 개인 분석
│       │   ├── class/route.ts        # GET: 학급 분석
│       │   ├── skill-analysis/route.ts # v2.0 GET: skill별 오답 패턴 분석 (G15)
│       │   └── export/route.ts       # v2.0 GET: 리포트 PDF/CSV 내보내기 (G18)
│       ├── topics/
│       │   └── route.ts              # CRUD: 사용자 토픽
│       ├── tts/
│       │   └── route.ts              # POST: TTS 생성 요청
│       └── content/
│           ├── generate/route.ts     # POST: 콘텐츠 생성 (admin)
│           └── validate/route.ts     # POST: 난이도 검증
│
├── components/
│   ├── ui/                           # shadcn/ui 프리미티브
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── ... (shadcn 생성 컴포넌트)
│   │
│   ├── providers/
│   │   ├── auth-provider.tsx         # 인증 컨텍스트
│   │   └── theme-provider.tsx        # 다크모드 (next-themes)
│   │
│   ├── layout/
│   │   ├── main-nav.tsx              # 상단 내비게이션
│   │   ├── user-nav.tsx              # 사용자 드롭다운
│   │   ├── sidebar.tsx               # 사이드바 (메인 그룹)
│   │   └── footer.tsx
│   │
│   ├── onboarding/                    # v2.0 온보딩 (G2)
│   │   ├── profile-step.tsx          # 프로필 입력 (이름, 학년, 역할)
│   │   ├── placement-quiz.tsx        # 배치 테스트 10문항 UI
│   │   ├── placement-result.tsx      # 배치 결과 안내 (추천 Stage)
│   │   └── first-text-picker.tsx     # 첫 텍스트 3편 추천
│   │
│   ├── reading/
│   │   ├── reading-tabs.tsx          # v2.0 Pre/While/Post 3탭 컨테이너 (G8)
│   │   ├── pre-reading.tsx           # v2.0 읽기 전 활동 (삽화 예측, 어휘 미리듣기)
│   │   ├── text-viewer.tsx           # 본문 렌더러 (문단별, 어휘 하이라이트)
│   │   ├── vocab-popup.tsx           # 어휘 팝오버 (5D 레이더 미니 + 단어장 추가)
│   │   ├── audio-player.tsx          # TTS 오디오 플레이어
│   │   ├── listen-mode.tsx           # v2.0 듣기 모드 (Listen First → Read Along) (G10)
│   │   ├── comprehension-quiz.tsx    # 이해도 문제 (오답 해설 강화)
│   │   ├── vocab-exercises.tsx       # 어휘 연습 인터페이스
│   │   ├── completion-screen.tsx     # v2.0 완료 축하 + 뱃지 + 다음 추천 (G5, G12)
│   │   ├── text-card.tsx             # 텍스트 목록 카드 (순서 번호 표시)
│   │   ├── stage-filter.tsx          # Stage/CEFR/카테고리 필터
│   │   ├── grammar-highlights.tsx    # 문법 구조 하이라이트
│   │   ├── illustration-viewer.tsx   # v2.0 삽화 렌더러 (G9)
│   │   ├── timed-mode-bar.tsx        # v2.0 수능 실전 타이머 (G14)
│   │   ├── word-bank-button.tsx      # v2.0 "단어장에 추가" 버튼 (G16)
│   │   └── related-texts.tsx         # v2.0 Stage 6→7 토픽 연결 (G17)
│   │
│   ├── stage/
│   │   ├── stage-map-visual.tsx      # 7-Stage 시각 맵 (잠금/해금 표시)
│   │   ├── stage-progress-ring.tsx   # Stage별 진도 원형 차트
│   │   ├── promotion-badge.tsx       # 진급 알림 뱃지
│   │   └── promotion-ceremony.tsx    # v2.0 진급 축하 전체 화면 (G12)
│   │
│   ├── dashboard/
│   │   ├── reading-stats.tsx         # 읽기 통계 카드
│   │   ├── streak-calendar.tsx       # 학습 스트릭 캘린더
│   │   ├── dimension-radar.tsx       # 5D 레이더 차트 (Recharts)
│   │   ├── recent-readings.tsx       # 최근 읽은 텍스트
│   │   ├── recommended-texts.tsx     # 추천 텍스트
│   │   ├── achievement-shelf.tsx     # v2.0 뱃지 진열대 (G5)
│   │   └── assignment-banner.tsx     # v2.0 과제 알림 배너 (G4)
│   │
│   ├── gamification/                  # v2.0 게이미피케이션 (G5)
│   │   ├── badge-icon.tsx            # 뱃지 아이콘 컴포넌트
│   │   ├── badge-toast.tsx           # 뱃지 획득 토스트 알림
│   │   └── streak-indicator.tsx      # 스트릭 불꽃 표시
│   │
│   ├── teacher/
│   │   ├── class-overview.tsx        # 학급 전체 현황
│   │   ├── class-manager.tsx         # v2.0 학급 생성/초대코드 (G3)
│   │   ├── assignment-creator.tsx    # v2.0 과제 생성 폼 (G4)
│   │   ├── submission-tracker.tsx    # v2.0 과제 제출 현황 (G4)
│   │   ├── student-detail.tsx        # 개인 상세 분석
│   │   ├── skill-analysis-chart.tsx  # v2.0 skill별 오답 분석 (G15)
│   │   ├── report-export.tsx         # v2.0 PDF/CSV 내보내기 (G18)
│   │   ├── topic-editor.tsx          # 토픽 편집기
│   │   └── content-review.tsx        # 콘텐츠 검수 인터페이스
│   │
│   └── charts/
│       ├── radar-chart.tsx           # 5D 레이더 차트 래퍼
│       ├── progress-line-chart.tsx   # 진도 추이 라인 차트
│       └── stage-bar-chart.tsx       # Stage별 완료율 바 차트
│
├── lib/
│   ├── utils.ts                      # cn() — clsx + tailwind-merge
│   ├── db.ts                         # Prisma 클라이언트 + DB 함수
│   ├── supabase.ts                   # Supabase 클라이언트 (서버/클라이언트)
│   ├── neo4j.ts                      # Neo4j API 호출 래퍼
│   ├── text-analyzer.ts             # 텍스트 분석 (FK, vocab profile)
│   ├── vocab-profiler.ts            # BNC/COCA K1-K3 + AWL 분류
│   ├── prompts.ts                    # LLM 프롬프트 템플릿 (Stage별 7종)
│   ├── constants.ts                  # Stage 상수, CEFR 매핑, 카테고리
│   └── types.ts                      # 공통 TypeScript 타입
│
├── data/
│   ├── bnc-coca-k1.json             # BNC/COCA 1-1000 단어 목록
│   ├── bnc-coca-k2.json             # 1001-2000
│   ├── bnc-coca-k3.json             # 2001-3000
│   ├── awl.json                      # Academic Word List (570 families)
│   └── fk-lexile-table.json         # FK Grade → Lexile 변환표
│
├── scripts/
│   ├── generate-content.ts           # 콘텐츠 생성 CLI
│   ├── analyze-text.ts              # 텍스트 분석 CLI
│   ├── generate-tts.ts              # TTS 음성 생성 CLI
│   ├── validate-texts.ts            # 난이도 검증 CLI
│   ├── seed.ts                       # DB 시드 (초기 데이터)
│   ├── migrate-5d-app.ts            # 5D 앱 SQLite → Supabase 마이그레이션
│   └── vocab-continuity-audit.ts    # Stage 간 어휘 연속성 검증
│
├── prisma/
│   ├── schema.prisma                 # 전체 DB 스키마
│   └── migrations/
│
├── public/
│   ├── audio/                        # TTS 음성 파일 (or Supabase Storage)
│   └── images/                       # 삽화
│
├── middleware.ts                      # 인증 라우트 보호
├── next.config.mjs
├── tailwind.config.ts
├── tsconfig.json
├── components.json                    # shadcn/ui 설정
├── postcss.config.mjs
└── .env.local
```

---

## 3. 환경 변수

```bash
# .env.local

# ─── Supabase ───
DATABASE_URL="postgresql://postgres.[project-ref]:[password]@aws-0-ap-northeast-1.pooler.supabase.com:6543/postgres"
DIRECT_URL="postgresql://postgres.[project-ref]:[password]@aws-0-ap-northeast-1.pooler.supabase.com:5432/postgres"
NEXT_PUBLIC_SUPABASE_URL="https://[project-ref].supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
SUPABASE_SERVICE_ROLE_KEY="eyJ..."

# ─── Neo4j ───
NEO4J_URI="bolt://localhost:7687"
NEO4J_USER="neo4j"
NEO4J_PASSWORD="iloveLD12**"

# ─── Neo4j API (vocab-knowledge-graph) ───
VOCAB_API_URL="http://localhost:3001"
# vocab-knowledge-graph/apps/api/ 서버 주소

# ─── OpenAI ───
OPENAI_API_KEY="sk-..."

# ─── App ───
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NODE_ENV="development"
```

---

## 4. 데이터베이스 스키마 (Prisma)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

// ════════════════════════════════════════════════
// 사용자 & 인증 (5D 앱 호환)
// ════════════════════════════════════════════════

model User {
  id           String   @id @default(cuid())
  email        String   @unique
  password     String
  name         String?
  role         String   @default("student") // student | teacher | admin
  grade        String?                       // 초3~고3, 대학, 성인 (온보딩에서 수집)
  onboarded    Boolean  @default(false)      // 온보딩 완료 여부
  currentStage Int      @default(1) @map("current_stage") // 현재 학습 Stage
  createdAt    DateTime @default(now()) @map("created_at")
  updatedAt    DateTime @updatedAt      @map("updated_at")

  assessments      Assessment[]
  knowledgeProfile KnowledgeProfile?
  userVocabulary   UserVocabulary[]
  readingProgress  ReadingProgress[]
  placementResult  PlacementResult?
  achievements     Achievement[]
  teacherClasses   Class[]            @relation("TeacherClasses")
  studentClasses   ClassMembership[]  @relation("StudentClasses")
  wordBank         WordBankEntry[]

  @@map("users")
}

// ════════════════════════════════════════════════
// 5D 평가 시스템 (5D 앱 호환)
// ════════════════════════════════════════════════

model Assessment {
  id             String   @id @default(cuid())
  userId         String   @map("user_id")
  assessmentType String   @map("assessment_type")
  difficulty     String
  domain         String
  totalQuestions Int      @map("total_questions")
  score          Float
  completedAt    DateTime @default(now()) @map("completed_at")
  createdAt      DateTime @default(now()) @map("created_at")

  user      User                 @relation(fields: [userId], references: [id], onDelete: Cascade)
  responses AssessmentResponse[]

  @@index([userId])
  @@map("assessments")
}

model AssessmentResponse {
  id              String   @id @default(cuid())
  assessmentId    String   @map("assessment_id")
  questionIndex   Int      @map("question_index")
  word            String
  dimension       String
  question        String
  userResponse    String   @map("user_response")
  correctResponse String   @map("correct_response")
  isCorrect       Boolean  @map("is_correct")
  score           Float
  feedback        String?
  createdAt       DateTime @default(now()) @map("created_at")

  assessment Assessment @relation(fields: [assessmentId], references: [id], onDelete: Cascade)

  @@index([assessmentId])
  @@map("assessment_responses")
}

model KnowledgeProfile {
  id               String   @id @default(cuid())
  userId           String   @unique @map("user_id")
  semanticScore    Float    @default(0) @map("semantic_score")
  contextualScore  Float    @default(0) @map("contextual_score")
  formScore        Float    @default(0) @map("form_score")
  relationalScore  Float    @default(0) @map("relational_score")
  pragmaticScore   Float    @default(0) @map("pragmatic_score")
  totalAssessments Int      @default(0) @map("total_assessments")
  createdAt        DateTime @default(now()) @map("created_at")
  updatedAt        DateTime @updatedAt      @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("knowledge_profiles")
}

model Vocabulary {
  id        String   @id @default(cuid())
  word      String   @unique
  cefrLevel String   @map("cefr_level")
  frequency Int      @default(0)
  domain    String
  createdAt DateTime @default(now()) @map("created_at")

  userVocabulary UserVocabulary[]

  @@map("vocabulary")
}

model UserVocabulary {
  id              String    @id @default(cuid())
  userId          String    @map("user_id")
  vocabularyId    String    @map("vocabulary_id")
  semanticScore   Float     @default(0) @map("semantic_score")
  contextualScore Float     @default(0) @map("contextual_score")
  formScore       Float     @default(0) @map("form_score")
  relationalScore Float     @default(0) @map("relational_score")
  pragmaticScore  Float     @default(0) @map("pragmatic_score")
  lastAssessed    DateTime? @map("last_assessed")
  createdAt       DateTime  @default(now()) @map("created_at")
  updatedAt       DateTime  @updatedAt      @map("updated_at")

  user       User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  vocabulary Vocabulary @relation(fields: [vocabularyId], references: [id], onDelete: Cascade)

  @@unique([userId, vocabularyId])
  @@index([userId])
  @@index([vocabularyId])
  @@map("user_vocabulary")
}

// ════════════════════════════════════════════════
// Reading Ladder 핵심 모델
// ════════════════════════════════════════════════

model ReadingText {
  id          String  @id @default(uuid())
  stage       Int
  title       String
  subtitle    String?
  category    String  // phonics, fable, classic, biography, career, csat, academic
  subcategory String?
  topicArea   String? @map("topic_area")
  topicNumber Int?    @map("topic_number")

  content     String
  contentHtml String? @map("content_html")

  // 난이도 지표
  cefrLevel          String @map("cefr_level")
  fleschKincaidGrade Float? @map("flesch_kincaid_grade")
  fleschReadingEase  Float? @map("flesch_reading_ease")
  colemanLiauIndex   Float? @map("coleman_liau_index")
  lexileEstimate     Int?   @map("lexile_estimate")
  lexileSource       String @default("estimated") @map("lexile_source")

  // 텍스트 분석 (자동 계산)
  wordCount        Int    @map("word_count")
  sentenceCount    Int?   @map("sentence_count")
  avgSentenceLength Float? @map("avg_sentence_length")
  avgWordLength    Float? @map("avg_word_length")
  uniqueWordCount  Int?   @map("unique_word_count")
  lexicalDensity   Float? @map("lexical_density")

  // 어휘 프로파일 (자동 계산)
  k1Pct      Float? @map("k1_pct")
  k2Pct      Float? @map("k2_pct")
  k3Pct      Float? @map("k3_pct")
  awlPct     Float? @map("awl_pct")
  offListPct Float? @map("off_list_pct")

  // 수능 연계
  csatType       String? @map("csat_type")
  csatDifficulty String? @map("csat_difficulty")

  // 메타
  author             String?
  originalWork       String?  @map("original_work")
  illustrationRatio  Float?   @default(0) @map("illustration_ratio")
  audioUrl           String?  @map("audio_url")
  koreanTranslation  String?  @map("korean_translation")
  status             String   @default("draft") // draft, reviewed, published

  // 순서 & 브릿지
  orderIndex      Int     @default(0) @map("order_index")  // Stage 내 권장 읽기 순서
  isBridge        Boolean @default(false) @map("is_bridge")
  bridgeFromStage Int?    @map("bridge_from_stage")
  bridgeToStage   Int?    @map("bridge_to_stage")

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt      @map("updated_at")

  paragraphs    TextParagraph[]
  vocabulary    TextVocabulary[]
  grammarTags   TextGrammarTag[]
  questions     TextQuestion[]
  exercises     TextExercise[]
  progress      ReadingProgress[]
  illustrations TextIllustration[]
  assignments   Assignment[]

  @@index([stage])
  @@index([cefrLevel])
  @@index([category])
  @@index([csatType])
  @@index([status])
  @@index([stage, orderIndex])
  @@map("reading_texts")
}

model TextParagraph {
  id             String  @id @default(uuid())
  textId         String  @map("text_id")
  paragraphIndex Int     @map("paragraph_index")
  content        String
  wordCount      Int?    @map("word_count")
  keyIdea        String? @map("key_idea")
  keyIdeaKo      String? @map("key_idea_ko")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@unique([textId, paragraphIndex])
  @@map("text_paragraphs")
}

model TextVocabulary {
  id              String  @id @default(uuid())
  textId          String  @map("text_id")
  neo4jWordId     String? @map("neo4j_word_id")
  word            String
  partOfSpeech    String? @map("part_of_speech")
  cefrLevel       String? @map("cefr_level")
  definitionEn    String? @map("definition_en")
  definitionKo    String? @map("definition_ko")
  exampleSentence String? @map("example_sentence")
  phonetic        String?
  fiveDDimension  String? @map("five_d_dimension")
  orderInText     Int?    @map("order_in_text")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@unique([textId, word])
  @@index([word])
  @@index([neo4jWordId])
  @@map("text_vocabulary")
}

model TextGrammarTag {
  id              String  @id @default(uuid())
  textId          String  @map("text_id")
  feature         String
  cefrLevel       String? @map("cefr_level")
  exampleFromText String? @map("example_from_text")
  explanation     String?
  explanationKo   String? @map("explanation_ko")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@map("text_grammar_tags")
}

model TextQuestion {
  id            String  @id @default(uuid())
  textId        String  @map("text_id")
  questionType  String  @map("question_type")  // multiple_choice, true_false, short_answer, open_ended
  question      String
  questionKo    String? @map("question_ko")
  options       Json?
  correctAnswer String  @map("correct_answer")
  explanation   String?
  explanationKo String? @map("explanation_ko")
  distractorExplanations Json? @map("distractor_explanations") // 각 오답별 해설 {"①":"...", "②":"..."}
  referenceLines Json?  @map("reference_lines")  // 정답 근거 문장 인덱스 [2, 5]
  skill         String? // literal, inferential, evaluative, creative
  csatType      String? @map("csat_type")
  orderIndex    Int     @default(0) @map("order_index")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@map("text_questions")
}

model TextExercise {
  id           String  @id @default(uuid())
  textId       String  @map("text_id")
  exerciseType String  @map("exercise_type")  // fill_blank, matching, word_form, synonym_antonym, context_clue
  prompt       String
  promptKo     String? @map("prompt_ko")
  answer       String
  distractors  Json?
  orderIndex   Int     @default(0) @map("order_index")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@map("text_exercises")
}

model ReadingProgress {
  id                 String    @id @default(uuid())
  userId             String    @map("user_id")
  textId             String    @map("text_id")
  status             String    @default("not_started") // not_started, in_progress, completed
  startedAt          DateTime? @map("started_at")
  completedAt        DateTime? @map("completed_at")
  comprehensionScore Int?      @map("comprehension_score")
  vocabularyScore    Int?      @map("vocabulary_score")
  timeSpent          Int       @default(0) @map("time_spent")      // 초 단위
  readCount          Int       @default(0) @map("read_count")
  quizAttempts       Int       @default(0) @map("quiz_attempts")   // 문제 재시도 횟수
  bestScore          Int?      @map("best_score")                  // 최고 점수 (재시도 중)
  readingMode        String?   @map("reading_mode")                // practice | timed (수능 실전)
  notes              String?

  user User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@unique([userId, textId])
  @@index([userId])
  @@index([textId])
  @@map("reading_progress")
}

model StagePromotion {
  stage                    Int @id
  requiredTextsCompleted   Int @default(14) @map("required_texts_completed")
  requiredAvgComprehension Int @default(70) @map("required_avg_comprehension")
  requiredVocabMastery     Int @default(60) @map("required_vocab_mastery")
  requiredBridgesCompleted Int @default(2)  @map("required_bridges_completed")

  @@map("stage_promotions")
}

model UserTopic {
  id                   String   @id @default(uuid())
  areaCode             String   @map("area_code")
  areaName             String   @map("area_name")
  areaNameKo           String?  @map("area_name_ko")
  topicNumber          Int      @map("topic_number")
  title                String
  titleKo              String?  @map("title_ko")
  description          String?
  keywords             Json?
  relatedAcademicField String?  @map("related_academic_field")
  createdBy            String?  @map("created_by")
  createdAt            DateTime @default(now()) @map("created_at")

  @@unique([areaCode, topicNumber])
  @@map("user_topics")
}

// ════════════════════════════════════════════════
// v2.0 추가: 배치 테스트 (UX Gap G1)
// ════════════════════════════════════════════════

model PlacementResult {
  id            String   @id @default(uuid())
  userId        String   @unique @map("user_id")
  assignedStage Int      @map("assigned_stage")
  score         Float
  responses     Json     // 각 문제별 응답 { questionIndex, stageLevel, answer, correct }
  completedAt   DateTime @default(now()) @map("completed_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("placement_results")
}

// ════════════════════════════════════════════════
// v2.0 추가: 학급 & 과제 (UX Gap G3, G4)
// ════════════════════════════════════════════════

model Class {
  id         String   @id @default(uuid())
  name       String   // "중2-3반"
  teacherId  String   @map("teacher_id")
  gradeLevel String?  @map("grade_level")
  inviteCode String   @unique @map("invite_code") // 학생 초대 코드 (6자리)
  createdAt  DateTime @default(now()) @map("created_at")

  teacher     User              @relation("TeacherClasses", fields: [teacherId], references: [id])
  memberships ClassMembership[]
  assignments Assignment[]

  @@map("classes")
}

model ClassMembership {
  id        String   @id @default(uuid())
  classId   String   @map("class_id")
  studentId String   @map("student_id")
  joinedAt  DateTime @default(now()) @map("joined_at")

  class   Class @relation(fields: [classId], references: [id], onDelete: Cascade)
  student User  @relation("StudentClasses", fields: [studentId], references: [id], onDelete: Cascade)

  @@unique([classId, studentId])
  @@map("class_memberships")
}

model Assignment {
  id           String    @id @default(uuid())
  classId      String    @map("class_id")
  textId       String    @map("text_id")
  teacherId    String    @map("teacher_id")
  title        String?
  instructions String?
  dueDate      DateTime? @map("due_date")
  createdAt    DateTime  @default(now()) @map("created_at")

  class       Class                 @relation(fields: [classId], references: [id], onDelete: Cascade)
  text        ReadingText           @relation(fields: [textId], references: [id])
  submissions AssignmentSubmission[]

  @@map("assignments")
}

model AssignmentSubmission {
  id           String    @id @default(uuid())
  assignmentId String    @map("assignment_id")
  studentId    String    @map("student_id")
  progressId   String?   @map("progress_id") // ReadingProgress 연결
  submittedAt  DateTime? @map("submitted_at")

  assignment Assignment @relation(fields: [assignmentId], references: [id], onDelete: Cascade)

  @@unique([assignmentId, studentId])
  @@map("assignment_submissions")
}

// ════════════════════════════════════════════════
// v2.0 추가: 게이미피케이션 (UX Gap G5)
// ════════════════════════════════════════════════

model Achievement {
  id       String   @id @default(uuid())
  userId   String   @map("user_id")
  type     String   // first_read, stage_complete_N, streak_7, streak_30, vocab_50, perfect_score, ...
  earnedAt DateTime @default(now()) @map("earned_at")
  metadata Json?    // { stage: 1, textTitle: "A Cat and a Hat", score: 100 }

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("achievements")
}

// ════════════════════════════════════════════════
// v2.0 추가: 삽화 모델 (UX Gap G9)
// ════════════════════════════════════════════════

model TextIllustration {
  id             String  @id @default(uuid())
  textId         String  @map("text_id")
  imageUrl       String  @map("image_url")    // Supabase Storage URL
  altText        String? @map("alt_text")
  position       String  @default("top")       // top, inline, paragraph_N
  paragraphIndex Int?    @map("paragraph_index") // inline일 경우 몇 번째 문단 뒤
  orderIndex     Int     @default(0) @map("order_index")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@map("text_illustrations")
}

// ════════════════════════════════════════════════
// v2.0 추가: 개인 단어장 (UX Gap G16)
// ════════════════════════════════════════════════

model WordBankEntry {
  id        String   @id @default(uuid())
  userId    String   @map("user_id")
  word      String
  textId    String?  @map("text_id")   // 어디서 추가했는지
  note      String?                     // 학생 메모
  mastered  Boolean  @default(false)
  createdAt DateTime @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([userId, word])
  @@index([userId])
  @@map("word_bank")
}
```

---

## 5. Neo4j 연동 모듈

```typescript
// lib/neo4j.ts
// vocab-knowledge-graph API 서버를 프록시 호출

const VOCAB_API_URL = process.env.VOCAB_API_URL || 'http://localhost:3001'

export interface VocabEntry {
  word: string
  display: string
  pos: string
  ipa: string
  cefr: string
  freqRank: number | null
  meaningKo: string
  definitionEn: string
  synonym: string
  antonym: string
  hypernym: string
  collocation: string
  sentence1: string
  sentence2: string
  sentence3: string
  domain: string
  topic: string
  wordFamily: string
  morpheme: string
  register: string
  errorPattern: string
}

export interface FiveDProfile {
  word: VocabEntry
  cefr: string
  topics: string[]
  domains: string[]
  synonyms: string[]
  antonyms: string[]
  hypernyms: string[]
  hyponyms: string[]
  wordFamily: string[]
  collocations: string[]
}

// 단어 조회
export async function getWord(word: string): Promise<VocabEntry | null> {
  const res = await fetch(`${VOCAB_API_URL}/api/vocab/${encodeURIComponent(word)}`)
  if (!res.ok) return null
  return res.json()
}

// 5D 프로필 조회
export async function get5DProfile(word: string): Promise<FiveDProfile | null> {
  const res = await fetch(`${VOCAB_API_URL}/api/vocab/5d-profile/${encodeURIComponent(word)}`)
  if (!res.ok) return null
  return res.json()
}

// 일괄 조회 (텍스트 내 어휘 매칭)
export async function batchLookup(words: string[]): Promise<VocabEntry[]> {
  const res = await fetch(`${VOCAB_API_URL}/api/vocab/batch`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ words }),
  })
  if (!res.ok) return []
  return res.json()
}

// 관련어 조회
export async function getRelatedWords(word: string, limit = 10): Promise<VocabEntry[]> {
  const res = await fetch(
    `${VOCAB_API_URL}/api/vocab/related/${encodeURIComponent(word)}?limit=${limit}`
  )
  if (!res.ok) return []
  return res.json()
}

// CEFR별 단어 검색
export async function searchByFilters(filters: {
  cefr?: string
  pos?: string
  topic?: string
  domain?: string
  limit?: number
}): Promise<VocabEntry[]> {
  const params = new URLSearchParams()
  Object.entries(filters).forEach(([k, v]) => { if (v) params.set(k, String(v)) })
  const res = await fetch(`${VOCAB_API_URL}/api/vocab/search?${params}`)
  if (!res.ok) return []
  return res.json()
}
```

---

## 6. API Routes 명세

### 6.1 인증 API

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/auth/signup` | POST | `{ email, password, name, role }` | `{ user }` | cookie 설정 |
| `/api/auth/signin` | POST | `{ email, password }` | `{ user }` | cookie 설정 |
| `/api/auth/signout` | POST | — | `{ ok }` | cookie 삭제 |
| `/api/auth/session` | GET | — | `{ user } \| null` | cookie 검증 |

### 6.2 Reading API

| Endpoint | Method | Request/Params | Response | 비고 |
|----------|--------|----------------|----------|------|
| `/api/reading` | GET | `?stage=3&category=classic&cefr=B1&status=published` | `ReadingText[]` | 필터링 + 페이지네이션 |
| `/api/reading/[id]` | GET | — | `ReadingText + paragraphs` | 본문 포함 |
| `/api/reading/[id]/vocabulary` | GET | — | `TextVocabulary[]` + Neo4j 5D 데이터 | Neo4j 병합 |
| `/api/reading/[id]/questions` | GET | — | `TextQuestion[]` | 문제 목록 |
| `/api/reading/[id]/exercises` | GET | — | `TextExercise[]` | 연습 목록 |

### 6.3 Progress API

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/progress` | GET | `?userId=...` | `ReadingProgress[]` | 전체 진도 |
| `/api/progress` | POST | `{ textId, status, scores... }` | `ReadingProgress` | 진도 저장/갱신 |
| `/api/progress/[textId]` | GET | — | `ReadingProgress` | 특정 텍스트 진도 |
| `/api/progress/[textId]` | PUT | `{ comprehensionScore, ... }` | `ReadingProgress` | 점수 업데이트 |

### 6.4 Stage API

| Endpoint | Method | Response | 비고 |
|----------|--------|----------|------|
| `/api/stage` | GET | `StageStats[]` — 각 Stage별 { total, completed, avgScore } | userId 기반 |
| `/api/stage/promotion` | GET | `{ currentStage, canPromote, missing }` | 진급 조건 체크 |

### 6.5 Vocab Proxy API (Neo4j)

| Endpoint | Method | 비고 |
|----------|--------|------|
| `/api/vocab/[word]` | GET | Neo4j → VocabEntry |
| `/api/vocab/batch` | POST | `{ words: string[] }` → VocabEntry[] |
| `/api/vocab/five-d/[word]` | GET | Neo4j → FiveDProfile |

### 6.6 Placement API (v2.0 — G1)

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/placement` | POST | `{ responses: [{questionIndex, stageLevel, answer}] }` | `{ assignedStage, score }` | Stage 자동 결정 알고리즘 |
| `/api/placement` | GET | — | `PlacementResult \| null` | 기존 배치 결과 조회 |

**Stage 결정 알고리즘:**
- Q1-Q2 (Stage 1-2) 오답 → Stage 1 배정
- Q3-Q5 (Stage 3-5) 까지 정답 → 해당 Stage 배정
- Q6-Q10 (Stage 6-7) 정답 패턴에 따라 Stage 6 또는 7 배정
- 70% 이상 정답인 최고 Stage를 `assignedStage`로 결정

### 6.7 Class API (v2.0 — G3)

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/class` | POST | `{ name, gradeLevel? }` | `{ class, inviteCode }` | 교사 전용, 6자리 초대코드 자동생성 |
| `/api/class` | GET | — | `Class[]` | 교사: 내 학급 / 학생: 참여 학급 |
| `/api/class/[id]` | GET | — | `Class + memberships + stats` | 학급 상세 + 학생 진도 요약 |
| `/api/class/[id]/students` | GET | — | `{ student, progress, stage }[]` | 학생별 진도 상세 |
| `/api/class/join` | POST | `{ inviteCode }` | `{ class }` | 학생이 초대코드로 학급 참가 |

### 6.8 Assignment API (v2.0 — G4)

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/assignments` | POST | `{ classId, textId, title?, instructions?, dueDate? }` | `Assignment` | 교사 전용 |
| `/api/assignments` | GET | `?role=student\|teacher` | `Assignment[]` | 학생: 내 과제 / 교사: 발행 과제 |
| `/api/assignments/[id]/submit` | POST | `{ progressId }` | `AssignmentSubmission` | 학생 과제 제출 |

### 6.9 Achievement API (v2.0 — G5)

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/achievements` | GET | — | `Achievement[]` | 내 뱃지 목록 |
| `/api/achievements` | POST | `{ trigger: 'reading_complete' \| 'stage_promote' \| ... }` | `Achievement[] \| null` | 조건 체크 후 신규 뱃지 부여, 없으면 null |

**뱃지 트리거 조건:**

| type | 조건 |
|------|------|
| `first_read` | 첫 텍스트 완료 |
| `stage_complete_N` | Stage N 진급 조건 충족 |
| `streak_7` / `streak_30` | 7/30일 연속 학습 |
| `vocab_50` / `vocab_100` / `vocab_500` | 단어장 누적 어휘 수 |
| `perfect_score` | 이해도 100점 |

### 6.10 Word Bank API (v2.0 — G16)

| Endpoint | Method | Request | Response | 비고 |
|----------|--------|---------|----------|------|
| `/api/word-bank` | GET | `?mastered=false` | `WordBankEntry[]` | 내 단어장 |
| `/api/word-bank` | POST | `{ word, textId?, note? }` | `WordBankEntry` | 단어 추가 |
| `/api/word-bank` | DELETE | `{ word }` | `{ ok }` | 단어 삭제 |

### 6.11 Analytics API (교사용, 확장)

| Endpoint | Method | Response | 비고 |
|----------|--------|----------|------|
| `/api/analytics/student/[id]` | GET | 개인 Stage별 진도, 5D 프로필, 어휘 숙달도 | 교사 권한 |
| `/api/analytics/class` | GET | 학급 평균, Stage 분포, 약점 차원 | SQL 집계 |
| `/api/analytics/skill-analysis` | GET | `?userId=...&csatType=...` → skill별 정답률 | v2.0 오답 패턴 분석 (G15) |
| `/api/analytics/export` | GET | `?classId=...&format=pdf\|csv` → 파일 다운로드 | v2.0 리포트 내보내기 (G18) |

### 6.12 콘텐츠 관리 API (admin)

| Endpoint | Method | 비고 |
|----------|--------|------|
| `/api/content/generate` | POST | 콘텐츠 생성 파이프라인 트리거 |
| `/api/content/validate` | POST | 텍스트 난이도 검증 |
| `/api/topics` | GET/POST/PUT/DELETE | 사용자 토픽 CRUD |
| `/api/tts` | POST | TTS 생성 요청 → Supabase Storage |

---

## 7. 페이지 라우트 & 화면 명세

### 7.1 배치 테스트 — `/placement` (v2.0 — G1)

```text
┌─────────────────────────────────────────────────────────┐
│                                                         │
│           어디서 시작할지 알아볼까요?                      │
│           간단한 문제 10개만 풀어보세요 (약 5분)            │
│                                                         │
│  ┌─── 진행 바 ──────────────────────────────────────┐   │
│  │ ●●●●●○○○○○  5/10                                 │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  Q5. [Stage 5 수준]                                     │
│  "The article suggests that renewable energy will       │
│   eventually ______ fossil fuels because of..."         │
│                                                         │
│   ○ replace                                             │
│   ○ complement                                          │
│   ● supplement                                          │
│   ○ overcome                                            │
│                                                         │
│                                      [다음 →]           │
│                                                         │
│  ─────────────────────────────────────────────────────  │
│  결과 화면:                                              │
│  "당신의 읽기 수준: Stage 4 (CEFR B1)입니다!"             │
│  [레이더 차트: Stage 1-3 강점, Stage 5+ 도전 영역]       │
│                                                         │
│  [내 Stage에서 시작하기 →]                                │
└─────────────────────────────────────────────────────────┘
```

### 7.2 온보딩 — `/welcome` (v2.0 — G2)

```text
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Step [1] → [2] → [3]                                   │
│                                                         │
│  ── Step 1: 프로필 ──                                    │
│  "이름이 뭐예요?"    [________________]                   │
│  "몇 학년이에요?"    [초5 ▼]                              │
│  "선생님이에요, 학생이에요?"  [학생 ○] [교사 ○]             │
│                                      [다음 →]           │
│                                                         │
│  ── Step 2: 배치 테스트 ──                               │
│  → /placement 리다이렉트                                 │
│                                                         │
│  ── Step 3: 첫 텍스트 추천 ──                            │
│  "당신에게 딱 맞는 첫 번째 이야기를 골라볼까요?"            │
│                                                         │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐           │
│  │ Aesop      │ │ Treasure   │ │ The Fox    │           │
│  │ Stage 2    │ │ Stage 2    │ │ Stage 2    │           │
│  │ [읽기 →]   │ │ [읽기 →]   │ │ [읽기 →]   │           │
│  └────────────┘ └────────────┘ └────────────┘           │
│                                                         │
│  [대시보드로 가기 →]                                      │
└─────────────────────────────────────────────────────────┘
```

**플로우 규칙:**
- 가입 직후 `onboarded === false` → `/welcome` 리다이렉트
- 배치 테스트 완료 시 `currentStage` 자동 설정
- Step 3 또는 "대시보드로" 클릭 시 `onboarded = true` 저장

### 7.3 랜딩 페이지 — `/`

```
┌─────────────────────────────────────────────────────────┐
│ [Logo] EFL Reading Ladder              [Login] [Signup] │
├─────────────────────────────────────────────────────────┤
│                                                         │
│     파닉스부터 학술 지문까지                              │
│     7단계 리딩 로드맵으로 영어 읽기를 정복하세요            │
│                                                         │
│     [시작하기 →]                                         │
│                                                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐                   │
│  │ Stage 1 │ │ Stage 4 │ │ Stage 7 │                   │
│  │ Phonics │ │위인전    │ │ Academic│                   │
│  │ 30-80w  │ │250-400w │ │600-1200w│                   │
│  └─────────┘ └─────────┘ └─────────┘                   │
│                                                         │
│  7단계 · 200편 · 5D 어휘 연동 · 수능 대비               │
└─────────────────────────────────────────────────────────┘
```

### 7.2 Stage Map — `/stage-map`

```
┌─────────────────────────────────────────────────────────┐
│  [Sidebar]  │  My Reading Journey                       │
│             │                                           │
│  Dashboard  │  ┌───┐                                    │
│  Stage Map ◄│  │ 7 │ Academic Discourse (C1)            │
│  Reading    │  └─┬─┘          ○○○○○○○○○○ 0/20          │
│  Vocabulary │    │                                      │
│  Progress   │  ┌─┴─┐                                    │
│             │  │ 6 │ CSAT Critical Reading (B2+)        │
│             │  └─┬─┘          ○○○○○○○○○○ 0/40          │
│             │    │                                      │
│             │  ┌─┴─┐                                    │
│             │  │ 5 │ Future Careers (B2)                │
│             │  └─┬─┘          ●●●○○○○○○○ 3/20          │
│             │    │                                      │
│             │  ┌─┴─┐                                    │
│             │  │ 4 │ Great Lives (B1+)  ← 현재 Stage   │
│             │  └─┬─┘          ●●●●●●●●○○ 18/30         │
│             │    │                                      │
│             │  ┌─┴─┐                                    │
│             │  │ 3 │ Classic Stories (B1) ✅ 완료       │
│             │  └─┬─┘          ●●●●●●●●●● 30/30         │
│             │    │                                      │
│             │  ┌─┴─┐                                    │
│             │  │ 2 │ Aesop's Fables (A2) ✅ 완료      │
│             │  └─┬─┘          ●●●●●●●●●● 20/20         │
│             │    │                                      │
│             │  ┌─┴─┐                                    │
│             │  │ 1 │ Phonics Readers (A1) ✅ 완료     │
│             │  └───┘          ●●●●●●●●●● 20/20         │
└─────────────┴───────────────────────────────────────────┘
```

### 7.3 Text Browser — `/reading`

```
┌─────────────────────────────────────────────────────────┐
│  [Sidebar]  │  Reading Library                          │
│             │                                           │
│             │  Stage: [All ▼] Category: [All ▼]         │
│             │  CEFR:  [All ▼] CSAT Type: [All ▼]       │
│             │  Search: [________________]               │
│             │                                           │
│             │  ┌────────────────────────────────┐       │
│             │  │ 📖 Treasure Island              │       │
│             │  │ Stage 3 · Classic · A2-B1       │       │
│             │  │ 260 words · Adventure           │       │
│             │  │ ●●●○○ 이해도 60%               │       │
│             │  │ [읽기 →]                        │       │
│             │  └────────────────────────────────┘       │
│             │  ┌────────────────────────────────┐       │
│             │  │ 📖 Marie Curie                  │       │
│             │  │ Stage 4 · Biography · B1        │       │
│             │  │ 300 words · Science             │       │
│             │  │ 아직 안 읽음                     │       │
│             │  │ [읽기 →]                        │       │
│             │  └────────────────────────────────┘       │
└─────────────┴───────────────────────────────────────────┘
```

### 7.6 Reading Viewer — `/reading/[id]` (v2.0: Pre/While/Post 3탭)

```text
┌─────────────────────────────────────────────────────────┐
│  ← Back    "Treasure Island"   🔊 Play Audio   Stage 3 │
│  Classic Stories > Adventure     CEFR A2-B1   260 words │
│  #12 / 30편                      [연습 모드 ▼]          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │  [1. 읽기 전]  →  [2. 읽기]  →  [3. 읽기 후]       │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  ═══ Tab 1: 읽기 전 (Pre-reading) ═══                   │
│  ┌────────────────────────────────────────────────────┐ │
│  │  🖼️ [삽화 미리보기]                                │ │
│  │                                                    │ │
│  │  "이 그림을 보고 무슨 이야기일까 생각해보세요"       │ │
│  │                                                    │ │
│  │  핵심 어휘 미리 듣기:                               │ │
│  │    🔊 treasure  🔊 island  🔊 map  🔊 captain     │ │
│  │                                                    │ │
│  │  [읽기 시작하기 →]                                  │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  ═══ Tab 2: 읽기 (While-reading) ═══                    │
│  ┌─── Text ──────────────────────┬── Vocabulary ──────┐ │
│  │                               │                     │ │
│  │  [삽화: 해적 보물 지도]       │ 학습 어휘 (8개)     │ │
│  │                               │                     │ │
│  │  Jim Hawkins found an old     │ treasure /ˈtreʒ.ər/ │ │
│  │  map in the sea captain's     │  보물 (n.)          │ │
│  │  chest. The [map] showed the  │  CEFR: A2           │ │
│  │  location of a hidden         │  5D: Semantic       │ │
│  │  [treasure] on a faraway      │  🔊 [+단어장] [↗5D] │ │
│  │  island.                      │                     │ │
│  │                               │ faraway /ˌfɑːrəˈweɪ/│ │
│  │  ¶ Key idea: Jim finds a      │  먼, 멀리 떨어진    │ │
│  │    treasure map.              │  CEFR: A2           │ │
│  │                               │  5D: Contextual     │ │
│  │  He showed the map to         │  🔊 [+단어장] [↗5D] │ │
│  │  Doctor Livesey and           │                     │ │
│  │  Squire Trelawney...          │ ...                  │ │
│  └───────────────────────────────┴─────────────────────┘ │
│                                                         │
│  ┌─── Grammar Highlights ────────────────────────────┐  │
│  │ past simple: "found", "showed", "decided"   [A2]  │  │
│  │ to-infinitive: "decided to sail"            [A2]  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ═══ Tab 3: 읽기 후 (Post-reading) ═══                  │
│  ┌─── Comprehension Check (0/5) ─────────────────────┐  │
│  │ Q1. What did Jim find in the chest?               │  │
│  │   ○ A sword  ○ A map  ○ Gold  ○ A letter         │  │
│  │   [Submit →]                                      │  │
│  │                                                   │  │
│  │ (오답 시) ❌ "정답은 A map이에요."                  │  │
│  │   📌 근거 문장: "Jim Hawkins found an old map..."  │  │
│  │   💡 다른 보기 해설:                               │  │
│  │     - A sword: 본문에 검 관련 내용 없음            │  │
│  │     - Gold: 보물은 있지만 직접 발견한 것은 지도    │  │
│  │   [다시 풀기] [다음 문제 →]                        │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─── Vocabulary Exercises (0/4) ────────────────────┐  │
│  │ Fill in the blank:                                │  │
│  │ "The pirate hid the ______ on the island."        │  │
│  │   [treasure]  [map]  [ship]  [chest]              │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─── 이 토픽 심화 학술 지문 (Stage 7) ────────────── ┐ │
│  │ 📖 "Maritime Exploration & Colonial Commerce"      │ │
│  │    Stage 7 · Academic · C1 · 800 words             │ │
│  │    [심화 지문 읽기 →]                              │ │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘

수능 실전 모드 (Stage 6-7, readingMode === 'timed'):
┌─────────────────────────────────────────────────────────┐
│  ⏱️ 02:00 남음                      [실전 모드 🔴]      │
│  ⚠️ 타이머 종료 시 자동 제출됩니다                        │
└─────────────────────────────────────────────────────────┘
```

### 7.7 완료 축하 화면 — `/reading/[id]/complete` (v2.0 — G5, G12)

```text
┌─────────────────────────────────────────────────────────┐
│                                                         │
│                 🎉 읽기 완료!                            │
│           "Treasure Island"                              │
│                                                         │
│     이해도: ⭐⭐⭐⭐☆ 80점                               │
│     학습 어휘: 8개                                       │
│     읽기 시간: 5분 32초                                  │
│                                                         │
│     ┌──────────────────────────┐                        │
│     │  🏅 뱃지 획득!           │                        │
│     │  "Stage 3 탐험가"        │                        │
│     │  Classic Stories 10편    │                        │
│     │  완료!                   │                        │
│     └──────────────────────────┘                        │
│                                                         │
│     [다음 이야기 →]  "Robinson Crusoe" (#13)             │
│     [대시보드로]                                         │
│     [다시 풀기]                                          │
│                                                         │
│  ─────────────────────────────────────────────────────  │
│  진급 알림 (조건 충족 시):                                │
│  🎊 "Stage 4 해금! Great Lives로 떠나볼까요?"            │
│  [Stage 4 시작하기 →]                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 7.5 Dashboard — `/dashboard`

```
┌─────────────────────────────────────────────────────────┐
│  [Sidebar]  │  My Dashboard                             │
│             │                                           │
│             │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐     │
│             │  │  78  │ │  12  │ │  4   │ │ 156  │     │
│             │  │텍스트 │ │연속일│ │현재  │ │학습  │     │
│             │  │완료   │ │스트릭│ │Stage │ │어휘  │     │
│             │  └──────┘ └──────┘ └──────┘ └──────┘     │
│             │                                           │
│             │  ┌─ 5D Profile ──┐ ┌─ Stage Progress ──┐ │
│             │  │   [레이더     │ │   [바 차트]        │ │
│             │  │    차트]      │ │   S1 ████████ 100% │ │
│             │  │    Sem: 72    │ │   S2 ████████ 100% │ │
│             │  │    Con: 65    │ │   S3 ████████ 100% │ │
│             │  │    For: 80    │ │   S4 ██████░░  60% │ │
│             │  │    Rel: 58    │ │   S5 ██░░░░░░  15% │ │
│             │  │    Pra: 45    │ │   S6 ░░░░░░░░   0% │ │
│             │  └───────────────┘ └────────────────────┘ │
│             │                                           │
│             │  ┌─ Recommended ────────────────────────┐ │
│             │  │ 📖 Ada Lovelace (Stage 4, B1)        │ │
│             │  │    약점: Pragmatic 차원 강화 텍스트   │ │
│             │  │ 📖 Cybersecurity Expert (Stage 5)    │ │
│             │  │    다음 Stage 미리보기               │ │
│             │  └──────────────────────────────────────┘ │
└─────────────┴───────────────────────────────────────────┘
```

---

## 8. 컴포넌트 설계

### 8.1 핵심 컴포넌트 트리

```
<RootLayout>
  <ThemeProvider>
    <AuthProvider>
      <Header>
        <MainNav />
        <UserNav />
      </Header>
      <MainLayout>        ← (main) 그룹
        <Sidebar />
        <Content>
          {children}       ← 페이지별 컴포넌트
        </Content>
      </MainLayout>
    </AuthProvider>
  </ThemeProvider>
</RootLayout>
```

### 8.2 컴포넌트별 Props & 역할

| 컴포넌트 | Props | 역할 |
|---------|-------|------|
| `ReadingTabs` | `{ textId, stage, children }` | Pre/While/Post 3탭 컨테이너 (v2.0) |
| `PreReading` | `{ text, illustrations, vocabulary }` | 읽기 전: 삽화 예측, 핵심 어휘 미리 듣기 (v2.0) |
| `TextViewer` | `{ text, paragraphs, vocabulary, illustrations }` | 본문 렌더링, 어휘 하이라이트, 삽화 인라인, key idea |
| `VocabPopup` | `{ word, vocab, fiveD?, onAddToBank }` | 팝오버 (발음, 뜻, 5D 미니 레이더, +단어장, ↗5D앱) (v2.0 확장) |
| `AudioPlayer` | `{ audioUrl, title }` | TTS 재생/일시정지, 속도 조절 |
| `ListenMode` | `{ audioUrl, paragraphs }` | Listen First → Read Along → Read Alone 모드 (v2.0) |
| `ComprehensionQuiz` | `{ questions, onComplete, allowRetry }` | 문제 풀기, 오답별 해설, 근거 문장 하이라이트, 재시도 (v2.0 확장) |
| `VocabExercises` | `{ exercises, onComplete }` | 어휘 연습 UI (빈칸, 매칭 등) |
| `CompletionScreen` | `{ text, progress, newAchievements, nextText }` | 완료 축하, 뱃지, 다음 텍스트 추천 (v2.0) |
| `TextCard` | `{ text, progress, orderIndex }` | 목록 카드 (순서 번호, 제목, Stage, CEFR, 진도) |
| `StageFilter` | `{ onFilter }` | Stage/CEFR/카테고리/수능유형/모드 필터 |
| `StageMapVisual` | `{ stages, currentStage, lockedStages }` | 7-Stage 맵 (잠금/해금 표시) (v2.0 확장) |
| `TimedModeBar` | `{ timeLimit, onTimeUp }` | 수능 실전 타이머 바 (v2.0) |
| `RelatedTexts` | `{ currentText, relatedTexts }` | Stage 6↔7 토픽 연결 패널 (v2.0) |
| `IllustrationViewer` | `{ illustrations, paragraphIndex? }` | 삽화 렌더러 (top/inline) (v2.0) |
| `WordBankButton` | `{ word, textId, onAdd }` | "단어장에 추가" 버튼 (v2.0) |
| `DimensionRadar` | `{ scores }` | Recharts 레이더 차트 |
| `ProgressLineChart` | `{ data: { date, score }[] }` | 이해도 추이 라인 차트 |
| `StreakCalendar` | `{ dates }` | GitHub 스타일 학습 캘린더 |
| `GrammarHighlights` | `{ tags }` | 문법 구조 목록 (예문 + 설명) |
| `AchievementShelf` | `{ achievements }` | 대시보드 뱃지 진열대 (v2.0) |
| `BadgeToast` | `{ achievement, onClose }` | 뱃지 획득 토스트 알림 (v2.0) |
| `PlacementQuiz` | `{ onComplete }` | 배치 테스트 10문항 UI (v2.0) |
| `ClassManager` | `{ classes, onCreateClass }` | 학급 생성/초대코드 관리 (v2.0) |
| `AssignmentCreator` | `{ classId, texts, onAssign }` | 과제 생성 폼 (v2.0) |
| `SubmissionTracker` | `{ assignment, submissions }` | 과제 제출 현황 테이블 (v2.0) |
| `SkillAnalysisChart` | `{ skillData }` | skill별 정답률 바 차트 (v2.0) |
| `ReportExport` | `{ classId, format }` | PDF/CSV 내보내기 버튼 (v2.0) |

---

## 9. Lib 모듈 명세

### 9.1 lib/constants.ts

```typescript
export const STAGES = [1, 2, 3, 4, 5, 6, 7] as const
export type Stage = typeof STAGES[number]

export const STAGE_INFO: Record<Stage, {
  name: string
  nameKo: string
  cefrRange: string
  targetAge: string
  wordCountRange: [number, number]
  sentenceLengthRange: [number, number]
  category: string
}> = {
  1: { name: 'Phonics Readers', nameKo: '파닉스 리더스', cefrRange: 'Pre-A1~A1', targetAge: '초3-4', wordCountRange: [30, 80], sentenceLengthRange: [3, 7], category: 'phonics' },
  2: { name: "Aesop's Fables", nameKo: '이솝이야기', cefrRange: 'A1~A2', targetAge: '초5-6', wordCountRange: [80, 150], sentenceLengthRange: [6, 10], category: 'fable' },
  3: { name: 'Classic Stories', nameKo: '클래식 스토리', cefrRange: 'A2~B1', targetAge: '중1-2', wordCountRange: [150, 300], sentenceLengthRange: [8, 14], category: 'classic' },
  4: { name: 'Great Lives', nameKo: '위인전', cefrRange: 'B1~B1+', targetAge: '중2-3', wordCountRange: [250, 400], sentenceLengthRange: [10, 16], category: 'biography' },
  5: { name: 'Future World & Careers', nameKo: '미래 직업', cefrRange: 'B1+~B2', targetAge: '고1-2', wordCountRange: [300, 500], sentenceLengthRange: [12, 18], category: 'career' },
  6: { name: 'CSAT Critical Reading', nameKo: '수능 독해', cefrRange: 'B2~B2+', targetAge: '고2-3', wordCountRange: [400, 700], sentenceLengthRange: [14, 22], category: 'csat' },
  7: { name: 'Academic Discourse', nameKo: '학술 지문', cefrRange: 'C1', targetAge: '대학/성인', wordCountRange: [600, 1200], sentenceLengthRange: [18, 30], category: 'academic' },
}

export const CEFR_LEVELS = ['Pre-A1', 'A1', 'A2', 'B1', 'B1+', 'B2', 'B2+', 'C1'] as const

export const CATEGORIES = ['phonics', 'fable', 'classic', 'biography', 'career', 'csat', 'academic'] as const

export const CSAT_TYPES = [
  'blank_inference', 'sentence_order', 'sentence_insertion',
  'main_idea', 'vocabulary_inference', 'summary_completion', 'long_passage'
] as const

export const CSAT_TYPE_LABELS: Record<string, string> = {
  blank_inference: '빈칸추론',
  sentence_order: '순서배열',
  sentence_insertion: '문장삽입',
  main_idea: '주제/요지',
  vocabulary_inference: '어휘추론',
  summary_completion: '요약문 완성',
  long_passage: '장문독해',
}

export const FIVE_DIMENSIONS = ['semantic', 'contextual', 'form', 'relational', 'pragmatic'] as const

export const DIMENSION_LABELS: Record<string, string> = {
  semantic: 'Semantic (의미)',
  contextual: 'Contextual (문맥)',
  form: 'Form (형태)',
  relational: 'Relational (관계)',
  pragmatic: 'Pragmatic (화용)',
}

// v2.0 추가: Stage 잠금 정책 (G13)
export const STAGE_LOCK_POLICY: Record<Stage, { requiresPrevious: boolean, bypassWithPlacement: boolean }> = {
  1: { requiresPrevious: false, bypassWithPlacement: false },
  2: { requiresPrevious: true, bypassWithPlacement: true },
  3: { requiresPrevious: true, bypassWithPlacement: true },
  4: { requiresPrevious: true, bypassWithPlacement: true },
  5: { requiresPrevious: true, bypassWithPlacement: true },
  6: { requiresPrevious: true, bypassWithPlacement: true },
  7: { requiresPrevious: true, bypassWithPlacement: true },
}
// 배치 테스트로 배정받은 Stage까지는 자동 해금, 그 이후는 순차 진급 필요

// v2.0 추가: 뱃지 정의 (G5)
export const ACHIEVEMENT_DEFINITIONS = [
  { type: 'first_read', label: 'First Reader', icon: '🌟', condition: '첫 텍스트 완료' },
  { type: 'stage_complete_1', label: 'Phonics Master', icon: '🏅', condition: 'Stage 1 완료' },
  { type: 'stage_complete_2', label: 'Fable Explorer', icon: '🏅', condition: 'Stage 2 완료' },
  { type: 'stage_complete_3', label: 'Classic Reader', icon: '🏅', condition: 'Stage 3 완료' },
  { type: 'stage_complete_4', label: 'Biography Fan', icon: '🏅', condition: 'Stage 4 완료' },
  { type: 'stage_complete_5', label: 'Future Thinker', icon: '🏅', condition: 'Stage 5 완료' },
  { type: 'stage_complete_6', label: 'CSAT Warrior', icon: '🏅', condition: 'Stage 6 완료' },
  { type: 'stage_complete_7', label: 'Academic Scholar', icon: '🏅', condition: 'Stage 7 완료' },
  { type: 'streak_7', label: 'Week Streak', icon: '🔥', condition: '7일 연속 학습' },
  { type: 'streak_30', label: 'Month Streak', icon: '💎', condition: '30일 연속 학습' },
  { type: 'vocab_50', label: 'Word Collector', icon: '📚', condition: '단어장 50개' },
  { type: 'vocab_100', label: 'Word Master', icon: '📚', condition: '단어장 100개' },
  { type: 'perfect_score', label: 'Perfect!', icon: '⭐', condition: '이해도 100점' },
] as const

// v2.0 추가: 학년 옵션 (G2 온보딩)
export const GRADE_OPTIONS = [
  '초3', '초4', '초5', '초6',
  '중1', '중2', '중3',
  '고1', '고2', '고3',
  '대학', '성인',
] as const

export const TOPIC_AREAS = [
  { code: 'ai_tech', name: 'AI & Technology', nameKo: 'AI & 기술' },
  { code: 'environment', name: 'Environment & Sustainability', nameKo: '환경 & 지속가능성' },
  { code: 'justice', name: 'Justice & Ethics', nameKo: '정의 & 윤리' },
  { code: 'psychology', name: 'Psychology & Cognitive Science', nameKo: '심리 & 인지과학' },
  { code: 'economy', name: 'Economy & Society', nameKo: '경제 & 사회' },
  { code: 'education', name: 'Education & Language', nameKo: '교육 & 언어' },
  { code: 'culture', name: 'Culture & Communication', nameKo: '문화 & 커뮤니케이션' },
  { code: 'science_philosophy', name: 'Science & Philosophy', nameKo: '과학 & 철학' },
] as const
```

### 9.2 lib/types.ts

```typescript
export type DimensionScores = {
  semantic: number
  contextual: number
  form: number
  relational: number
  pragmatic: number
}

export type StageStats = {
  stage: number
  totalTexts: number
  completedTexts: number
  avgComprehension: number
  avgVocabulary: number
  canPromote: boolean
}

export type ReadingFilter = {
  stage?: number
  category?: string
  cefrLevel?: string
  csatType?: string
  status?: string
  search?: string
  page?: number
  limit?: number
}

// v2.0 추가 타입

export type PlacementQuestion = {
  questionIndex: number
  stageLevel: number
  question: string
  options: string[]
  correctAnswer: string
}

export type AchievementType =
  | 'first_read'
  | `stage_complete_${number}`
  | 'streak_7' | 'streak_30'
  | 'vocab_50' | 'vocab_100' | 'vocab_500'
  | 'perfect_score'

export type ReadingMode = 'practice' | 'timed'

export type SkillAnalysis = {
  skill: string  // literal, inferential, evaluative, creative
  totalQuestions: number
  correctCount: number
  accuracy: number
}

export type StageLockPolicy = {
  stage: number
  locked: boolean
  unlockCondition: string  // e.g. "Stage 2 완료 (70%+)"
}
```

### 9.3 lib/text-analyzer.ts

```typescript
import nlp from 'compromise'

export interface TextAnalysis {
  wordCount: number
  sentenceCount: number
  avgSentenceLength: number
  avgWordLength: number
  uniqueWordCount: number
  lexicalDensity: number
  fleschKincaidGrade: number
  fleschReadingEase: number
  colemanLiauIndex: number
  lexileEstimate: number
  words: string[]
}

export function analyzeText(text: string): TextAnalysis {
  const doc = nlp(text)
  const sentences = doc.sentences().out('array') as string[]
  const words = doc.terms().out('array') as string[]
  const uniqueWords = new Set(words.map(w => w.toLowerCase()))

  const wordCount = words.length
  const sentenceCount = sentences.length
  const avgSentenceLength = wordCount / Math.max(sentenceCount, 1)
  const totalSyllables = words.reduce((sum, w) => sum + countSyllables(w), 0)
  const avgWordLength = totalSyllables / Math.max(wordCount, 1)

  const fkGrade = 0.39 * avgSentenceLength + 11.8 * avgWordLength - 15.59
  const fkEase = 206.835 - 1.015 * avgSentenceLength - 84.6 * avgWordLength

  // Coleman-Liau
  const avgCharsPerWord = words.reduce((s, w) => s + w.length, 0) / Math.max(wordCount, 1)
  const L = avgCharsPerWord * 100  // letters per 100 words
  const S = (sentenceCount / wordCount) * 100  // sentences per 100 words
  const cli = 0.0588 * L - 0.296 * S - 15.8

  return {
    wordCount,
    sentenceCount,
    avgSentenceLength: Math.round(avgSentenceLength * 10) / 10,
    avgWordLength: Math.round(avgWordLength * 10) / 10,
    uniqueWordCount: uniqueWords.size,
    lexicalDensity: Math.round((uniqueWords.size / Math.max(wordCount, 1)) * 100) / 100,
    fleschKincaidGrade: Math.round(Math.max(fkGrade, 0) * 10) / 10,
    fleschReadingEase: Math.round(Math.min(Math.max(fkEase, 0), 100) * 10) / 10,
    colemanLiauIndex: Math.round(Math.max(cli, 0) * 10) / 10,
    lexileEstimate: fkToLexile(fkGrade),
    words: Array.from(uniqueWords),
  }
}

function countSyllables(word: string): number {
  word = word.toLowerCase().replace(/[^a-z]/g, '')
  if (word.length <= 3) return 1
  word = word.replace(/(?:[^laeiouy]es|ed|[^laeiouy]e)$/, '')
  word = word.replace(/^y/, '')
  const matches = word.match(/[aeiouy]{1,2}/g)
  return matches ? matches.length : 1
}

const FK_LEXILE_TABLE: Record<number, number> = {
  1: 190, 2: 420, 3: 520, 4: 620, 5: 730,
  6: 830, 7: 925, 8: 1010, 9: 1050, 10: 1080,
  11: 1100, 12: 1130, 13: 1200
}

function fkToLexile(fkGrade: number): number {
  const grade = Math.round(Math.max(1, Math.min(13, fkGrade)))
  return FK_LEXILE_TABLE[grade] ?? 600
}
```

### 9.4 lib/db.ts (주요 함수 시그니처)

```typescript
// Prisma 클라이언트 + Reading Ladder DB 함수

// ── 텍스트 조회 ──
getTexts(filters: ReadingFilter): Promise<ReadingText[]>
getTextById(id: string): Promise<ReadingText & { paragraphs, vocabulary, grammarTags }>
getTextsByStage(stage: number): Promise<ReadingText[]>

// ── 진도 관리 ──
getProgress(userId: string): Promise<ReadingProgress[]>
getProgressForText(userId: string, textId: string): Promise<ReadingProgress | null>
upsertProgress(userId: string, textId: string, data: Partial<ReadingProgress>): Promise<ReadingProgress>

// ── Stage 통계 ──
getStageStats(userId: string): Promise<StageStats[]>
checkPromotion(userId: string, stage: number): Promise<{ canPromote, missing }>
getCurrentStage(userId: string): Promise<number>

// ── 분석 (교사) ──
getStudentAnalytics(studentId: string): Promise<StudentAnalytics>
getClassAnalytics(teacherId: string, classId: string): Promise<ClassAnalytics>

// ── v2.0 추가: 배치 테스트 ──
savePlacementResult(userId: string, data: { score, responses, assignedStage }): Promise<PlacementResult>
getPlacementResult(userId: string): Promise<PlacementResult | null>

// ── v2.0 추가: 학급/과제 ──
createClass(teacherId: string, name: string, gradeLevel?: string): Promise<Class>
joinClass(studentId: string, inviteCode: string): Promise<ClassMembership>
getClassesForUser(userId: string, role: string): Promise<Class[]>
createAssignment(data: { classId, textId, teacherId, title?, instructions?, dueDate? }): Promise<Assignment>
getAssignmentsForStudent(studentId: string): Promise<Assignment[]>
submitAssignment(assignmentId: string, studentId: string, progressId: string): Promise<AssignmentSubmission>

// ── v2.0 추가: 게이미피케이션 ──
checkAndGrantAchievements(userId: string, trigger: string): Promise<Achievement[]>
getAchievements(userId: string): Promise<Achievement[]>

// ── v2.0 추가: 단어장 ──
addToWordBank(userId: string, word: string, textId?: string): Promise<WordBankEntry>
getWordBank(userId: string, mastered?: boolean): Promise<WordBankEntry[]>
removeFromWordBank(userId: string, word: string): Promise<void>

// ── v2.0 추가: skill 분석 ──
getSkillAnalysis(userId: string, csatType?: string): Promise<SkillAnalysis[]>

// ── v2.0 추가: Stage 잠금 ──
getUnlockedStages(userId: string): Promise<number[]>
getNextRecommendedText(userId: string, stage: number): Promise<ReadingText | null>

// ── 5D 앱 호환 (기존 함수 유지) ──
getUserByEmail(email: string)
saveAssessment(userId, config, responses, totalScore)
updateKnowledgeProfile(userId, responses)
getKnowledgeProfile(userId)
getDashboardStats(userId)
```

---

## 10. 콘텐츠 생성 파이프라인 (CLI)

### 10.1 사용법

```bash
# 단일 텍스트 생성
npm run content:generate -- --stage 2 --title "The Fox and the Grapes" --words 80

# Stage 전체 배치 생성
npm run content:generate -- --stage 2 --batch --config data/stage2-config.json

# 텍스트 분석만
npm run content:analyze -- --file "path/to/text.txt"

# TTS 생성
npm run content:tts -- --text-id abc123

# 전체 검증
npm run content:validate -- --stage 2
```

### 10.2 scripts/generate-content.ts 구조

```typescript
// 8-Step Pipeline
async function generateContent(config: ContentConfig) {
  // Step 1: GPT-4o로 본문 생성
  const text = await generateText(config)

  // Step 2: 자동 분석
  const analysis = analyzeText(text)

  // Step 3: 난이도 검증 (최대 3회 재시도)
  const validated = await validateDifficulty(text, analysis, config)

  // Step 4: Neo4j 어휘 매칭
  const vocabulary = await matchVocabulary(validated.words)

  // Step 5: 문법 태깅 (GPT-4o)
  const grammar = await tagGrammar(validated.text, config.cefrLevel)

  // Step 6: 문제 생성 (GPT-4o)
  const { questions, exercises } = await generateQuestions(validated.text, config)

  // Step 7: TTS 생성
  const audioUrl = await generateTTS(validated.text, config.title)

  // Step 8: DB 저장
  await saveToDatabase({ ...config, ...validated, vocabulary, grammar, questions, exercises, audioUrl })
}
```

---

## 11. 인증 & 미들웨어

### 11.1 middleware.ts

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

const publicRoutes = ['/', '/login', '/signup']
const onboardingRoutes = ['/welcome', '/placement']
const teacherRoutes = ['/console', '/analytics', '/topics', '/content', '/class']

export function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl
  const session = req.cookies.get('auth-session')

  // 공개 라우트 → 통과
  if (publicRoutes.includes(pathname)) {
    if (session && (pathname === '/login' || pathname === '/signup')) {
      return NextResponse.redirect(new URL('/dashboard', req.url))
    }
    return NextResponse.next()
  }

  // 인증 필요 라우트
  if (!session) {
    const url = new URL('/login', req.url)
    url.searchParams.set('from', pathname)
    return NextResponse.redirect(url)
  }

  // v2.0: 온보딩 미완료 사용자 → /welcome 리다이렉트 (G2)
  // 온보딩 라우트 자체는 통과
  if (onboardingRoutes.some(r => pathname.startsWith(r))) {
    return NextResponse.next()
  }
  // session에 onboarded 플래그 확인 (쿠키 payload 또는 별도 체크)
  const onboarded = req.cookies.get('onboarded')
  if (!onboarded || onboarded.value !== 'true') {
    return NextResponse.redirect(new URL('/welcome', req.url))
  }

  // 교사 전용 라우트 — role 체크
  if (teacherRoutes.some(r => pathname.startsWith(`/${r.slice(1)}`))) {
    const role = req.cookies.get('user-role')
    if (!role || role.value !== 'teacher') {
      return NextResponse.redirect(new URL('/dashboard', req.url))
    }
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

### 11.2 인증 흐름

```text
쿠키 기반 (5D 앱과 동일 패턴):

Signup → POST /api/auth/signup → User 생성 → Set cookie → /welcome (v2.0: 온보딩)
Login  → POST /api/auth/signin → 비밀번호 검증 → Set cookie
         → onboarded? /dashboard : /welcome
Logout → POST /api/auth/signout → Clear cookie → /

v2.0 온보딩 플로우:
Signup → /welcome (Step 1: 프로필) → /placement (Step 2: 배치 테스트)
       → /welcome (Step 3: 첫 텍스트 추천) → onboarded=true → /dashboard

AuthProvider (React Context):
  - user 상태 관리 (+ grade, onboarded, currentStage)
  - signIn(), signUp(), signOut() 메서드
  - 초기 로드 시 GET /api/auth/session
```

---

## 12. 스타일링 & 테마

### 12.1 Tailwind 설정

5D 앱과 **동일한 Tailwind v3 + HSL CSS 변수** 패턴 사용:

- `tailwind.config.ts`: shadcn/ui 표준 설정 (5D 앱과 동일)
- `globals.css`: `:root` + `.dark` CSS 변수 (HSL)
- 다크모드: `next-themes` + `class` 전략

### 12.2 Stage별 컬러 팔레트

```css
/* globals.css에 추가 */
:root {
  --stage-1: 45 93% 58%;    /* warm yellow — phonics */
  --stage-2: 25 95% 53%;    /* orange — fables */
  --stage-3: 142 71% 45%;   /* green — classic */
  --stage-4: 199 89% 48%;   /* blue — biography */
  --stage-5: 262 83% 58%;   /* purple — career */
  --stage-6: 346 77% 50%;   /* red — CSAT */
  --stage-7: 220 14% 35%;   /* slate — academic */
}
```

---

## 13. 배포 설정

### 13.1 next.config.mjs

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  eslint: { ignoreDuringBuilds: true },
  typescript: { ignoreBuildErrors: true },
  images: { unoptimized: true },
}

export default nextConfig
```

### 13.2 Vercel 배포

```bash
# Vercel에 배포
vercel --prod

# 환경 변수 설정 (Vercel Dashboard 또는 CLI)
vercel env add DATABASE_URL
vercel env add DIRECT_URL
vercel env add OPENAI_API_KEY
vercel env add NEXT_PUBLIC_SUPABASE_URL
vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY
vercel env add SUPABASE_SERVICE_ROLE_KEY
vercel env add VOCAB_API_URL        # ← Neo4j API는 로컬 전용, 배포 시 별도 호스팅 필요
```

> **주의:** Neo4j는 로컬 Desktop 인스턴스. 프로덕션에서는 Neo4j AuraDB (클라우드) 또는 vocab-knowledge-graph API를 별도 서버에 배포해야 함.

---

## 14. 개발 순서 (Sprint Plan) — v2.0 개선사항 반영

> v2.0에서 추가된 항목은 `[v2]`로 표시. 해당 UX Gap 번호(G1~G23)도 함께 기재.

### Sprint 1 (Week 1-2): 프로젝트 셋업 + DB + 온보딩

```text
□ 프로젝트 생성 (create-next-app + 의존성 설치)
□ Supabase 프로젝트 생성 (Free tier, Tokyo region)
□ Prisma 스키마 작성 → prisma migrate dev (v2.0 모델 포함)
□ lib/db.ts 기본 함수 구현 (getTexts, getTextById)
□ lib/neo4j.ts 구현 + Neo4j API 테스트
□ 인증 (auth API + middleware + AuthProvider)
□ 기본 레이아웃 (Header, Sidebar, ThemeProvider)
□ [v2] 배치 테스트 UI + API (/placement, PlacementQuiz) — G1
□ [v2] 온보딩 플로우 (/welcome, 3단계) — G2
□ [v2] 미들웨어 온보딩 리다이렉트 로직 — G2
□ [v2] User 모델 확장 (grade, onboarded, currentStage) — G2
```

### Sprint 2 (Week 3-4): 콘텐츠 파이프라인 + 시드 + 삽화

```text
□ lib/text-analyzer.ts 구현
□ lib/vocab-profiler.ts 구현 (BNC/COCA 데이터 준비)
□ lib/prompts.ts (Stage별 7종 프롬프트)
□ scripts/generate-content.ts (8-Step 파이프라인)
□ POC: Stage 2 이솝이야기 3편 생성 → DB 저장 → 검증
□ scripts/seed.ts (StagePromotion + Achievement 초기 데이터)
□ [v2] TextIllustration 모델 + 삽화 생성 파이프라인 — G9
□ [v2] 콘텐츠 파이프라인에 orderIndex 자동 할당 — G7
```

### Sprint 3 (Week 5-6): 핵심 화면 + Pre/While/Post + 게이미피케이션

```text
□ /reading (텍스트 브라우저 + 필터 + 순서 번호 표시)
□ [v2] /reading/[id] Pre/While/Post 3탭 (ReadingTabs) — G8
□ [v2] Pre-reading 탭 (삽화 예측 + 어휘 미리듣기) — G8
□ /reading/[id] While-reading (TextViewer + VocabPopup + AudioPlayer)
□ [v2] VocabPopup에 +단어장, ↗5D앱 버튼 추가 — G6, G16
□ [v2] ListenMode (Listen First → Read Along → Read Alone) — G10
□ [v2] IllustrationViewer (문단별 삽화 렌더링) — G9
□ ComprehensionQuiz (오답별 해설 + 근거 문장 하이라이트) — G8 확장
□ VocabExercises 컴포넌트
□ [v2] /reading/[id]/complete 완료 축하 화면 — G5, G12
□ [v2] 뱃지 시스템 (Achievement API + BadgeToast) — G5
□ [v2] "다음 텍스트" 연속 동선 (CompletionScreen) — G12
□ /api/progress (진도 저장/조회 + 재시도 필드)
□ /api/reading (텍스트 목록 API + 필터)
□ [v2] /api/word-bank (단어장 CRUD) — G16
□ [v2] /api/achievements (뱃지 확인 + 부여) — G5
```

### Sprint 4 (Week 7-8): 대시보드 + Stage + 잠금/진급

```text
□ /dashboard (통계 카드 + 5D 레이더 + 추천)
□ [v2] AchievementShelf (대시보드 뱃지 진열대) — G5
□ [v2] AssignmentBanner (과제 알림 배너) — G4
□ /stage-map (7-Stage 시각 맵)
□ [v2] Stage 잠금/해금 표시 (StageLockPolicy 적용) — G13
□ [v2] 진급 세레모니 전체 화면 (PromotionCeremony) — G12
□ /api/stage (Stage 통계 + 진급 체크 + 잠금 해제)
□ /progress (상세 진도 페이지)
□ Stage 진급 로직 구현 + 자동 뱃지 부여
□ [v2] /word-bank 개인 단어장 페이지 — G16
□ [v2] Stage 6→7 토픽 연결 UI (RelatedTexts) — G17
```

### Sprint 5 (Week 9-10): 콘텐츠 대량 생성

```text
□ Stage 1 Phonics 20편 생성 + 검수
□ Stage 2 Aesop 20편 생성 + 검수
□ Bridge 1→2, 2→3 텍스트 생성
□ TTS 음성 파일 생성 + Storage 업로드
□ [v2] 각 텍스트별 삽화 생성 (AI 또는 스톡) — G9
□ [v2] 배치 테스트 문항 시드 (Stage 1-7 각 1-2문항) — G1
```

### Sprint 6 (Week 11-12): 교사 기능 + 학급/과제

```text
□ [v2] Class 모델 + 학급 생성/초대코드 (ClassManager) — G3
□ [v2] ClassMembership + 학생 초대 코드 참가 — G3
□ [v2] Assignment 모델 + 과제 생성 (AssignmentCreator) — G4
□ [v2] AssignmentSubmission + 제출 현황 (SubmissionTracker) — G4
□ [v2] /class/[id] 학급 상세 페이지 — G3
□ [v2] /class/[id]/assignments 과제 관리 페이지 — G4
□ [v2] /assignments 학생 과제 목록 페이지 — G4
□ /console (교사 콘솔 — 학급 기반으로 재설계)
□ /analytics (학급/개인 분석)
□ /topics (토픽 관리 CRUD)
□ /api/analytics (SQL 집계 쿼리)
□ 교사 역할 권한 체크 (미들웨어 + RLS)
```

### Sprint 7 (Week 13-14): 분석 강화 + 수능 모드

```text
□ [v2] skill별 오답 패턴 분석 (SkillAnalysisChart) — G15
□ [v2] /api/analytics/skill-analysis — G15
□ [v2] 수능 실전 모드 타이머 (TimedModeBar) — G14
□ [v2] readingMode 'practice' vs 'timed' 구분 — G14
□ [v2] 리포트 PDF/CSV 내보내기 (ReportExport) — G18
□ [v2] /api/analytics/export — G18
□ Stage 3-4 콘텐츠 생성 시작
```

### Sprint 8~ (Week 15+): 콘텐츠 확장 + 최적화

```text
□ Stage 5-7 콘텐츠 점진적 생성 (READING_TEXT_DB_PLAN.md 참조)
□ 어휘 연속성 검증 (cross-stage audit)
□ 성능 최적화 (ISR, 캐싱)
□ AI 에이전트 연동 API (5D 앱 핸드오프 — G6)
□ E2E 테스트
□ [v2 Future] 읽기 속도(WPM) 측정 — G19
□ [v2 Future] 텍스트 내 하이라이트/메모 — G20
□ [v2 Future] 어휘 복습 SRS — G21
□ [v2 Future] 리더보드 — G22
□ [v2 Future] 오프라인 PWA — G23
```

---

## 부록: 문서 간 참조 관계

```
APP_DEV_PRD.md (이 문서, v2.0)
  ├── 참조: PRD.md ─── 제품 요구사항, 아키텍처 결정 근거
  ├── 참조: READING_TEXT_DB_PLAN.md ─── 200편 콘텐츠 상세 (Stage별 목록)
  ├── 참조: TECH_REVIEW.md ─── 기술 검토, 리스크, 비용
  ├── 참조: UX_SIMULATION_REVIEW.md ─── 23개 UX 갭 분석 (v2.0 근거)
  └── 참조: 5dimension_vocablearning ─── 동일 패턴 (Prisma, Auth, shadcn/ui)
```

---

*App Development PRD v2.0 — 2026-02-28 (UX Simulation 23개 개선사항 반영)*
