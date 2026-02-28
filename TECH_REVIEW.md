# Reading Text DB 계획 — 기술 구현 검토서

> 검토일: 2026-02-28
> 대상: READING_TEXT_DB_PLAN.md
> 기존 프로젝트 현황 대조 포함

---

## 종합 판정

| 영역 | 상태 | 요약 |
|------|------|------|
| DB 스키마 설계 | ⚠️ 수정 필요 | 단일 도큐먼트 모델 → 정규화 분리 필요 |
| 기존 시스템 연동 | 🔴 갭 존재 | Supabase 미활성, Neo4j API 미테스트, 5D 앱 어휘소스 부재 |
| 텍스트 분석 파이프라인 | ⚠️ 도구 검증 필요 | Lexile 자동 계산 불가, 대안 필요 |
| 콘텐츠 생산 워크플로 | ⚠️ 병목 예상 | 200편 × 메타데이터 = 수작업 폭발, LLM 자동화 전략 구체화 필요 |
| 일정 | 🔴 비현실적 | 18주 → 최소 28-32주로 조정 필요 |
| 프론트엔드 | ✅ 양호 | Next.js + shadcn/ui 기존 패턴 활용 가능 |
| 5D 연동 모델 | ✅ 설계 양호 | 개념적으로 잘 맞음, 구현 경로만 명확화 필요 |

---

## 1. DB 스키마 — 구조적 문제

### 문제 1-1: 과도한 비정규화 (Denormalization)

현재 `ReadingText` 인터페이스에 **모든 것이 하나의 도큐먼트**에 들어 있음:

```typescript
interface ReadingText {
  // 메타데이터 + 본문 + 난이도 + 어휘[] + 문법[] + 문제[] + 연습[] + 5D + ...
}
```

**문제점:**
- `targetVocabulary: VocabItem[]` — Stage 6 한 편에 15-20개 어휘 × 각 10+ 필드 = 단일 레코드가 거대해짐
- `comprehensionQuestions: Question[]` — 편당 5-8문항 네스팅
- `vocabularyExercises: Exercise[]` — 추가 네스팅
- Supabase(PostgreSQL)에서 JSONB로 저장 시: 쿼리 성능 저하, 인덱싱 불가, 부분 업데이트 비효율

**수정안: 정규화 분리**

```
reading_texts          ← 본문 + 메타데이터 + 난이도 지표
├── text_paragraphs    ← 문단별 분리 (1:N)
├── text_vocabulary    ← 텍스트-어휘 매핑 (M:N, join table)
│   └── vocabulary     ← 어휘 마스터 테이블 (Neo4j 연동)
├── text_grammar_tags  ← 문법 태그 (1:N)
├── text_questions     ← 이해도 문제 (1:N)
│   └── question_options ← 선택지 (1:N)
├── text_exercises     ← 어휘 연습 (1:N)
│   └── exercise_items ← 연습 항목 (1:N)
└── text_5d_mapping    ← 5D 차원별 어휘 매핑 (1:N)
```

**이유:**
- 어휘는 여러 텍스트에 재등장함 → M:N 관계 필수
- 문제/연습은 독립적으로 CRUD 가능해야 함
- 5D 매핑은 기존 Neo4j 어휘 그래프와 JOIN 필요

### 문제 1-2: 어휘 마스터 테이블 이중화 위험

계획서의 `VocabItem`과 기존 Neo4j vocab-knowledge-graph의 Word 노드가 **같은 데이터를 다른 곳에 저장**:

| 필드 | 계획서 VocabItem | Neo4j Word 노드 |
|------|-----------------|----------------|
| word | ✅ | ✅ |
| POS | ✅ | ✅ |
| CEFR | ✅ | ✅ |
| definition | ✅ | ✅ |
| 한국어 뜻 | ✅ definitionKo | ✅ (CSV에 있음) |
| 발음 | ✅ phonetic | ✅ |
| 5D 차원 | ✅ dimension (단일) | ✅ (복수 관계) |

