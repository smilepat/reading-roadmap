# EFL Reading Ladder — App Development PRD

> **목적:** 개발자가 이 문서만으로 앱을 구축할 수 있는 실행 가능한 명세서
> **버전:** 1.0 | **작성일:** 2026-02-28
> **선행 문서:** PRD.md (제품 요구사항), TECH_REVIEW.md (기술 검토), READING_TEXT_DB_PLAN.md (콘텐츠 설계)
> **참조 앱:** 5dimension_vocablearning (동일 패턴 적용)

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
│   ├── (main)/                       # 메인 앱 그룹 (사이드바 레이아웃)
│   │   ├── layout.tsx                # 사이드바 + 메인 영역
│   │   ├── dashboard/page.tsx        # 학생 대시보드
│   │   ├── stage-map/page.tsx        # 7-Stage 시각 맵
│   │   ├── reading/
│   │   │   ├── page.tsx              # 텍스트 브라우저 (목록)
│   │   │   └── [id]/
│   │   │       ├── page.tsx          # Reading Viewer (본문 + 어휘 + 문제)
│   │   │       └── loading.tsx
│   │   ├── vocabulary/page.tsx       # 어휘 학습 현황
│   │   └── progress/page.tsx         # 학습 진도 상세
│   │
│   ├── (teacher)/                    # 교사 전용 그룹
│   │   ├── layout.tsx                # 교사 전용 레이아웃
│   │   ├── console/page.tsx          # 교사 콘솔 (학급 관리)
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
│       ├── analytics/
│       │   ├── student/[id]/route.ts # GET: 개인 분석
│       │   └── class/route.ts        # GET: 학급 분석
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
│   ├── reading/
│   │   ├── text-viewer.tsx           # 본문 렌더러 (문단별, 어휘 하이라이트)
│   │   ├── vocab-popup.tsx           # 어휘 팝오버 (5D 레이더 미니)
│   │   ├── audio-player.tsx          # TTS 오디오 플레이어
│   │   ├── comprehension-quiz.tsx    # 이해도 문제 인터페이스
│   │   ├── vocab-exercises.tsx       # 어휘 연습 인터페이스
│   │   ├── text-card.tsx             # 텍스트 목록 카드
│   │   ├── stage-filter.tsx          # Stage/CEFR/카테고리 필터
│   │   └── grammar-highlights.tsx    # 문법 구조 하이라이트
│   │
│   ├── stage/
│   │   ├── stage-map-visual.tsx      # 7-Stage 시각 맵 (SVG/Canvas)
│   │   ├── stage-progress-ring.tsx   # Stage별 진도 원형 차트
│   │   └── promotion-badge.tsx       # 진급 알림 뱃지
│   │
│   ├── dashboard/
│   │   ├── reading-stats.tsx         # 읽기 통계 카드
│   │   ├── streak-calendar.tsx       # 학습 스트릭 캘린더
│   │   ├── dimension-radar.tsx       # 5D 레이더 차트 (Recharts)
│   │   ├── recent-readings.tsx       # 최근 읽은 텍스트
│   │   └── recommended-texts.tsx     # 추천 텍스트
│   │
│   ├── teacher/
│   │   ├── class-overview.tsx        # 학급 전체 현황
│   │   ├── student-detail.tsx        # 개인 상세 분석
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
  id        String   @id @default(cuid())
  email     String   @unique
  password  String
  name      String?
  role      String   @default("student") // student | teacher | admin
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt      @map("updated_at")

  assessments      Assessment[]
  knowledgeProfile KnowledgeProfile?
  userVocabulary   UserVocabulary[]
  readingProgress  ReadingProgress[]

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

  // 브릿지
  isBridge        Boolean @default(false) @map("is_bridge")
  bridgeFromStage Int?    @map("bridge_from_stage")
  bridgeToStage   Int?    @map("bridge_to_stage")

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt      @map("updated_at")

  paragraphs  TextParagraph[]
  vocabulary  TextVocabulary[]
  grammarTags TextGrammarTag[]
  questions   TextQuestion[]
  exercises   TextExercise[]
  progress    ReadingProgress[]

  @@index([stage])
  @@index([cefrLevel])
  @@index([category])
  @@index([csatType])
  @@index([status])
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
  status             String    @default("not_started")
  startedAt          DateTime? @map("started_at")
  completedAt        DateTime? @map("completed_at")
  comprehensionScore Int?      @map("comprehension_score")
  vocabularyScore    Int?      @map("vocabulary_score")
  timeSpent          Int       @default(0) @map("time_spent")
  readCount          Int       @default(0) @map("read_count")
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

### 6.6 Analytics API (교사용)

