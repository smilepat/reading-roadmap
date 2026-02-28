# EFL Reading Ladder — Product Requirements Document (PRD)

> **프로젝트명:** EFL Reading Ladder
> **버전:** 1.0
> **작성일:** 2026-02-28
> **상위 프로젝트:** AI TESOL 멀티에이전트 시스템
> **상태:** Draft → Review

---

## 목차

1. [제품 비전 & 목표](#1-제품-비전--목표)
2. [사용자 & 페르소나](#2-사용자--페르소나)
3. [7-Stage Reading Ladder 프레임워크](#3-7-stage-reading-ladder-프레임워크)
4. [Stage별 콘텐츠 상세 설계](#4-stage별-콘텐츠-상세-설계)
5. [시스템 아키텍처](#5-시스템-아키텍처)
6. [데이터베이스 설계](#6-데이터베이스-설계)
7. [기존 시스템 연동](#7-기존-시스템-연동)
8. [콘텐츠 생산 파이프라인](#8-콘텐츠-생산-파이프라인)
9. [프론트엔드 & UX](#9-프론트엔드--ux)
10. [AI 에이전트 연동](#10-ai-에이전트-연동)
11. [비기능 요구사항](#11-비기능-요구사항)
12. [구현 로드맵](#12-구현-로드맵)
13. [비용 추정](#13-비용-추정)
14. [리스크 & 완화 전략](#14-리스크--완화-전략)
15. [성공 지표](#15-성공-지표)
16. [부록](#16-부록)

---

## 1. 제품 비전 & 목표

### 1.1 비전

비영어권(한국) EFL 학습자가 **파닉스 동화**부터 **미국 Liberal Arts 학술 지문**까지 자연스럽게 성장하는 체계적 리딩 로드맵을 제공한다. 각 단계에서 5D 어휘학습, 수능 문항 연습, 작문 연계까지 원스톱으로 지원하는 통합 리딩 플랫폼.

### 1.2 핵심 목표

| 목표 | 측정 기준 | 목표값 |
|------|----------|--------|
| 단계적 읽기 능력 향상 | Stage 진급률 | 월 1단계 이상 진급 70% |
| 어휘-독해 연계 학습 | 텍스트 내 학습 어휘 활용률 | 80%+ |
| 수능 독해 대비 | Stage 6 이해도 점수 | 평균 75점+ |
| 교사 커리큘럼 지원 | 콘텐츠 활용률 | 주 3회 이상 |
| 학습 지속성 | 주간 스트릭 유지율 | 60%+ |

### 1.3 범위

**In Scope:**
- 7단계 리딩 텍스트 DB 구축 (~200편)
- 텍스트별 어휘/문법/문제/연습 메타데이터
- 학습 진도 추적 & Stage 진급 시스템
- 5D 어휘학습 앱 및 Neo4j 어휘 그래프 연동
- 수능 유형별 문항 연계
- TTS 음성 지원

**Out of Scope (v1):**
- 사용자 직접 텍스트 업로드
- 다국어 (일본어 등) 확장
- 모바일 네이티브 앱
- 동료 리뷰/소셜 기능

---

## 2. 사용자 & 페르소나

### 2.1 Primary: 한국 EFL 학생

| 속성 | 상세 |
|------|------|
| 연령 | 9-19세 (초3 ~ 고3) |
| CEFR | Pre-A1 ~ B2+ |
| 목표 | 영어 독해력 향상, 수능 대비 |
| 특성 | L1(한국어) 간섭, 제한된 영어 노출 환경 |
| 기기 | 데스크톱/태블릿 (학교 환경), 모바일 (자습) |

### 2.2 Secondary: 영어 교사

| 속성 | 상세 |
|------|------|
| 역할 | 초/중/고 영어 교사, 학원 강사 |
| 목표 | 수준별 리딩 자료 확보, 학생 진도 모니터링 |
| Pain Point | 수준별 텍스트 찾기 어려움, 문제 출제 시간 과다 |
| 기기 | 데스크톱 |

### 2.3 Tertiary: 성인 EFL 학습자

| 속성 | 상세 |
|------|------|
| 연령 | 20세+ |
| 목표 | 학술 영어 독해 (유학 준비, 자기계발) |
| 해당 Stage | 5-7 |

---

## 3. 7-Stage Reading Ladder 프레임워크

### 3.1 단계 구조

```
Stage 7 ─ Academic Discourse        ─ CEFR C1    ─ 대학/성인    ─ 20편
Stage 6 ─ CSAT & Critical Reading   ─ CEFR B2+   ─ 고등 2-3    ─ 40편
Stage 5 ─ Future World & Careers    ─ CEFR B2    ─ 고등 1-2    ─ 20편
Stage 4 ─ Great Lives (위인전)       ─ CEFR B1+   ─ 중등 2-3    ─ 30편
Stage 3 ─ Classic Stories           ─ CEFR B1    ─ 중등 1-2    ─ 30편
Stage 2 ─ Aesop's Fables           ─ CEFR A2    ─ 초등 5-6    ─ 20편
Stage 1 ─ Phonics Readers          ─ CEFR A1    ─ 초등 3-4    ─ 20편
Bridge  ─ 단계 전이 텍스트           ─ 각 전이구간  ─            ─ ~20편
```

### 3.2 핵심 매개변수

| Stage | CEFR | Lexile (추정) | 어휘 수준 | 평균 문장 길이 | 지문 길이 | 문법 상한 |
|-------|------|--------------|-----------|---------------|-----------|----------|
| 1 | Pre-A1~A1 | <200L | ~500 words | 3-7 words | 30-80 words | be동사, 현재형, 명령문 |
| 2 | A1~A2 | 200-450L | ~1,500 words | 6-10 words | 80-150 words | 과거형, and/but/because |
| 3 | A2~B1 | 450-650L | ~2,500 words | 8-14 words | 150-300 words | 현재완료, 관계대명사, 수동태 |
| 4 | B1~B1+ | 650-800L | ~3,000 words | 10-16 words | 250-400 words | 과거완료, 분사구문, 가정법2 |
| 5 | B1+~B2 | 800-950L | ~3,500 words | 12-18 words | 300-500 words | 복합 수동태, 도치, 화법전환 |
| 6 | B2~B2+ | 950-1100L | ~4,500 words | 14-22 words | 400-700 words | 강조구문, 동격, 가정법 복합 |
| 7 | C1 | 1100L+ | 5,000+ words | 18-30 words | 600-1200 words | 학술적 hedging, 명사화, 인용 동사 |

> **Note:** Lexile 점수는 Flesch-Kincaid Grade 기반 추정값. 공식 Lexile 계산은 MetaMetrics 독점 알고리즘으로 자동화 불가. `lexile_source: 'estimated'`로 표기.

### 3.3 Stage 진급 기준

| Stage 완료 조건 | 기준값 |
|----------------|--------|
| 해당 Stage 텍스트 완료 | 전체의 70% 이상 |
| 평균 이해도 점수 | 70점 이상 |
| 어휘 숙달도 | 대상 어휘의 60% 이상 |
| 브릿지 텍스트 | 해당 구간 브릿지 2편 이상 완료 |

---

## 4. Stage별 콘텐츠 상세 설계

### Stage 1: Phonics Readers (20편)

**목표:** 음소-문자 대응(GPC) 습득, 디코딩 자동화

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 30-80 words |
| 문장 구조 | SV, SVO 단순문 |
| 반복률 | 핵심 패턴 3회+ 반복 |
| 삽화 비율 | 70-80% |

**20편 음소 진행:**
```
#1-5:   Short vowels (a, e, i, o, u) — CVC 패턴
#6-8:   Long vowels (a_e, i_e, o_e) — CVCe 패턴
#9-10:  Consonant blends (bl, cl, st, sp)
#11-12: Digraphs (sh, ch, th, wh)
#13-14: Inflections (-ing, -ed)
#15-16: Vowel teams (ee/ea, oo/ou)
#17-18: R-controlled vowels (ar, or) + Diphthongs (ow, aw)
#19-20: 복합 패턴 (silent letters, 종합)
```

**한국인 EFL 특화:** /r/-/l/, /f/-/p/, /v/-/b/ 구분 집중 배치

---

### Stage 2: Aesop's Fables (20편)

**목표:** 서사 구조(Beginning-Middle-End), 도덕적 교훈, 기본 독해 전략

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 80-150 words (점진 증가) |
| 문장 구조 | SVO, SVC + and/but 접속 |
| 새 어휘/편 | 5-8개 |
| 삽화 비율 | 50-60% |

**20편 선정 기준:** 교훈의 다양성, 어휘 난이도 점진적 상승, 문화 보편성

| # | 이야기 | 핵심 교훈 | 단어 수 |
|---|--------|----------|---------|
| 1 | The Fox and the Grapes | 합리화 | 80 |
| 2 | The Tortoise and the Hare | 꾸준함 | 90 |
| 3 | The Boy Who Cried Wolf | 정직 | 100 |
| ... | ... | ... | ... |
| 20 | The Wolf and the Crane | 감사 | 150 |

*(전체 목록: 부록 A 참조)*

---

### Stage 3: Classic Stories (30편)

**목표:** 확장된 서사, 인물 심리 추론, 문맥 추론 전략

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 150-300 words (축약 리텔링) |
| 새 어휘/편 | 8-12개 |
| 삽화 비율 | 30-40% |

**6개 하위 카테고리 (각 4-6편):**
- A. 세계 명작 동화 (6편): Ugly Duckling, Cinderella, ...
- B. 모험 이야기 (6편): Robinson Crusoe, Treasure Island, ...
- C. 미스터리 & 추리 (5편): Sherlock Holmes, ...
- D. 판타지 & SF (5편): Alice in Wonderland, ...
- E. 감동 & 성장 (4편): The Happy Prince, Little Women, ...
- F. 한국 & 아시아 이야기 (4편): 선녀와 나무꾼, 콩쥐팥쥐, ...

*(전체 목록: 부록 B 참조)*

---

### Stage 4: Great Lives — 위인전 (30편)

**목표:** 논픽션 독해, 시간순 서사, 원인-결과, 문화적 배경

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 250-400 words |
| 새 어휘/편 | 10-15개 (도메인 어휘) |
| 텍스트 유형 | 전기문(biography) |

**6개 하위 카테고리 (각 4-6편):**
- A. 과학자 & 발명가: Marie Curie, Einstein, Ada Lovelace, ...
- B. 인권 & 사회 운동가: MLK, Mandela, Malala, ...
- C. 예술가 & 음악가: Da Vinci, Beethoven, BTS, ...
- D. 탐험가 & 개척자: Amelia Earhart, Armstrong, 장보고, ...
- E. 현대 혁신가: Steve Jobs, 반기문, ...
- F. 한국 위인: 세종대왕, 이순신, 유관순, 김연아

*(전체 목록: 부록 C 참조)*

---

### Stage 5: Future World & Careers (20편)

**목표:** 정보텍스트(expository) 독해, 주장-근거 구분, 비판적 사고

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 300-500 words |
| 새 어휘/편 | 15-20개 (전문용어) |
| 시각 자료 | 인포그래픽, 차트 포함 |

**4개 하위 카테고리 (각 5편):**
- A. AI & 테크놀로지: AI Engineer, Data Scientist, Cybersecurity, UX, Robotics
- B. 지속가능 미래: Climate Scientist, Renewable Energy, Urban Farm, Ocean Conservation, Circular Economy
- C. 의료 & 바이오: Genetic Counselor, Telemedicine, Biomedical, Mental Health App, Pandemic Preparedness
- D. 크리에이티브 & 글로벌: Content Creator, Space Tourism, Global Education, Social Entrepreneur, Metaverse Architect

---

### Stage 6: CSAT & Critical Reading (40편)

**목표:** 수능 유형별 독해 전략, 추론적 이해, 구조 분석

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 400-700 words |
| 문장 구조 | 도치, 강조구문, 삽입, 동격 |
| FK Grade | 10-12 |

**수능 유형 매핑:**

| 유형 | 편수 | 핵심 전략 |
|------|------|----------|
| 빈칸추론 | 8 | 논리적 추론, 문맥 파악 |
| 순서배열 | 6 | 담화 구조, 연결사 |
| 문장삽입 | 6 | 응집성, 지시어 추적 |
| 주제/요지 | 5 | 중심생각, 주제문 |
| 어휘추론 | 5 | 문맥적 의미 파악 |
| 요약문 완성 | 5 | 핵심 논지 압축 |
| 장문독해 | 5 | 복합 스킬 |

**40개 토픽 — 8개 대영역 × 5개 소주제:**

| 영역 | 5개 토픽 |
|------|----------|
| **AI & 기술** | AI and Creativity, Algorithmic Bias, Digital Privacy, Autonomous Weapons Ethics, Brain-Computer Interfaces |
| **환경 & 지속가능성** | Climate Tipping Points, Biodiversity Loss, Greenwashing, Environmental Justice, Degrowth vs Green Growth |
| **정의 & 윤리** | Restorative Justice, Gene Editing Ethics, Animal Rights, Intergenerational Justice, Cancel Culture & Free Speech |
| **심리 & 인지과학** | Cognitive Biases, Psychology of Misinformation, Emotional Intelligence, Paradox of Choice, Grit & Growth Mindset |
| **경제 & 사회** | Universal Basic Income, Gig Economy, Behavioral Economics, Inequality & Social Mobility, Attention Economy |
| **교육 & 언어** | Multilingualism, Hidden Curriculum, Digital Literacy, Standardized Testing, Lifelong Learning |
| **문화 & 커뮤니케이션** | Cultural Appropriation vs Exchange, Power of Narrative, Echo Chambers, Visual Literacy & Deepfakes, English as Lingua Franca |
| **과학 & 철학** | Nature of Consciousness, Reductionism vs Holism, Fermi Paradox, Epistemology, Ship of Theseus |

> **사용자 제공 토픽:** 교사가 40개 토픽을 직접 입력/수정할 수 있는 관리 인터페이스 제공

---

### Stage 7: Academic Discourse (20편)

**목표:** 미국 Liberal Arts 학술 지문, 비판적 분석, 학문적 글쓰기 기초

| 사양 | 기준 |
|------|------|
| 단어 수/편 | 600-1200 words |
| 어휘 | Academic Word List (AWL) 중심 |
| 텍스트 유형 | 학술 에세이, 강의 발췌, 연구 요약 |
| 출처 표기 | APA/MLA 인용 포함 |

**Stage 6 토픽의 학술적 심화 (20편):**

| 학문 분야 | Stage 6 연계 | 학술적 접근 |
|----------|-------------|------------|
| Philosophy | Restorative Justice | Rawls 정의론 vs 공동체주의 |
| Philosophy | Consciousness | Chalmers Hard Problem |
| Philosophy | Epistemology | 과학적 실재론 논쟁 |
| Psychology | Cognitive Biases | Kahneman 이중 처리 이론 |
| Psychology | Grit & Mindset | Dweck/Duckworth 비판적 검토 |
| Economics | UBI | 핀란드 실험 + 경제 모델링 |
| Economics | Behavioral Economics | Thaler/Sunstein 넛지 이론 |
| Sociology | Gig Economy | 불안정 노동(precariat) |
| Sociology | Inequality | Piketty 자본론 핵심 논지 |
| Political Sci. | Cancel Culture | Mill 자유론 현대 적용 |
| Env. Science | Tipping Points | 기후 피드백 루프 |
| Env. Science | Env. Justice | 환경 인종주의 사례 |
| Linguistics | Multilingualism | 이중언어 인지 이점 연구 |
| Linguistics | English as LF | 언어 제국주의 vs 실용주의 |
| Computer Sci. | AI and Creativity | 생성AI 저작권 쟁점 |
| Computer Sci. | Algorithmic Bias | 공정성 측정 지표 |
| Bioethics | Gene Editing | 생식세포 편집 윤리 |
| Media Studies | Deepfakes | 정보 생태계와 신뢰 |
| Education | Standardized Testing | Washback effect 연구 |
| Cognitive Sci. | Paradox of Choice | Schwartz vs Iyengar 실험 |

---

### Bridge Texts (~20편)

각 Stage 전이 구간에 3-4편의 브릿지 텍스트:

| 전이 | 브릿지 전략 | 편수 |
|------|-----------|------|
| 1→2 | 파닉스 패턴이 포함된 짧은 이솝이야기 | 3 |
| 2→3 | 교훈이 있는 짧은 명작 요약 | 3 |
| 3→4 | 클래식 작품 속 실제 인물 이야기 | 3 |
| 4→5 | 위인의 직업 세계를 현대적 재해석 | 4 |
| 5→6 | 직업 관련 사회적 이슈를 논설 형식으로 | 4 |
| 6→7 | 수능 지문 핵심 논지를 학술적 확장 | 3 |

---

## 5. 시스템 아키텍처

### 5.1 전체 아키텍처

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Client (Browser)                            │
│                                                                     │
│   Next.js 15 App Router + React 19 + Tailwind CSS + shadcn/ui     │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐            │
│   │ Reading   │ │ Vocab    │ │ Teacher  │ │ Dashboard │            │
│   │ Viewer    │ │ 5D Quiz  │ │ Console  │ │ Analytics │            │
│   └──────────┘ └──────────┘ └──────────┘ └───────────┘            │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ HTTPS
┌──────────────────────────────┴──────────────────────────────────────┐
│                      Next.js API Routes                             │
│                                                                     │
│   /api/reading/*     ── 리딩 텍스트 CRUD, 진도 추적                  │
│   /api/vocab/*       ── Neo4j 어휘 그래프 프록시                      │
│   /api/assessment/*  ── 5D 평가 및 프로필                            │
│   /api/analytics/*   ── 학습 분석 & 리포트                           │
│   /api/content/*     ── 콘텐츠 생성 파이프라인 (admin)                │
│   /api/tts/*         ── 음성 생성 프록시                             │
│                                                                     │
│   Prisma ORM ──────────────────┐                                    │
│   Vercel AI SDK (@ai-sdk/openai)│                                   │
└──────────────┬─────────────────┴────────────────────────────────────┘
               │                           │
    ┌──────────┴──────────┐     ┌──────────┴──────────┐
    │                     │     │                     │
    │  Supabase           │     │  Neo4j              │
    │  (PostgreSQL)       │     │  (Graph DB)         │
    │                     │     │                     │
    │  ┌───────────────┐  │     │  ┌───────────────┐  │
    │  │ reading_texts  │  │     │  │ Word (8,958)  │  │
    │  │ text_vocab     │◄─┼─FK──┼─►│ + 50K rels    │  │
    │  │ text_questions │  │     │  │               │  │
    │  │ text_exercises │  │     │  │ SYNONYM_OF    │  │
    │  │ text_grammar   │  │     │  │ ANTONYM_OF    │  │
    │  │ text_paragraphs│  │     │  │ HYPERNYM_OF   │  │
    │  │ reading_progress│ │     │  │ COLLOCATES    │  │
    │  │ users          │  │     │  │ WORD_FAMILY   │  │
    │  │ knowledge_prof │  │     │  │ HAS_CEFR      │  │
    │  │ assessments    │  │     │  │ IN_DOMAIN     │  │
    │  │ user_topics    │  │     │  │ ...           │  │
    │  └───────────────┘  │     │  └───────────────┘  │
    │                     │     │                     │
    │  Supabase Auth      │     │  bolt://localhost   │
    │  Supabase Storage   │     │  :7687              │
    │  (audio, images)    │     │                     │
    └─────────────────────┘     └─────────────────────┘

    ┌─────────────────────────────────────────────────┐
    │           External Services                      │
    │                                                  │
    │  OpenAI API ─── GPT-4o (콘텐츠 생성, 문법 태깅)   │
    │              ─── TTS-1-HD (음성 생성)             │
    │  Google Sheets API ── 교육과정 마스터 테이블        │
    └─────────────────────────────────────────────────┘
```

### 5.2 기술 스택

| 영역 | 기술 | 버전/상세 |
|------|------|----------|
| **프레임워크** | Next.js (App Router) | 15.x |
| **언어** | TypeScript | 5.x |
| **UI** | React + Tailwind CSS + shadcn/ui | React 19, Tailwind v4 |
| **ORM** | Prisma | 7.x |
| **관계형 DB** | Supabase (PostgreSQL) | — |
| **그래프 DB** | Neo4j Desktop | 2.1.1 (로컬) |
| **인증** | Supabase Auth | — |
| **스토리지** | Supabase Storage | 음성 파일, 삽화 |
| **AI** | Vercel AI SDK + OpenAI | GPT-4o, TTS-1-HD |
| **NLP** | compromise | JS 기반 토크나이징/POS |
| **차트** | Recharts | v3 |
| **배포** | Vercel | — |

### 5.3 아키텍처 결정 근거

**Why Supabase over Firebase?**

| 기준 | Supabase 적합 이유 |
|------|-------------------|
| Agent 복합 쿼리 | 문항생성 Agent: `WHERE stage=5 AND cefr='B2' AND csat_type='빈칸추론'` — SQL 네이티브 |
| 분석 집계 | 분석 Agent: `GROUP BY + AVG + JOIN` — PostgreSQL 강점 |
| 어휘 연속성 검증 | `JOIN text_vocabulary ON word WHERE stage IN (3,4)` — Firestore 불가 |
| 기존 스택 호환 | 5D 앱이 Prisma 사용 중 → `datasource` URL 변경만으로 전환 |
| 타입 안전성 | Prisma 자동 생성 타입 유지 |
| 마이그레이션 | `prisma migrate` 지원 |

**Why Neo4j를 유지?**

| 기준 | Neo4j 유지 이유 |
|------|----------------|
| 이미 구축 완료 | 8,958 Word, 50,000+ 관계 |
| 그래프 순회 | 동의어 체인, 연관어 네트워크 — 관계형 DB로 비효율 |
| 5D 차원 매핑 | `DIMENSION_GRAPH_MAP`이 이미 관계-차원 매핑 |
| 단어 네트워크 시각화 | `getWordNetwork()` API 존재 |

**어휘 데이터 역할 분담:**

```
Neo4j  = 어휘 마스터 (Single Source of Truth)
         WordNode 속성, 관계(SYNONYM, ANTONYM, ...), 5D 매핑

Supabase text_vocabulary = 텍스트-어휘 매핑 (컨텍스트 데이터)
         해당 텍스트 내 예문, 해당 텍스트에서의 5D 학습 포커스, 등장 순서
         neo4j_word_id로 참조
```

---

## 6. 데이터베이스 설계

### 6.1 Supabase PostgreSQL 스키마

```sql
-- ============================================================
-- EFL Reading Ladder — Supabase Schema
-- ============================================================

-- 기존 5D 앱에서 마이그레이션되는 테이블 (Prisma 관리)
-- users, assessments, assessment_responses,
-- knowledge_profiles, vocabulary, user_vocabulary
-- → Prisma schema.prisma에서 관리 (datasource만 변경)

-- ============================================================
-- 리딩 텍스트 핵심 테이블
-- ============================================================

CREATE TABLE reading_texts (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stage         SMALLINT NOT NULL CHECK (stage BETWEEN 1 AND 7),
  title         TEXT NOT NULL,
  subtitle      TEXT,
  category      TEXT NOT NULL,
  -- ENUM: phonics, fable, classic, biography, career, csat, academic
  subcategory   TEXT,
  -- e.g.: science_inventors, adventure, ai_tech
  topic_area    TEXT,
  -- Stage 6-7: ai_tech, environment, justice, psychology,
  --            economy, education, culture, science_philosophy
  topic_number  SMALLINT,
  -- Stage 6-7: 1-40

  -- 본문
  content       TEXT NOT NULL,
  content_html  TEXT,

  -- 난이도 지표
  cefr_level    TEXT NOT NULL,
  -- Pre-A1, A1, A2, B1, B1+, B2, B2+, C1
  flesch_kincaid_grade  REAL,
  flesch_reading_ease   REAL,
  coleman_liau_index    REAL,
  lexile_estimate       SMALLINT,
  -- FK Grade 기반 근사값
  lexile_source         TEXT DEFAULT 'estimated',
  -- 'estimated' | 'official'

  -- 텍스트 분석 (자동 계산)
  word_count            SMALLINT NOT NULL,
  sentence_count        SMALLINT,
  avg_sentence_length   REAL,
  avg_word_length       REAL,
  -- syllables per word
  unique_word_count     SMALLINT,
  lexical_density       REAL,
  -- unique / total

  -- 어휘 프로파일 (자동 계산, content 변경 시 재계산)
  k1_pct        REAL,   -- BNC/COCA 1-1000
  k2_pct        REAL,   -- 1001-2000
  k3_pct        REAL,   -- 2001-3000
  awl_pct       REAL,   -- Academic Word List
  off_list_pct  REAL,

  -- 수능 연계 (Stage 6 only)
  csat_type       TEXT,
  -- blank_inference, sentence_order, sentence_insertion,
  -- main_idea, vocabulary_inference, summary_completion, long_passage
  csat_difficulty TEXT,
  -- basic, intermediate, advanced

  -- 메타 정보
  author            TEXT,
  original_work     TEXT,
  illustration_ratio REAL DEFAULT 0,
  audio_url         TEXT,
  korean_translation TEXT,

  -- 브릿지 텍스트 표시
  is_bridge         BOOLEAN DEFAULT FALSE,
  bridge_from_stage SMALLINT,
  bridge_to_stage   SMALLINT,

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================
-- 문단 (1:N)
-- ============================================================

CREATE TABLE text_paragraphs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id         UUID NOT NULL REFERENCES reading_texts(id) ON DELETE CASCADE,
  paragraph_index SMALLINT NOT NULL,
  content         TEXT NOT NULL,
  word_count      SMALLINT,
  key_idea        TEXT,
  -- Stage 4+ 문단 핵심 아이디어
  key_idea_ko     TEXT,
  UNIQUE(text_id, paragraph_index)
);

-- ============================================================
-- 텍스트-어휘 매핑 (M:N, Neo4j 참조)
-- ============================================================

CREATE TABLE text_vocabulary (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id         UUID NOT NULL REFERENCES reading_texts(id) ON DELETE CASCADE,
  neo4j_word_id   TEXT,
  -- Neo4j Word 노드 id (stem), e.g. "environment"
  word            TEXT NOT NULL,
  -- 검색/fallback 용 중복 저장
  part_of_speech  TEXT,
  cefr_level      TEXT,
  definition_en   TEXT,
  definition_ko   TEXT,
  example_sentence TEXT,
  -- 이 텍스트 내 예문 (텍스트 고유)
  phonetic        TEXT,
  -- IPA
  five_d_dimension TEXT,
  -- 이 텍스트에서의 학습 포커스: semantic|contextual|form|relational|pragmatic
  order_in_text   SMALLINT,
  UNIQUE(text_id, word)
);

-- ============================================================
-- 문법 태그 (1:N)
-- ============================================================

CREATE TABLE text_grammar_tags (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id           UUID NOT NULL REFERENCES reading_texts(id) ON DELETE CASCADE,
  feature           TEXT NOT NULL,
  -- e.g.: past_simple, passive_voice, relative_clause, inversion
  cefr_level        TEXT,
  example_from_text TEXT,
  explanation       TEXT,
  explanation_ko    TEXT
);

-- ============================================================
-- 이해도 문제 (1:N)
-- ============================================================

CREATE TABLE text_questions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id         UUID NOT NULL REFERENCES reading_texts(id) ON DELETE CASCADE,
  question_type   TEXT NOT NULL,
  -- multiple_choice, true_false, short_answer, open_ended
  question        TEXT NOT NULL,
  question_ko     TEXT,
  options         JSONB,
  -- ["option A", "option B", ...]
  correct_answer  TEXT NOT NULL,
  explanation     TEXT,
  explanation_ko  TEXT,
  skill           TEXT,
  -- literal, inferential, evaluative, creative
  csat_type       TEXT,
  -- 수능 유형 연계 (Stage 6)
  order_index     SMALLINT DEFAULT 0
);

-- ============================================================
-- 어휘 연습 (1:N)
-- ============================================================

CREATE TABLE text_exercises (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  text_id         UUID NOT NULL REFERENCES reading_texts(id) ON DELETE CASCADE,
  exercise_type   TEXT NOT NULL,
  -- fill_blank, matching, word_form, synonym_antonym, context_clue
  prompt          TEXT NOT NULL,
  prompt_ko       TEXT,
  answer          TEXT NOT NULL,
  distractors     JSONB,
  -- ["wrong1", "wrong2", ...]
  order_index     SMALLINT DEFAULT 0
);

-- ============================================================
-- 학습 진도 추적
-- ============================================================

CREATE TABLE reading_progress (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID NOT NULL,
  -- Supabase Auth user id
  text_id             UUID NOT NULL REFERENCES reading_texts(id) ON DELETE CASCADE,
  status              TEXT DEFAULT 'not_started',
  -- not_started, in_progress, completed
  started_at          TIMESTAMPTZ,
  completed_at        TIMESTAMPTZ,
  comprehension_score SMALLINT,
  -- 0-100
  vocabulary_score    SMALLINT,
  -- 0-100
  time_spent          INTEGER DEFAULT 0,
  -- seconds
  read_count          SMALLINT DEFAULT 0,
  notes               TEXT,
  UNIQUE(user_id, text_id)
);

-- ============================================================
-- Stage 진급 기준 (설정 테이블)
-- ============================================================

CREATE TABLE stage_promotions (
  stage                       SMALLINT PRIMARY KEY,
  required_texts_completed    SMALLINT NOT NULL DEFAULT 14,
  -- 70% of 20
  required_avg_comprehension  SMALLINT NOT NULL DEFAULT 70,
  required_vocab_mastery      SMALLINT NOT NULL DEFAULT 60,
  required_bridges_completed  SMALLINT NOT NULL DEFAULT 2
);

-- 기본값 삽입
INSERT INTO stage_promotions (stage, required_texts_completed, required_avg_comprehension) VALUES
  (1, 14, 70), (2, 14, 70), (3, 21, 70), (4, 21, 70),
  (5, 14, 75), (6, 28, 75), (7, 14, 80);

-- ============================================================
-- 사용자 제공 토픽 (Stage 6-7)
-- ============================================================

CREATE TABLE user_topics (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  area_code       TEXT NOT NULL,
  area_name       TEXT NOT NULL,
  area_name_ko    TEXT,
  topic_number    SMALLINT NOT NULL,
  title           TEXT NOT NULL,
  title_ko        TEXT,
  description     TEXT,
  keywords        JSONB,
  -- ["keyword1", "keyword2", ...]
  related_academic_field TEXT,
  -- Stage 7 연계 학문 분야
  created_by      UUID,
  -- 토픽 제공 교사
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(area_code, topic_number)
);

-- ============================================================
-- 인덱스
-- ============================================================

CREATE INDEX idx_texts_stage        ON reading_texts(stage);
CREATE INDEX idx_texts_cefr         ON reading_texts(cefr_level);
CREATE INDEX idx_texts_category     ON reading_texts(category);
CREATE INDEX idx_texts_csat_type    ON reading_texts(csat_type);
CREATE INDEX idx_texts_topic_area   ON reading_texts(topic_area);
CREATE INDEX idx_texts_bridge       ON reading_texts(is_bridge) WHERE is_bridge = TRUE;

CREATE INDEX idx_vocab_text         ON text_vocabulary(text_id);
CREATE INDEX idx_vocab_word         ON text_vocabulary(word);
CREATE INDEX idx_vocab_neo4j        ON text_vocabulary(neo4j_word_id);

CREATE INDEX idx_paragraphs_text    ON text_paragraphs(text_id);
CREATE INDEX idx_grammar_text       ON text_grammar_tags(text_id);
CREATE INDEX idx_questions_text     ON text_questions(text_id);
CREATE INDEX idx_exercises_text     ON text_exercises(text_id);

CREATE INDEX idx_progress_user      ON reading_progress(user_id);
CREATE INDEX idx_progress_text      ON reading_progress(text_id);
CREATE INDEX idx_progress_status    ON reading_progress(user_id, status);

-- ============================================================
-- RLS (Row Level Security)
-- ============================================================

ALTER TABLE reading_texts ENABLE ROW LEVEL SECURITY;
ALTER TABLE reading_progress ENABLE ROW LEVEL SECURITY;

-- 텍스트: 누구나 읽기 가능
CREATE POLICY "texts_read_all" ON reading_texts
  FOR SELECT USING (true);

-- 텍스트: 교사만 쓰기 가능 (role 기반, 추후 구현)
-- CREATE POLICY "texts_write_teacher" ON reading_texts
--   FOR INSERT WITH CHECK (auth.jwt() ->> 'role' = 'teacher');

-- 진도: 본인 데이터만 접근
CREATE POLICY "progress_own_read" ON reading_progress
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "progress_own_write" ON reading_progress
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "progress_own_update" ON reading_progress
  FOR UPDATE USING (auth.uid() = user_id);
```

### 6.2 Prisma 스키마 확장

기존 5D 앱의 Prisma 스키마에 Reading 모델을 추가하는 방식:

```prisma
// === 기존 모델 유지 (5D 어휘학습) ===
// User, Assessment, AssessmentResponse,
// KnowledgeProfile, Vocabulary, UserVocabulary

// === datasource 변경 (SQLite → Supabase) ===
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  // DATABASE_URL = Supabase connection string
}

// === 새 모델 추가 (Reading Ladder) ===

model ReadingText {
  id            String   @id @default(uuid())
  stage         Int
  title         String
  subtitle      String?
  category      String
  subcategory   String?
  topicArea     String?  @map("topic_area")
  topicNumber   Int?     @map("topic_number")

  content       String
  contentHtml   String?  @map("content_html")

  cefrLevel           String  @map("cefr_level")
  fleschKincaidGrade  Float?  @map("flesch_kincaid_grade")
  fleschReadingEase   Float?  @map("flesch_reading_ease")
  colemanLiauIndex    Float?  @map("coleman_liau_index")
  lexileEstimate      Int?    @map("lexile_estimate")
  lexileSource        String  @default("estimated") @map("lexile_source")

  wordCount         Int     @map("word_count")
  sentenceCount     Int?    @map("sentence_count")
  avgSentenceLength Float?  @map("avg_sentence_length")
  avgWordLength     Float?  @map("avg_word_length")
  uniqueWordCount   Int?    @map("unique_word_count")
  lexicalDensity    Float?  @map("lexical_density")

  k1Pct       Float?  @map("k1_pct")
  k2Pct       Float?  @map("k2_pct")
  k3Pct       Float?  @map("k3_pct")
  awlPct      Float?  @map("awl_pct")
  offListPct  Float?  @map("off_list_pct")

  csatType       String?  @map("csat_type")
  csatDifficulty String?  @map("csat_difficulty")

  author             String?
  originalWork       String?  @map("original_work")
  illustrationRatio  Float?   @default(0) @map("illustration_ratio")
  audioUrl           String?  @map("audio_url")
  koreanTranslation  String?  @map("korean_translation")

  isBridge        Boolean  @default(false) @map("is_bridge")
  bridgeFromStage Int?     @map("bridge_from_stage")
  bridgeToStage   Int?     @map("bridge_to_stage")

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt      @map("updated_at")

  paragraphs  TextParagraph[]
  vocabulary  TextVocabulary[]
  grammarTags TextGrammarTag[]
  questions   TextQuestion[]
  exercises   TextExercise[]
  progress    ReadingProgress[]

  @@map("reading_texts")
  @@index([stage])
  @@index([cefrLevel])
  @@index([category])
}

model TextParagraph {
  id             String @id @default(uuid())
  textId         String @map("text_id")
  paragraphIndex Int    @map("paragraph_index")
  content        String
  wordCount      Int?   @map("word_count")
  keyIdea        String? @map("key_idea")
  keyIdeaKo      String? @map("key_idea_ko")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@unique([textId, paragraphIndex])
  @@map("text_paragraphs")
}

model TextVocabulary {
  id             String  @id @default(uuid())
  textId         String  @map("text_id")
  neo4jWordId    String? @map("neo4j_word_id")
  word           String
  partOfSpeech   String? @map("part_of_speech")
  cefrLevel      String? @map("cefr_level")
  definitionEn   String? @map("definition_en")
  definitionKo   String? @map("definition_ko")
  exampleSentence String? @map("example_sentence")
  phonetic       String?
  fiveDDimension String? @map("five_d_dimension")
  orderInText    Int?    @map("order_in_text")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@unique([textId, word])
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
  id             String  @id @default(uuid())
  textId         String  @map("text_id")
  questionType   String  @map("question_type")
  question       String
  questionKo     String? @map("question_ko")
  options        Json?
  correctAnswer  String  @map("correct_answer")
  explanation    String?
  explanationKo  String? @map("explanation_ko")
  skill          String?
  csatType       String? @map("csat_type")
  orderIndex     Int     @default(0) @map("order_index")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@map("text_questions")
}

model TextExercise {
  id           String  @id @default(uuid())
  textId       String  @map("text_id")
  exerciseType String  @map("exercise_type")
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

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@unique([userId, textId])
  @@map("reading_progress")
}
```

### 6.3 Neo4j 기존 스키마 (변경 없음)

기존 vocab-knowledge-graph의 구조를 그대로 활용:

```
Node Labels:
  Word (8,958개) — id, display, pos, ipa, meaningKo, definitionEn, ...
  CEFRLevel      — A1, A2, B1, B2, C1, C2
  Topic          — animals, science, emotion, ...
  Domain         — academic, business, general, technical
  GradeRange     — 초3-초6, 중1-중3, 고1-고3
  WordList       — oxford3000, oxford5000, toefl, toeic, sat, ngsl

Relationships:
  SYNONYM_OF, ANTONYM_OF, HYPERNYM_OF, HYPONYM_OF
  WORD_FAMILY, COLLOCATES_WITH
  HAS_CEFR, BELONGS_TO_TOPIC, IN_DOMAIN, IN_GRADE, MEMBER_OF
```

**연결 방식:** `text_vocabulary.neo4j_word_id` → Neo4j `Word.id` (stem)

### 6.4 데이터 흐름

```
[콘텐츠 생성 시]
  GPT-4o → 텍스트 본문
    → compromise → 자동 분석 (word_count, FK, etc.)
    → 어휘 추출 → Neo4j 대조 → text_vocabulary 레코드 생성
    → GPT-4o → 문법 태깅 → text_grammar_tags
    → GPT-4o → 문제 생성 → text_questions + text_exercises
    → OpenAI TTS → audio_url (Supabase Storage)

[학생 학습 시]
  reading_texts 조회 (stage 필터)
    → text_vocabulary 로드 + Neo4j 5D 프로필 조회
    → 학생 답변 → reading_progress 업데이트
    → knowledge_profile 갱신 (5D 점수)

[교사 분석 시]
  SQL 집계 → 학급별/개인별 Stage 진행 현황
    → reading_progress JOIN reading_texts → 차원별 성취도
    → Recharts 시각화
```

---

## 7. 기존 시스템 연동

### 7.1 5D 어휘학습 앱 (5dimension_vocablearning)

**현재 상태:**
- Prisma + SQLite (로컬)
- 모델: User, Assessment, AssessmentResponse, KnowledgeProfile, Vocabulary, UserVocabulary
- 어휘 데이터 소스 없음 (빈 상태)

**마이그레이션 계획:**

```
Step 1: Prisma datasource를 Supabase PostgreSQL로 변경
        datasource db {
          provider = "postgresql"
          url      = env("DATABASE_URL")
        }

Step 2: 기존 SQLite 데이터를 Supabase로 마이그레이션
        (사용자/평가 데이터가 있다면)

Step 3: PrismaBetterSqlite3 어댑터 제거
        → 표준 Prisma PostgreSQL 클라이언트 사용

Step 4: Reading Ladder 모델 추가 → prisma migrate dev

Step 5: 어휘 소스 연결
        → Neo4j API를 통해 어휘 데이터 공급
        → 또는 text_vocabulary에서 Stage별 학습 어휘 조회
```

**공유 모델:**
- `User` — 동일 사용자가 5D 퀴즈 + 리딩 모두 사용
- `KnowledgeProfile` — 5D 점수가 리딩 추천에 활용
- `Vocabulary` → `TextVocabulary` — 어휘 학습과 텍스트 어휘 연결

### 7.2 Neo4j Vocabulary Knowledge Graph

**현재 상태:**
- 8,958 Word 노드, 50,000+ 관계
- API 서버 존재 (`apps/api/`) — **미테스트**
- 7개 API 엔드포인트

**연동 계획:**

```
기존 API (테스트 후 사용):
  GET  /api/vocab/[word]           → 단어 조회
  GET  /api/vocab/related/[word]   → 관련어
  GET  /api/vocab/5d-profile/[word]→ 5D 프로필
  POST /api/vocab/search           → 필터 검색
  GET  /api/graph/stats            → 통계
  GET  /api/graph/cefr-distribution→ CEFR 분포
  GET  /api/graph/network/[word]   → 네트워크

신규 API (추가 구현):
  POST /api/vocab/batch            → 단어 목록 일괄 조회
  GET  /api/vocab/frequency/[word] → BNC/COCA 빈도 레벨
```

**선행 작업:**
1. Neo4j API 서버 기동 + 전 엔드포인트 테스트
2. `POST /api/vocab/batch` 추가 (텍스트 내 어휘 일괄 매칭)
3. Reading Ladder 앱에서 Neo4j API를 프록시 호출

### 7.3 연동 시퀀스 다이어그램

```
[콘텐츠 등록 시]

Admin CLI                  Supabase              Neo4j API
    │                         │                      │
    │── INSERT reading_text ──►│                      │
    │                         │                      │
    │── POST /vocab/batch ────┼──────────────────────►│
    │   (텍스트 내 모든 단어)   │                      │
    │◄─ matched words[] ──────┼──────────────────────│
    │                         │                      │
    │── INSERT text_vocabulary │                      │
    │   (neo4j_word_id 포함) ──►│                      │
    │                         │                      │


[학생 학습 시]

Student Browser            Next.js API            Supabase         Neo4j API
    │                         │                      │                │
    │── GET /reading/stage/3 ─►│                      │                │
    │                         │── SELECT texts ──────►│                │
    │◄─ text list ────────────│◄─ results ───────────│                │
    │                         │                      │                │
    │── GET /reading/[id] ────►│                      │                │
    │                         │── SELECT text + vocab►│                │
    │                         │                      │                │
    │                         │── GET /vocab/5d/[w] ──┼───────────────►│
    │                         │◄─ 5D profiles ────────┼───────────────│
    │◄─ full text + 5D vocab ─│                      │                │
    │                         │                      │                │
    │── POST /progress ───────►│                      │                │
    │                         │── UPSERT progress ───►│                │
    │◄─ OK + stage check ─────│◄─ OK ────────────────│                │
```

---

## 8. 콘텐츠 생산 파이프라인

### 8.1 자동화 파이프라인 구조

```
┌──────────────────────────────────────────────────────────────┐
│                  Content Generation Pipeline                  │
│                  (CLI Script — Node.js/TypeScript)             │
│                                                               │
│  Input:                                                       │
│    stage, category, title, targetWordCount, cefrLevel,        │
│    targetVocabulary[], phonicsFocus? (Stage 1),               │
│    csatType? (Stage 6), academicField? (Stage 7)              │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 1: Text Generation (GPT-4o)                        │ │
│  │   - Stage별 프롬프트 템플릿 7종                          │ │
│  │   - 목표 어휘 포함 지시                                  │ │
│  │   - CEFR 레벨 문법 제한 지시                             │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 2: Auto Analysis (compromise + custom)             │ │
│  │   - 문장 분리, 단어 수, 평균 문장 길이                    │ │
│  │   - FK Grade, Reading Ease, Coleman-Liau 계산           │ │
│  │   - Lexile 근사값 산출                                   │ │
│  │   - K1/K2/K3/AWL 어휘 프로파일                           │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 3: Difficulty Validation                           │ │
│  │   - FK Grade가 Stage 범위 내인지 확인                    │ │
│  │   - 평균 문장 길이가 기준 내인지 확인                     │ │
│  │   - off_list 어휘 비율이 과도하지 않은지 확인             │ │
│  │   ❌ 실패 → Step 1로 복귀 (최대 3회)                    │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 4: Vocabulary Matching (Neo4j)                     │ │
│  │   - 텍스트 내 어휘 추출 → POST /api/vocab/batch         │ │
│  │   - 매칭된 어휘의 5D 프로필 조회                         │ │
│  │   - 학습 대상 어휘 선정 (CEFR + 빈도 기반)               │ │
│  │   - text_vocabulary 레코드 생성                          │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 5: Grammar Tagging (GPT-4o)                        │ │
│  │   - "이 텍스트에서 CEFR [level] 문법 구조를 식별하세요"   │ │
│  │   - 문법 요소 + 해당 예문 + 설명 추출                    │ │
│  │   - text_grammar_tags 레코드 생성                        │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 6: Question Generation (GPT-4o)                    │ │
│  │   - Stage별 문제 유형 분기                               │ │
│  │     Stage 1-2: true/false, 그림 매칭                     │ │
│  │     Stage 3-4: multiple choice, short answer             │ │
│  │     Stage 5-7: 수능 유형, open-ended                     │ │
│  │   - text_questions + text_exercises 레코드 생성           │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 7: TTS Audio Generation (OpenAI TTS-1-HD)          │ │
│  │   - 텍스트 본문 → mp3/ogg 생성                          │ │
│  │   - Supabase Storage 업로드                              │ │
│  │   - audio_url 업데이트                                   │ │
│  └────────────────────────┬────────────────────────────────┘ │
│                           ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Step 8: Expert Review Queue                             │ │
│  │   - 자동 생성 결과를 검수 대시보드에 등록                  │ │
│  │   - status: 'draft' → 'reviewed' → 'published'          │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### 8.2 프롬프트 템플릿 구조 (Stage별)

```typescript
const STAGE_PROMPTS: Record<number, string> = {
  1: `You are creating a phonics reader for Korean EFL learners (age 9-10, CEFR Pre-A1 to A1).

CONSTRAINTS:
- Target word count: {targetWordCount} words
- Focus phonics pattern: {phonicsFocus}
- Use only CVC/CVCe words and sight words
- Sentences: SV or SVO only, max 7 words
- Repeat the target pattern at least 3 times
- No words with more than 2 syllables
- Include only present tense and imperatives

Write a short story titled "{title}" that practices the {phonicsFocus} pattern.`,

  // 2-7: 각 Stage별 유사 구조
  6: `You are writing a CSAT-style English reading passage for Korean high school students (CEFR B2-B2+).

CONSTRAINTS:
- Target word count: {targetWordCount} words (400-700)
- Topic: {topicTitle}
- CSAT question type: {csatType}
- Use complex sentence structures: inversion, cleft sentences, appositives
- Include abstract vocabulary and academic register
- Average sentence length: 14-22 words
- Flesch-Kincaid Grade: 10-12
- Must include at least 2 instances of: passive voice, relative clauses, discourse markers

REQUIRED VOCABULARY (must appear naturally):
{targetVocabulary}

Write an argumentative/expository passage on the topic.`,
}
```

### 8.3 텍스트 분석 모듈

```typescript
// lib/text-analyzer.ts

import nlp from 'compromise'

interface TextAnalysis {
  wordCount: number
  sentenceCount: number
  avgSentenceLength: number
  avgWordLength: number       // syllables
  uniqueWordCount: number
  lexicalDensity: number
  fleschKincaidGrade: number
  fleschReadingEase: number
  colemanLiauIndex: number
  lexileEstimate: number
  vocabularyProfile: {
    k1Pct: number
    k2Pct: number
    k3Pct: number
    awlPct: number
    offListPct: number
  }
}

// Flesch-Kincaid Grade Level
// = 0.39 × (words/sentences) + 11.8 × (syllables/words) − 15.59

// Lexile 근사 변환 (FK Grade → Lexile)
// 연구 기반 look-up table
const FK_TO_LEXILE: Record<number, number> = {
  1: 190, 2: 420, 3: 520, 4: 620, 5: 730,
  6: 830, 7: 925, 8: 1010, 9: 1050, 10: 1080,
  11: 1100, 12: 1130, 13: 1200
}
```

### 8.4 어휘 프로파일링

```
BNC/COCA 빈도 목록 (Paul Nation's word lists):
  K1: 1-1000      → 고빈도 기초 어휘
  K2: 1001-2000   → 일상 어휘
  K3: 2001-3000   → 중급 어휘
  AWL: Coxhead's Academic Word List (570 word families)
  Off-list: 위 목록에 없는 어휘

도구: BNC/COCA 단어 목록 CSV + Set lookup
  → 각 텍스트의 모든 단어를 분류 → 비율 계산
```

---

## 9. 프론트엔드 & UX

### 9.1 주요 화면

| 화면 | 사용자 | 핵심 기능 |
|------|--------|----------|
| **Stage Map** | 학생 | 7단계 지도 시각화, 현재 위치, 진급 진도 |
| **Text Browser** | 학생/교사 | Stage별 텍스트 목록, 필터 (카테고리, CEFR, 수능유형) |
| **Reading Viewer** | 학생 | 텍스트 본문 + 어휘 팝업 + TTS + 문제 풀기 |
| **Vocab Panel** | 학생 | 텍스트 내 학습 어휘 목록, 5D 레이더 차트 |
| **Progress Dashboard** | 학생 | 완료 텍스트, 스트릭, Stage 진급 현황 |
| **Teacher Console** | 교사 | 학급 진도 모니터링, 텍스트 할당, 토픽 관리 |
| **Analytics** | 교사 | 학급/개인 이해도 추이, 어휘 숙달도, 5D 프로필 |
| **Admin: Content** | 관리자 | 콘텐츠 생성 파이프라인 실행, 검수 대시보드 |

### 9.2 Reading Viewer 상세

```
┌──────────────────────────────────────────────────────┐
│  ← Back to Stage 3    "Treasure Island"     🔊 Audio │
│  Classic Stories > Adventure     CEFR A2-B1          │
├──────────────────────────────────────────────────────┤
│                                                      │
│  [삽화 영역 — Stage 1-3]                              │
│                                                      │
│  Jim Hawkins found an old map in the sea            │
│  captain's chest. The map showed the                │
│  location of a hidden treasure on a                 │
│  faraway island. He was very excited.               │
│                                                      │
│  [treasure] [faraway] ← 밑줄 어휘 클릭 시 팝업       │
│                                                      │
│  ┌──────────────────────────────┐                    │
│  │ treasure /ˈtreʒ.ər/ (n.)    │                    │
│  │ 보물                         │                    │
│  │ CEFR: A2 | 5D: Semantic      │                    │
│  │ 🔊 발음 듣기                  │                    │
│  │ Synonyms: riches, fortune     │ ← Neo4j 연동      │
│  └──────────────────────────────┘                    │
│                                                      │
├──────────────────────────────────────────────────────┤
│  📝 Comprehension Check (3/5)                        │
│                                                      │
│  Q1. What did Jim find in the chest?                 │
│    ○ A sword  ● A map  ○ A key  ○ Gold coins        │
│  ✅ Correct!                                         │
│                                                      │
│  Q2. How did Jim feel?                               │
│    ...                                               │
└──────────────────────────────────────────────────────┘
```

### 9.3 기술 구현

- **텍스트 렌더링:** React 컴포넌트로 문단별 렌더링, 어휘 자동 하이라이트
- **어휘 팝업:** Radix Popover, Neo4j API에서 실시간 5D/관계어 조회
- **TTS:** HTML5 Audio, Supabase Storage에서 mp3 스트리밍
- **문제 풀기:** 인터랙티브 폼, 즉시 피드백, 점수 자동 계산
- **차트:** Recharts v3 레이더 차트 (5D 프로필), 라인 차트 (진도 추이)

---

## 10. AI 에이전트 연동

### 10.1 멀티에이전트 시스템 내 위치

```
Reading Ladder는 다음 에이전트와 연동:

┌─────────────────┐
│  Orchestrator    │─── 학생 CEFR + 5D 프로필 기반 Stage/텍스트 추천
└────────┬────────┘
         │
    ┌────┴─────────────────────────────────────────┐
    │                                              │
    ▼                                              ▼
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│ 어휘 코치    │  │ 문항생성      │  │ 분석 리포트   │
│ Agent       │  │ Agent        │  │ Agent        │
│             │  │              │  │              │
│ 텍스트 어휘  │  │ 텍스트 기반   │  │ 진도 + 이해도 │
│ → 5D 퀴즈   │  │ 추가 문항    │  │ 집계 리포트   │
└─────────────┘  └──────────────┘  └──────────────┘
    │                                    │
    ▼                                    ▼
┌─────────────┐              ┌──────────────┐
│ 작문 코치    │              │ 커리큘럼 설계  │
│ Agent       │              │ Agent        │
│             │              │              │
│ 텍스트 주제  │              │ Stage별 학습  │
│ → 에세이    │              │ 계획 생성     │
└─────────────┘              └──────────────┘
```

### 10.2 에이전트별 API 소비 패턴

| Agent | 사용 API | SQL 쿼리 유형 |
|-------|---------|--------------|
| **Orchestrator** | `GET /api/reading?stage={n}` | `SELECT * FROM reading_texts WHERE stage = ?` + KnowledgeProfile JOIN |
| **어휘 코치** | `GET /api/reading/{id}/vocabulary` | `SELECT * FROM text_vocabulary WHERE text_id = ?` + Neo4j batch |
| **문항생성** | `POST /api/content/generate-questions` | `SELECT * FROM reading_texts WHERE stage = ? AND csat_type = ?` |
| **분석 리포트** | `GET /api/analytics/class/{id}` | `GROUP BY stage, AVG(comprehension_score)` + JOIN progress |
| **작문 코치** | `GET /api/reading/{id}` | 텍스트 본문 + 토픽 정보 → 에세이 주제 생성 |
| **커리큘럼** | `GET /api/analytics/student/{id}/gaps` | 어휘 숙달도 갭 분석 → 다음 텍스트 추천 |

### 10.3 에이전트 간 데이터 흐름 예시

**시나리오: 어휘 학습 → 리딩 → 작문 연계**

```
1. 어휘 코치 Agent: "environment" 단어 5D 학습 완료
   → KnowledgeProfile.contextualScore 갱신

2. Orchestrator: contextual 약점 감지
   → Reading Ladder에서 "environment"가 포함된 Stage 5 텍스트 추천
   → SELECT * FROM text_vocabulary tv
     JOIN reading_texts rt ON tv.text_id = rt.id
     WHERE tv.word = 'environment' AND rt.stage = 5

3. 학생이 텍스트 읽기 + 문제 풀기 완료
   → reading_progress 업데이트

4. Orchestrator → 작문 코치: 텍스트 토픽("Climate Scientist") 전달
   → 작문 코치: "환경 과학자의 역할에 대한 의견문 작성" 과제 생성
   → 학습 어휘("environment", "renewable", "sustainability") 포함 유도

5. 분석 리포트 Agent: 어휘→리딩→작문 연계 학습 효과 리포트 생성
```

---

## 11. 비기능 요구사항

### 11.1 성능

| 지표 | 목표 |
|------|------|
| 텍스트 목록 로딩 | < 500ms |
| 텍스트 상세 로딩 (어휘 포함) | < 1s |
| Neo4j 어휘 조회 | < 200ms/단어 |
| TTS 오디오 첫 재생 | < 2s |
| 문제 채점/저장 | < 300ms |

### 11.2 확장성

| 항목 | 현재 | 확장 계획 |
|------|------|----------|
| 텍스트 수 | ~200편 | 500편+ (사용자 기여) |
| 동시 사용자 | ~50 | Supabase Pro 시 ~500 |
| 어휘 DB | 8,958 단어 | 15,000+ (COCA 확장) |

### 11.3 보안

- Supabase Auth 기반 인증 (이메일/비밀번호, 추후 소셜 로그인)
- RLS로 학생은 본인 진도만 접근
- 교사는 학급 학생 진도 조회 가능 (추후 역할 기반 정책)
- API 키 (OpenAI, Neo4j) 서버사이드만 저장

### 11.4 접근성

- 반응형 디자인 (모바일 지원)
- 텍스트 크기 조절 (Stage 1-2: 기본 18px)
- TTS로 텍스트 음성 제공
- 한국어 UI + 영어 콘텐츠 이중 언어 인터페이스

---

## 12. 구현 로드맵

### Phase 0: 인프라 구축 (4주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W1 | Supabase 프로젝트 생성, Auth 설정 | Supabase 인스턴스 |
| W1 | Neo4j API 서버 기동 + 전 엔드포인트 테스트 | 테스트 결과 보고서 |
| W2 | Prisma 스키마 확장 (Reading 모델) + 마이그레이션 | schema.prisma v2 |
| W2 | 5D 앱 datasource 전환 (SQLite → Supabase) | 5D 앱 정상 동작 확인 |
| W3 | Neo4j 배치 조회 API 추가 (`POST /vocab/batch`) | API 배포 |
| W3 | BNC/COCA 빈도 목록 준비 (K1-K3 CSV) | 어휘 분류 데이터 |
| W4 | **POC: 이솝이야기 3편** — 파이프라인 전체 테스트 | 3편 + 메타데이터 + 음성 |

### Phase 1: 기반 도구 (3주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W5 | text-analyzer 모듈 구현 (FK, vocab profile) | lib/text-analyzer.ts |
| W5 | 콘텐츠 파이프라인 CLI 도구 구현 | scripts/generate-content.ts |
| W6 | Stage별 프롬프트 템플릿 7종 작성 | lib/prompts/ |
| W6 | 난이도 검증 + 자동 재생성 루프 구현 | 파이프라인 Step 1-3 |
| W7 | Neo4j 어휘 매칭 + 문법 태깅 + 문제 생성 자동화 | 파이프라인 Step 4-6 |
| W7 | TTS 생성 + Supabase Storage 업로드 | 파이프라인 Step 7 |

### Phase 2: Stage 1-2 콘텐츠 (4주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W8-9 | Phonics Readers 20편 생성 + 검수 | 20편 published |
| W10-11 | Aesop's Fables 20편 생성 + 검수 | 20편 published |
| W10 | Bridge 1→2 텍스트 3편 | 3편 published |
| W11 | Reading Viewer 프론트엔드 구현 | 화면 동작 |

### Phase 3: Stage 3-4 콘텐츠 (6주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W12-14 | Classic Stories 30편 | 30편 published |
| W15-17 | Great Lives 30편 | 30편 published |
| W14 | Bridge 2→3, 3→4 텍스트 6편 | 6편 published |
| W16 | Stage Map + Progress Dashboard | 화면 동작 |

### Phase 4: Stage 5-6 콘텐츠 (6주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W18-19 | Future Careers 20편 | 20편 published |
| W20-22 | CSAT 지문 40편 (8영역 × 5토픽) | 40편 published |
| W20 | 수능 유형별 문제 생성 특화 프롬프트 | 유형당 5-8문항 |
| W21 | Bridge 4→5, 5→6 텍스트 8편 | 8편 published |
| W22 | Teacher Console + 토픽 관리 화면 | 화면 동작 |
| W23 | 사용자 토픽 입력/수정 기능 | user_topics CRUD |

### Phase 5: Stage 7 & 통합 (4주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W24-25 | Academic Discourse 20편 | 20편 published |
| W25 | Bridge 6→7 텍스트 3편 | 3편 published |
| W26 | 전체 어휘 연속성 검증 (cross-stage vocabulary audit) | 검증 보고서 |
| W26 | Stage 진급 로직 구현 + 테스트 | 진급 기능 동작 |
| W27 | Analytics 대시보드 (교사용) | 화면 동작 |

### Phase 6: AI 에이전트 연동 (3주)

| 주 | 작업 | 산출물 |
|---|------|--------|
| W28 | Orchestrator → Reading Ladder 추천 API | 연동 동작 |
| W28 | 어휘 코치 → text_vocabulary 연동 | 5D 퀴즈 연계 |
| W29 | 문항생성 Agent → 추가 문항 생성 API | 동작 테스트 |
| W29 | 분석 리포트 Agent → 학급별 집계 API | 리포트 생성 |
| W30 | E2E 테스트 + 버그 수정 + 성능 최적화 | v1.0 출시 준비 |

**총 기간: 30주 (약 7.5개월)**

---

## 13. 비용 추정

### 13.1 개발 비용 (도구/서비스)

| 항목 | 단가 | 수량 | 비용 |
|------|------|------|------|
| GPT-4o 본문 생성 | ~$0.03/편 | 200편 | $6 |
| GPT-4o 문법 태깅 | ~$0.01/편 | 200편 | $2 |
| GPT-4o 문제 생성 | ~$0.02/편 | 200편 | $4 |
| GPT-4o 검증/수정 | ~$0.02/편 | 200편 | $4 |
| OpenAI TTS-1-HD | $30/1M chars | ~350K chars | ~$11 |
| **LLM + TTS 소계** | | | **~$27** |

### 13.2 운영 비용 (월간)

| 서비스 | 플랜 | 월 비용 |
|--------|------|---------|
| Supabase | Free (500MB, 50K MAU) | $0 |
| Supabase | Pro (필요시) | $25/월 |
| Neo4j Desktop | 로컬 무료 | $0 |
| Vercel | Hobby | $0 |
| OpenAI API (운영 중 추가 생성) | 사용량 기반 | ~$5/월 |
| **월 소계** | | **$0 ~ $30** |

### 13.3 총 비용 요약

| 구분 | 비용 |
|------|------|
| 초기 구축 (LLM + TTS) | ~$27 |
| 월 운영 (Free tier) | $0 |
| 월 운영 (Pro tier) | ~$30 |
| **연간 운영** | **$0 ~ $360** |

---

## 14. 리스크 & 완화 전략

| # | 리스크 | 확률 | 영향 | 완화 전략 |
|---|--------|------|------|----------|
| R1 | Lexile 점수 부정확 | 높음 | 중 | FK 기반 근사 + 교사 검수 + `lexile_source` 필드로 투명성 확보 |
| R2 | LLM 생성 텍스트 CEFR 불일치 | 중 | 높음 | Step 3 자동 검증 + 최대 3회 재생성 + 전문가 검수 |
| R3 | Neo4j API 통합 실패 | 중 | 높음 | Phase 0에서 POC 선행, fallback으로 text_vocabulary만으로 동작 |
| R4 | 콘텐츠 생산 일정 초과 | 높음 | 중 | 파이프라인 자동화 우선, Stage 1-4 먼저 출시 (MVP) |
| R5 | 저작권 이슈 | 낮음 | 높음 | Classic Stories 전편 공공 도메인 확인 (저작권 만료 70년+) |
| R6 | Supabase Free tier 초과 | 중 | 낮음 | 200편 + 메타데이터 ≈ 50MB, Free 500MB 내 충분 |
| R7 | 5D 앱 DB 마이그레이션 실패 | 중 | 높음 | 로컬 SQLite 백업 유지, 단계적 전환, 롤백 스크립트 |
| R8 | 어휘 연속성 보장 어려움 | 중 | 중 | cross-stage audit 스크립트, Stage 전이 시 어휘 체크리스트 |

---

## 15. 성공 지표

### 15.1 학습 효과 지표

| 지표 | 측정 방법 | 목표 |
|------|----------|------|
| Stage 진급률 | 월간 stage_promotions 달성 비율 | 70%+ |
| 이해도 점수 추이 | reading_progress.comprehension_score 월간 평균 | 5%+/월 상승 |
| 어휘 숙달도 | text_vocabulary 기반 퀴즈 정답률 | Stage 내 80%+ |
| 학습 지속성 | 주간 활성 사용자 / 전체 등록 사용자 | 60%+ |
| 5D 프로필 성장 | KnowledgeProfile 차원별 점수 추이 | 전 차원 월 3%+ |

### 15.2 플랫폼 지표

| 지표 | 목표 |
|------|------|
| 콘텐츠 완성률 | 200편 중 95%+ published |
| 자동 파이프라인 성공률 | Step 1-7 무수정 통과 70%+ |
| 교사 텍스트 활용률 | 등록 교사의 80%+ 주 1회 이상 사용 |
| 에이전트 연동 성공률 | API 호출 오류율 < 2% |

---

## 16. 부록

### 부록 A: Stage 2 Aesop's Fables 전체 목록

*(READING_TEXT_DB_PLAN.md Section 2 — Stage 2 참조)*

### 부록 B: Stage 3 Classic Stories 전체 목록

*(READING_TEXT_DB_PLAN.md Section 2 — Stage 3 참조)*

### 부록 C: Stage 4 Great Lives 전체 목록

*(READING_TEXT_DB_PLAN.md Section 2 — Stage 4 참조)*

### 부록 D: Stage 6 — 40개 토픽 전체 목록

*(READING_TEXT_DB_PLAN.md Section 2 — Stage 6 참조)*

### 부록 E: 기존 시스템 스키마 참조

| 시스템 | 스키마 파일 |
|--------|-----------|
| 5D 어휘학습 Prisma | `5dimension_vocablearning/prisma/schema.prisma` |
| 5D 어휘학습 DB 함수 | `5dimension_vocablearning/lib/db.ts` |
| Neo4j 그래프 스키마 | `vocab-knowledge-graph/packages/graph-schema/src/schema.ts` |
| Neo4j Repository | `vocab-knowledge-graph/packages/graph-client/src/vocab-repository.ts` |
| Neo4j API Routes | `vocab-knowledge-graph/apps/api/app/api/` (7개 엔드포인트) |

### 부록 F: 관련 문서

| 문서 | 위치 |
|------|------|
| 콘텐츠 상세 계획 | `E:\01_Projects\reading-roadmap\READING_TEXT_DB_PLAN.md` |
| 기술 검토서 | `E:\01_Projects\reading-roadmap\TECH_REVIEW.md` |
| 어휘 그래프 이관 문서 | `E:\01_Projects\vocab-knowledge-graph\HANDOFF.md` |
| AI TESOL 팀 에이전트 계획 | `C:\Users\user\CLAUDE.md` |

---

*PRD v1.0 — 2026-02-28*
*AI TESOL 멀티에이전트 프로젝트*