**위험:** 두 데이터 소스가 동기화되지 않으면 불일치 발생

**수정안:**
- 어휘 마스터 데이터 = Neo4j (Single Source of Truth)
- ReadingText DB는 `vocabulary_id` (Neo4j Word 노드 ID)로 **참조만** 저장
- 텍스트별 컨텍스트 데이터(exampleSentence 등)만 ReadingText 쪽에 저장

```typescript
// 수정된 text_vocabulary join table
interface TextVocabulary {
  textId: string
  vocabularyId: string          // → Neo4j Word 노드 참조
  exampleSentence: string       // 이 텍스트 내 예문 (텍스트 고유)
  dimension: FiveDDimension     // 이 텍스트에서의 5D 학습 포커스
  orderInText: number           // 등장 순서
}
```

### 문제 1-3: `vocabularyLevel` 계산 타이밍

```typescript
vocabularyLevel: {
  k1Percentage: number   // 고빈도 1,000단어 비율
  k2Percentage: number
  // ...
}
```

이 값들은 **정적 저장이 아닌 동적 계산**이어야 함:
- K1/K2/K3 빈도 목록 자체가 업데이트될 수 있음
- 텍스트 수정 시 자동 재계산 필요

**수정안:** 저장하되, `content` 변경 시 트리거로 재계산하는 파이프라인 구축. 또는 computed column / materialized view 활용.

---

## 2. 기존 시스템 연동 — 현실 갭 분석

### 갭 2-1: Supabase가 실제로 미활성 상태

계획서는 "Supabase (PostgreSQL)"을 DB로 명시하지만, 현재 상태:

| 프로젝트 | Supabase 상태 |
|----------|--------------|
| 5D 어휘학습 앱 | `.env`에 placeholder URL만 존재, 실제 SQLite+Prisma 사용 중 |
| Vocab Knowledge Graph | Supabase 설정 없음, 순수 Neo4j |
| Reading Roadmap | 아직 프로젝트 없음 |

**영향:**
- Phase 1 "Supabase 테이블 생성"은 **Supabase 프로젝트 자체를 먼저 생성**해야 함
- 5D 앱의 user/assessment 데이터가 SQLite에 있으므로 **마이그레이션** 필요
- 또는 SQLite → Supabase 전환을 선행해야 3개 앱이 공통 DB 사용 가능

**선택지:**

| 옵션 | 장점 | 단점 |
|------|------|------|
| A) Supabase 올인 | 공통 DB, 인증, RLS | 마이그레이션 비용, 5D 앱 수정 |
| B) SQLite 유지 + API 분리 | 기존 코드 변경 최소 | 앱 간 데이터 공유 불가 |
| C) Prisma + Supabase Postgres | Prisma 유지하면서 원격 DB | 설정 변경만으로 전환 가능 |

**권장: 옵션 C** — 5D 앱의 Prisma 스키마는 유지하되 `datasource`만 Supabase Postgres로 변경. Reading Text DB도 같은 Supabase 인스턴스에 테이블 추가.

### 갭 2-2: Neo4j API 서버 미테스트

Vocab Knowledge Graph의 API 서버(`apps/api/`)가 존재하지만 **한 번도 테스트되지 않음**.

현재 API 엔드포인트:
- `/api/vocab/[word]` — 단어 조회
- `/api/vocab/search` — 검색
- `/api/vocab/related` — 관련어
- `/api/vocab/5d-profile` — 5D 프로파일
- `/api/graph/stats` — 통계
- `/api/graph/network` — 네트워크
- `/api/graph/cefr-distribution` — CEFR 분포

**Reading Text DB 연동에 필수적인 API:**
- 텍스트 내 어휘의 5D 프로파일 일괄 조회
- K1/K2/K3 빈도 레벨 조회
- 동의어/반의어/연어 데이터 (관계 어휘 문제 생성용)