| Endpoint | Method | Response | 비고 |
|----------|--------|----------|------|
| `/api/analytics/student/[id]` | GET | 개인 Stage별 진도, 5D 프로필, 어휘 숙달도 | 교사 권한 |
| `/api/analytics/class` | GET | 학급 평균, Stage 분포, 약점 차원 | SQL 집계 |

### 6.7 콘텐츠 관리 API (admin)

| Endpoint | Method | 비고 |
|----------|--------|------|
| `/api/content/generate` | POST | 콘텐츠 생성 파이프라인 트리거 |
| `/api/content/validate` | POST | 텍스트 난이도 검증 |
| `/api/topics` | GET/POST/PUT/DELETE | 사용자 토픽 CRUD |
| `/api/tts` | POST | TTS 생성 요청 → Supabase Storage |

---

## 7. 페이지 라우트 & 화면 명세

### 7.1 랜딩 페이지 — `/`

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

### 7.4 Reading Viewer — `/reading/[id]`

```
┌─────────────────────────────────────────────────────────┐
│  ← Back    "Treasure Island"   🔊 Play Audio   Stage 3 │
│  Classic Stories > Adventure     CEFR A2-B1   260 words │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─── Text ──────────────────────┬── Vocabulary ──────┐ │
│  │                               │                     │ │
│  │  [삽화]                       │ 학습 어휘 (8개)     │ │
│  │                               │                     │ │
│  │  Jim Hawkins found an old     │ treasure /ˈtreʒ.ər/ │ │
│  │  map in the sea captain's     │  보물 (n.)          │ │
│  │  chest. The [map] showed the  │  CEFR: A2           │ │
│  │  location of a hidden         │  5D: Semantic       │ │
│  │  [treasure] on a faraway      │  🔊  ↗ Neo4j       │ │
│  │  island.                      │                     │ │
│  │                               │ faraway /ˌfɑːrəˈweɪ/│ │
│  │  ¶ Key idea: Jim finds a      │  먼, 멀리 떨어진    │ │
│  │    treasure map.              │  CEFR: A2           │ │
│  │                               │  5D: Contextual     │ │
│  │  He showed the map to         │                     │ │
│  │  Doctor Livesey and           │ ...                  │ │
│  │  Squire Trelawney. They       │                     │ │
│  │  decided to sail to the       │                     │ │
│  │  island...                    │                     │ │
│  │                               │                     │ │
│  └───────────────────────────────┴─────────────────────┘ │
│                                                         │
│  ┌─── Grammar Highlights ────────────────────────────┐  │
│  │ past simple: "found", "showed", "decided"   [A2]  │  │
│  │ to-infinitive: "decided to sail"            [A2]  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─── Comprehension Check (0/5) ─────────────────────┐  │
│  │ Q1. What did Jim find in the chest?               │  │
│  │   ○ A sword  ○ A map  ○ Gold  ○ A letter         │  │
│  │   [Submit →]                                      │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─── Vocabulary Exercises (0/4) ────────────────────┐  │
│  │ Fill in the blank:                                │  │
│  │ "The pirate hid the ______ on the island."        │  │
│  │   [treasure]  [map]  [ship]  [chest]              │  │
│  └───────────────────────────────────────────────────┘  │
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
| `TextViewer` | `{ text: ReadingText, paragraphs: TextParagraph[], vocabulary: TextVocabulary[] }` | 본문 렌더링, 어휘 하이라이트, 문단별 key idea 표시 |
| `VocabPopup` | `{ word: string, vocab: TextVocabulary, fiveD?: FiveDProfile }` | 어휘 클릭 시 팝오버 (발음, 뜻, 5D 미니 레이더, 관련어) |
| `AudioPlayer` | `{ audioUrl: string, title: string }` | TTS 재생/일시정지, 속도 조절 |
| `ComprehensionQuiz` | `{ questions: TextQuestion[], onComplete: (score) => void }` | 문제 풀기 UI, 즉시 피드백, 점수 계산 |
| `VocabExercises` | `{ exercises: TextExercise[], onComplete: (score) => void }` | 어휘 연습 UI (빈칸, 매칭 등) |
| `TextCard` | `{ text: ReadingText, progress?: ReadingProgress }` | 목록 카드 (제목, Stage, CEFR, 진도) |
| `StageFilter` | `{ onFilter: (filters) => void }` | Stage/CEFR/카테고리/수능유형 필터 |
| `StageMapVisual` | `{ stages: StageStats[], currentStage: number }` | 7-Stage 수직 맵 시각화 |
| `DimensionRadar` | `{ scores: DimensionScores }` | Recharts 레이더 차트 |
| `ProgressLineChart` | `{ data: { date, score }[] }` | 이해도 추이 라인 차트 |
| `StreakCalendar` | `{ dates: string[] }` | GitHub 스타일 학습 캘린더 |
| `GrammarHighlights` | `{ tags: TextGrammarTag[] }` | 문법 구조 목록 (예문 + 설명) |

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
getClassAnalytics(teacherId: string): Promise<ClassAnalytics>

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
const teacherRoutes = ['/console', '/analytics', '/topics', '/content']

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

  // 교사 전용 라우트 (추후 role 체크 추가)
  // if (teacherRoutes.some(r => pathname.startsWith(r))) { ... }

  return NextResponse.next()
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

### 11.2 인증 흐름

```
쿠키 기반 (5D 앱과 동일 패턴):