**선행 작업:**
1. Neo4j API 서버 기동 및 전 엔드포인트 테스트
2. 배치 조회 API 추가 (`POST /api/vocab/batch` — 단어 목록 일괄 조회)
3. 어휘 빈도 레벨 API 추가 (BNC/COCA 기반)

### 갭 2-3: 5D 앱에 어휘 데이터 소스 부재

5D 어휘학습 앱의 Prisma 스키마에 `Vocabulary` 모델이 있지만, **실제 어휘 데이터가 비어 있음**. 앱 내에서 어휘 퀴즈를 출제하려면 데이터 소스가 필요.

```
5D 앱 ←(현재 없음)→ 어휘 데이터
5D 앱 ←(계획됨)→ Neo4j API ← 9,000 단어
5D 앱 ←(새로 추가)→ Reading Text DB ← 텍스트 내 학습 어휘
```

**Reading Text DB는 5D 앱의 어휘 데이터 소스 역할도 겸해야** 함 → API 설계에 반영 필요.

---

## 3. 텍스트 분석 파이프라인 — 도구 검증

### 문제 3-1: Lexile 자동 계산 불가

계획서: `text-readability` (Lexile/FK)

**현실:**
- **Lexile 점수는 MetaMetrics 사의 독점 알고리즘**. 공개 API나 npm 패키지로 정확한 Lexile 계산은 불가능.
- `text-readability` npm 패키지는 Flesch-Kincaid, Gunning Fog, Coleman-Liau 등은 지원하지만 **Lexile은 미지원**.
- MetaMetrics Lexile Analyzer(유료)를 사용하거나, FK/ARI 점수에서 **근사값을 역산**해야 함.

**수정안:**

```typescript
// 계산 가능한 지표 (npm text-readability 또는 직접 구현)
fleschKincaidGrade: number      // ✅ 계산 가능
fleschReadingEase: number       // ✅ 계산 가능
colemanLiauIndex: number        // ✅ 계산 가능
automatedReadabilityIndex: number // ✅ 계산 가능

// 근사값 변환
lexileEstimate: number          // FK Grade → Lexile 근사 변환 테이블 사용
lexileSource: 'estimated' | 'official'  // 출처 표기
```

**FK → Lexile 근사 변환식** (연구 기반 회귀 모델):
```
Lexile ≈ FK_Grade × 130 - 100  (매우 대략적)
```
더 정확한 매핑은 MetaMetrics 연구 데이터를 참조하여 look-up table 구성.

### 문제 3-2: `compromise` NLP의 한계

계획서: `compromise` (NLP)

- `compromise`는 영어 텍스트의 토크나이징, POS 태깅, 문장 분리에 적합
- 그러나 **dependency parsing(의존 구문 분석)**은 불가 → 문법 구조 자동 태깅에 한계
- 예: "관계대명사 which가 사용됨"은 감지 가능하지만, "비제한적 관계절"과 "제한적 관계절" 구분은 어려움

**수정안: 2단계 접근**

```
1단계 (자동, 빠름): compromise로 기초 분석
  - 문장 분리, 단어 수, POS 태그
  - 정규식 기반 패턴 매칭 (if절, 수동태, 비교급 등)

2단계 (LLM 보조, 정확): GPT-4o로 문법 정밀 태깅
  - 프롬프트: "이 텍스트에서 CEFR B2 수준의 문법 구조를 식별하세요"
  - 비용: ~$0.01/편 (500 words 기준)
  - 200편 × $0.01 = $2 (무시 가능)
```

### 문제 3-3: 문법 태깅 자동화 범위

계획서의 `SpaCy / Stanford NLP (dependency parsing)`은 **Python 생태계**. 프로젝트는 TypeScript/Next.js 스택.

**선택지:**

| 옵션 | 언어 | 정확도 | 통합 난이도 |
|------|------|--------|------------|
| compromise (JS) | TS/JS | 중 | 쉬움 |
| wink-nlp (JS) | TS/JS | 중-상 | 쉬움 |
| SpaCy Python microservice | Python | 상 | API 분리 필요 |
| LLM 기반 태깅 | API | 상 | 프롬프트 설계 |

**권장:** `compromise` + LLM 하이브리드. SpaCy 마이크로서비스는 운영 복잡성 대비 이득이 적음.

---

## 4. 콘텐츠 생산 파이프라인 — 병목 분석

### 병목 4-1: 200편 × 풍부한 메타데이터 = 엄청난 수작업

각 텍스트 1편에 필요한 작업:

| 작업 | 수작업 여부 | 예상 시간/편 |
|------|-----------|-------------|
| 텍스트 본문 작성 | LLM 초안 + 검수 | 30-60분 |
| 어휘 선정 + 5D 태깅 | 반자동 | 20-30분 |
| 문법 태깅 | LLM + 검수 | 10-15분 |
| 이해도 문제 5개 | LLM 초안 + 검수 | 20-30분 |
| 어휘 연습 문제 | LLM 초안 + 검수 | 15-20분 |
| 한국어 번역/주석 | 수작업 | 15-20분 |
| 삽화 지시/검토 | 수작업 | 10-20분 |
| 음성 파일 생성 | TTS 자동 | 5분 |
| QA 검수 | 수작업 | 15-20분 |
| **합계** | | **~2.5-3.5시간/편** |

**200편 × 3시간 = 600시간 = 풀타임 기준 약 15주**

### 병목 4-2: LLM 콘텐츠 파이프라인 구체화 필요

계획서의 "GPT-4o (초안) + 전문가 검수"를 **자동화 가능한 파이프라인**으로 설계해야 함:

```
입력: { stage, category, title, targetWordCount, cefrLevel, targetVocabulary[] }
  │
  ├─ Step 1: GPT-4o로 본문 생성 (프롬프트 템플릿 stage별 7종)
  ├─ Step 2: 자동 분석 (compromise → 단어 수, 문장 길이, FK 점수)
  ├─ Step 3: 난이도 검증 (목표 CEFR 범위 내인지 자동 체크)
  │   └─ ❌ 벗어남 → GPT-4o에 수정 요청 (최대 3회 반복)
  ├─ Step 4: 어휘 추출 + Neo4j 대조 (기존 Word 노드와 매칭)
  ├─ Step 5: 문법 태깅 (LLM)
  ├─ Step 6: 문제 생성 (LLM, stage별 유형 분기)
  ├─ Step 7: TTS 음성 생성
  └─ Step 8: 전문가 검수 큐에 추가
```

**핵심:** Step 1-7을 자동화하면 편당 **30분 → 검수만 15-20분**으로 단축 가능.

### 병목 4-3: 삽화/이미지 생산

Stage 1-3에서 삽화 비율이 30-80%로 높음. 200편 중 ~70편에 삽화 필요.

**선택지:**
- AI 생성 (DALL-E, Midjourney): 빠르지만 일관성 부족
- 스톡 일러스트: 권당 $5-15 비용
- 커스텀 일러스트레이터: 고비용, 고품질

**권장:** 초기에는 AI 생성으로 프로토타입, 출시 시 전문 일러스트로 교체.

---

## 5. 일정 현실성 — 재산정

### 원래 계획 vs 수정안

| Phase | 원래 | 수정 | 이유 |
|-------|------|------|------|
| **Phase 0 (신규)** | — | **4주** | Supabase 설정, Neo4j API 테스트, 5D 앱 DB 마이그레이션 |
| Phase 1: 기반 구축 | 2주 | **3주** | 정규화 스키마 설계 + 분석 파이프라인 구축 |
| Phase 2: Stage 1-2 | 3주 | **4주** | 40편 작성 + 파이프라인 디버깅 |
| Phase 3: Stage 3-4 | 4주 | **6주** | 60편 (가장 많은 물량) |
| Phase 4: Stage 5-6 | 4주 | **6주** | 60편 + 수능 유형 문제 생성 복잡성 |
| Phase 5: Stage 7 + 통합 | 3주 | **4주** | 20편 + 어휘 연속성 검증 |
| Phase 6: AI 연동 | 2주 | **3주** | Agent 연동 테스트 |
| **합계** | **18주** | **30주** | |