Signup → POST /api/auth/signup → User 생성 → Set cookie → /dashboard
Login  → POST /api/auth/signin → 비밀번호 검증 → Set cookie → /dashboard
Logout → POST /api/auth/signout → Clear cookie → /

AuthProvider (React Context):
  - user 상태 관리
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

## 14. 개발 순서 (Sprint Plan)

### Sprint 1 (Week 1-2): 프로젝트 셋업 + DB

```
□ 프로젝트 생성 (create-next-app + 의존성 설치)
□ Supabase 프로젝트 생성 (Free tier, Tokyo region)
□ Prisma 스키마 작성 → prisma migrate dev
□ lib/db.ts 기본 함수 구현 (getTexts, getTextById)
□ lib/neo4j.ts 구현 + Neo4j API 테스트
□ 인증 (auth API + middleware + AuthProvider)
□ 기본 레이아웃 (Header, Sidebar, ThemeProvider)
```

### Sprint 2 (Week 3-4): 콘텐츠 파이프라인 + 시드

```
□ lib/text-analyzer.ts 구현
□ lib/vocab-profiler.ts 구현 (BNC/COCA 데이터 준비)
□ lib/prompts.ts (Stage별 7종 프롬프트)
□ scripts/generate-content.ts (8-Step 파이프라인)
□ POC: Stage 2 이솝이야기 3편 생성 → DB 저장 → 검증
□ scripts/seed.ts (StagePromotion 초기 데이터)
```

### Sprint 3 (Week 5-6): 핵심 화면

```
□ /reading (텍스트 브라우저 + 필터)
□ /reading/[id] (Reading Viewer + VocabPopup + AudioPlayer)
□ ComprehensionQuiz + VocabExercises 컴포넌트
□ /api/progress (진도 저장/조회)
□ /api/reading (텍스트 목록 API + 필터)
```

### Sprint 4 (Week 7-8): 대시보드 + Stage

```
□ /dashboard (통계 카드 + 5D 레이더 + 추천)
□ /stage-map (7-Stage 시각 맵)
□ /api/stage (Stage 통계 + 진급 체크)
□ /progress (상세 진도 페이지)
□ Stage 진급 로직 구현
```

### Sprint 5 (Week 9-10): 콘텐츠 대량 생성

```
□ Stage 1 Phonics 20편 생성 + 검수
□ Stage 2 Aesop 20편 생성 + 검수
□ Bridge 1→2, 2→3 텍스트 생성
□ TTS 음성 파일 생성 + Storage 업로드
```

### Sprint 6 (Week 11-12): 교사 기능 + 분석

```
□ /console (교사 콘솔)
□ /analytics (학급/개인 분석)
□ /topics (토픽 관리 CRUD)
□ /api/analytics (SQL 집계 쿼리)
□ 교사 역할 권한 체크
```

### Sprint 7~ (Week 13+): 콘텐츠 확장 + 최적화

```
□ Stage 3-7 콘텐츠 점진적 생성 (READING_TEXT_DB_PLAN.md 참조)
□ 어휘 연속성 검증 (cross-stage audit)
□ 성능 최적화 (ISR, 캐싱)
□ AI 에이전트 연동 API
□ E2E 테스트
```

---

## 부록: 문서 간 참조 관계

```
APP_DEV_PRD.md (이 문서)
  ├── 참조: PRD.md ─── 제품 요구사항, 아키텍처 결정 근거
  ├── 참조: READING_TEXT_DB_PLAN.md ─── 200편 콘텐츠 상세 (Stage별 목록)
  ├── 참조: TECH_REVIEW.md ─── 기술 검토, 리스크, 비용
  └── 참조: 5dimension_vocablearning ─── 동일 패턴 (Prisma, Auth, shadcn/ui)
```

---

*App Development PRD v1.0 — 2026-02-28*