### Phase 0 (선행 작업) 상세

```
Week 1-2:
  - Supabase 프로젝트 생성 + reading_texts 정규화 스키마 마이그레이션
  - 5D 앱 Prisma datasource를 Supabase Postgres로 전환
  - 기존 SQLite 데이터 마이그레이션

Week 3-4:
  - Neo4j vocab-graph API 서버 테스트 + 배치 조회 API 추가
  - Reading Text ↔ Neo4j 어휘 연동 POC (Proof of Concept)
  - 콘텐츠 생성 파이프라인 프로토타입 (Stage 2 이솝이야기 3편으로 테스트)
```

---

## 6. 아키텍처 권장안

### 6.1 데이터 흐름도 (수정)

```
┌─────────────────────────────────────────────────────────────┐
│                    Supabase (PostgreSQL)                     │
│                                                             │
│  reading_texts ──── text_vocabulary ────┐                   │
│  text_paragraphs    text_grammar_tags   │                   │
│  text_questions     text_exercises      │                   │
│  reading_progress   stage_promotions    │                   │
│                                         │ vocabulary_id     │
│  user_profiles ─── knowledge_profiles   │ (FK reference)    │
│  assessments ───── assessment_responses │                   │
│                                         │                   │
└────────────────────────────────────┬────┘                   │
                                     │                        │
                              Supabase Auth                    │
                              (공통 인증)                       │
                                     │                        │
┌────────────────────────────────────┼────────────────────────┘
│                                    │
│    ┌───────────────────────────────┼──────────────────┐
│    │         Next.js API Layer     │                  │
│    │                               │                  │
│    │  /api/reading/*    ───────────┤                  │
│    │  /api/vocab/*      ──────────→│  Neo4j           │
│    │  /api/assessment/* ───────────┤  (9,000 words    │
│    │  /api/analytics/*  ───────────┤   + 50K rels)    │
│    │                               │                  │
│    └───────────────────────────────┘                  │
│                                                       │
│    ┌─────────────────────────────────────────────────┐│
│    │         콘텐츠 생성 파이프라인 (CLI/Script)       ││
│    │                                                 ││
│    │  GPT-4o ─→ compromise ─→ Neo4j 대조 ─→ Supabase ││
│    │    ↑           ↑                                ││
│    │  프롬프트     분석/검증                           ││
│    │  템플릿 7종                                      ││
│    └─────────────────────────────────────────────────┘│
└───────────────────────────────────────────────────────┘
```

### 6.2 Supabase 테이블 스키마 (정규화)

```sql
-- 핵심 테이블
CREATE TABLE reading_texts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stage SMALLINT NOT NULL CHECK (stage BETWEEN 1 AND 7),
  title TEXT NOT NULL,
  subtitle TEXT,
  category TEXT NOT NULL,        -- phonics, fable, classic, biography, career, csat, academic
  subcategory TEXT,
  topic_area TEXT,               -- Stage 6-7
  topic_number SMALLINT,

  content TEXT NOT NULL,
  content_html TEXT,

  cefr_level TEXT NOT NULL,
  lexile_estimate SMALLINT,
  lexile_source TEXT DEFAULT 'estimated',
  flesch_kincaid_grade REAL,
  flesch_reading_ease REAL,

  word_count SMALLINT NOT NULL,
  sentence_count SMALLINT,
  avg_sentence_length REAL,
  unique_word_count SMALLINT,
  lexical_density REAL,

  k1_pct REAL,  k2_pct REAL,  k3_pct REAL,  awl_pct REAL,  off_list_pct REAL,

  csat_type TEXT,                -- Stage 6 only
  csat_difficulty TEXT,

  author TEXT,
  original_work TEXT,
  illustration_ratio REAL,
  audio_url TEXT,
  korean_translation TEXT,

  is_bridge BOOLEAN DEFAULT FALSE,
  bridge_from_stage SMALLINT,
  bridge_to_stage SMALLINT,

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 문단
CREATE TABLE text_paragraphs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id UUID REFERENCES reading_texts(id) ON DELETE CASCADE,
  paragraph_index SMALLINT NOT NULL,
  content TEXT NOT NULL,
  word_count SMALLINT,
  key_idea TEXT,                -- Stage 4+
  UNIQUE(text_id, paragraph_index)
);

-- 텍스트-어휘 매핑 (M:N)
CREATE TABLE text_vocabulary (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id UUID REFERENCES reading_texts(id) ON DELETE CASCADE,
  neo4j_word_id TEXT,           -- Neo4j Word 노드 참조
  word TEXT NOT NULL,           -- 검색/fallback용 중복 저장
  example_sentence TEXT,        -- 이 텍스트 내 예문
  five_d_dimension TEXT,        -- 이 텍스트에서의 학습 포커스 차원
  order_in_text SMALLINT,
  UNIQUE(text_id, word)
);

-- 문법 태그
CREATE TABLE text_grammar_tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id UUID REFERENCES reading_texts(id) ON DELETE CASCADE,
  feature TEXT NOT NULL,         -- past_simple, passive_voice, etc.
  cefr_level TEXT,
  example_from_text TEXT,
  explanation TEXT,
  explanation_ko TEXT
);

-- 이해도 문제
CREATE TABLE text_questions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id UUID REFERENCES reading_texts(id) ON DELETE CASCADE,
  question_type TEXT NOT NULL,   -- multiple_choice, true_false, short_answer, open_ended
  question TEXT NOT NULL,
  question_ko TEXT,
  options JSONB,                 -- 선택지 배열
  correct_answer TEXT NOT NULL,
  explanation TEXT,
  explanation_ko TEXT,
  skill TEXT,                    -- literal, inferential, evaluative, creative
  csat_type TEXT,                -- 수능 유형 연계
  order_index SMALLINT
);

-- 어휘 연습
CREATE TABLE text_exercises (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id UUID REFERENCES reading_texts(id) ON DELETE CASCADE,
  exercise_type TEXT NOT NULL,   -- fill_blank, matching, word_form, synonym_antonym, context_clue
  prompt TEXT NOT NULL,
  answer TEXT NOT NULL,
  distractors JSONB,             -- 오답 선택지 배열
  order_index SMALLINT
);

-- 학습 진도
CREATE TABLE reading_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  text_id UUID REFERENCES reading_texts(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'not_started',
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  comprehension_score SMALLINT,
  vocabulary_score SMALLINT,
  time_spent INTEGER DEFAULT 0,  -- seconds
  read_count SMALLINT DEFAULT 0,
  notes TEXT,
  UNIQUE(user_id, text_id)
);

-- 인덱스
CREATE INDEX idx_reading_texts_stage ON reading_texts(stage);
CREATE INDEX idx_reading_texts_cefr ON reading_texts(cefr_level);
CREATE INDEX idx_reading_texts_category ON reading_texts(category);
CREATE INDEX idx_text_vocabulary_word ON text_vocabulary(word);
CREATE INDEX idx_reading_progress_user ON reading_progress(user_id);
```

---

## 7. 기술 도구 검증 결과

| 계획서 도구 | 검증 결과 | 대안/수정 |
|------------|----------|----------|
| `text-readability` | ⚠️ Lexile 미지원 | FK/ARI/Coleman-Liau는 가능, Lexile은 근사값 |
| `compromise` | ✅ 기본 NLP 충분 | POS, 문장 분리, 토크나이징 OK |
| SpaCy/Stanford NLP | ⚠️ Python, 과잉 | JS 스택에 불필요, LLM으로 대체 |
| BNC/COCA 빈도 목록 | ✅ 사용 가능 | 공개 빈도 목록 다운로드 필요 (CSV 형태) |
| Academic Word List | ✅ 사용 가능 | Coxhead AWL 570 word families |
| GPT-4o | ✅ | 텍스트 생성 + 문법 태깅 + 문제 생성 |
| OpenAI TTS | ✅ | `tts-1-hd` 모델, 6가지 음성 |
| ElevenLabs | ✅ | 더 자연스럽지만 비용 높음 |
| Supabase | ⚠️ 미구축 | 프로젝트 생성부터 시작 |
| Neo4j | ✅ 데이터 있음 | API 테스트 선행 필요 |

---

## 8. 비용 추정

### 콘텐츠 생성 LLM 비용

| 항목 | 단가 | 수량 | 비용 |
|------|------|------|------|
| 본문 생성 (GPT-4o) | ~$0.03/편 | 200편 | $6 |
| 문법 태깅 (GPT-4o) | ~$0.01/편 | 200편 | $2 |
| 문제 생성 (GPT-4o) | ~$0.02/편 | 200편 | $4 |
| 어휘 연습 생성 | ~$0.01/편 | 200편 | $2 |
| 검증/수정 반복 | ~$0.02/편 | 200편 | $4 |
| **LLM 소계** | | | **~$18** |

### TTS 비용

| 서비스 | 단가 | 예상 문자수 | 비용 |
|--------|------|-----------|------|
| OpenAI TTS-1-HD | $30/1M chars | ~350K chars | ~$11 |
| ElevenLabs | $0.30/1K chars | ~350K chars | ~$105 |

### 인프라 비용 (월간)

| 서비스 | 플랜 | 월 비용 |
|--------|------|---------|
| Supabase | Free tier (500MB) | $0 |
| Supabase | Pro (8GB, 더 필요시) | $25/월 |
| Neo4j Desktop | 로컬 | $0 |
| Vercel | Hobby | $0 |

**총 초기 비용: ~$30-130** (TTS 서비스 선택에 따라)

---

## 9. 리스크 매트릭스

| 리스크 | 확률 | 영향 | 완화 전략 |
|--------|------|------|----------|
| Lexile 점수 부정확 | 높음 | 중 | FK 기반 근사 + 교사 검수 |
| LLM 생성 텍스트 CEFR 불일치 | 중 | 높음 | 자동 검증 + 재생성 루프 |
| Neo4j API 통합 실패 | 중 | 높음 | Phase 0에서 POC 선행 |
| 콘텐츠 생산 일정 초과 | 높음 | 중 | 배치 자동화 파이프라인 우선 구축 |
| 저작권 이슈 (Classic Stories) | 낮음 | 높음 | 공공 도메인 작품만 선정 (확인 완료 대부분 OK) |
| 삽화 품질 불균일 | 중 | 낮음 | AI 생성 → 스타일 가이드 통일 |
| 5D 앱 DB 마이그레이션 실패 | 중 | 높음 | 로컬 백업 + 단계적 전환 |

---

## 10. 즉시 실행 가능한 액션 아이템

### 이번 주 (Week 0)

- [ ] **Supabase 프로젝트 생성** — 무료 티어, 리전: Northeast Asia (Tokyo)
- [ ] **Neo4j API 서버 테스트** — `npm run dev` → 전 엔드포인트 확인
- [ ] **POC: 이솝이야기 3편** — LLM 파이프라인으로 생성 → 메타데이터 자동 추출 → 검증

### 다음 주 (Week 1)

- [ ] **Supabase 스키마 생성** — 위 SQL 기반
- [ ] **compromise + FK 분석 스크립트** — POC 텍스트 3편에 적용
- [ ] **BNC/COCA 빈도 목록 확보** — K1-K3 분류 CSV 준비
- [ ] **콘텐츠 생성 프롬프트 템플릿** — Stage 1-2 용 작성

---

*검토 완료: 2026-02-28*
