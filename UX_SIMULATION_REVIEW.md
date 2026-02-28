# UX Simulation & PRD Gap Analysis

> 3개 페르소나 × 전체 사용 여정 시뮬레이션 → PRD 개선점 도출
> 작성일: 2026-02-28

---

## 시뮬레이션 A: 초등 5학년 학생 (민지)

**프로필:** 11세, CEFR A1-A2, 영어학원 다님, 학교에서 기초 영어만 배움, 엄마가 가입시켜줌

---

### Step 1: 가입 & 온보딩

```
민지가 앱에 접속한다.
→ 랜딩 페이지: "파닉스부터 학술 지문까지..."
→ [시작하기] 클릭 → 회원가입 페이지

이메일/비밀번호 입력, 이름: "민지", 가입 완료.
→ /dashboard로 이동
```

**🔴 문제 1: 배치 테스트(Placement Test)가 없다**

민지의 현재 수준이 Stage 1인지 Stage 2인지 알 수 없다. 시스템도 모른다.
- 대시보드에 7개 Stage가 모두 보이지만, 어디서 시작해야 할지 가이드가 없음
- 5D 레이더 차트도 전부 0점 — 의미 없는 화면
- "추천 텍스트"도 데이터가 없어 추천 불가

**🔴 문제 2: 온보딩 플로우가 없다**

가입 직후 빈 대시보드로 떨어지면 초등학생은 무엇을 해야 할지 모른다.
- "처음이에요? 여기서 시작하세요!" 같은 안내가 없음
- 엄마가 옆에서 도와줘야 하는 상황

**🟡 문제 3: 연령/학년 정보를 수집하지 않는다**

User 모델에 `role`만 있고, `grade`(학년), `birthYear` 등이 없다.
Stage 추천에 연령 정보가 핵심인데 수집하지 않음.

---

### Step 2: 첫 텍스트 선택

```
민지(또는 엄마)가 Stage Map을 본다.
→ Stage 1 "Phonics Readers" 클릭
→ /reading 페이지, stage=1 필터

20개 파닉스 리더가 보인다.
"A Cat and a Hat"을 클릭한다.
```

**🟡 문제 4: 텍스트 순서 가이드가 없다**

20편이 나열되지만 #1부터 순서대로 읽어야 한다는 안내가 없다.
파닉스는 음소 순서(short a → short e → ...)가 교육적으로 중요한데,
학생이 랜덤으로 #19 "The Knight's Bright Light"를 먼저 읽으면 학습 효과 저하.

**개선안:** `orderIndex` 또는 `recommendedOrder` 필드 추가, UI에서 순서 표시 + "다음 추천" 하이라이트

---

### Step 3: 텍스트 읽기

```
Reading Viewer가 열린다:
- 제목: "A Cat and a Hat"
- 본문: "A cat sat on a mat. The cat had a hat..." (30 words)
- 오른쪽: 학습 어휘 패널 (cat, hat, mat, sat)
- 🔊 오디오 버튼
```

**🔴 문제 5: Pre-reading 활동이 없다**

READING_TEXT_DB_PLAN.md에서는 "Pre-reading: 표지 예측, 핵심 어휘 사전학습"을 명시했지만,
앱 화면에는 바로 본문이 보인다.

초등학생에게는 읽기 전 활동이 필수:
- "이 그림을 보고 무슨 이야기일까 생각해보세요"
- "이 단어들을 먼저 들어보세요: cat, hat, mat"

**🟡 문제 6: 삽화가 어떻게 표시되는지 불명확**

Stage 1은 삽화 비율 70-80%인데, `illustrationRatio` 필드만 있고
실제 이미지 저장/표시 방식이 PRD에 없다.
- `ReadingText`에 이미지 URL 배열 필드가 없음
- 문단별 삽화 매핑 구조가 없음

**🟡 문제 7: 오디오 먼저 듣기 옵션이 없다**

파닉스 단계에서는 "Listen First → Read Along → Read Alone" 순서가 교육적으로 효과적.
현재 UI는 텍스트가 먼저 보이고 오디오는 버튼 클릭.
- "듣기 모드" / "읽기 모드" 전환이 필요

---

### Step 4: 이해도 문제 풀기

```
본문 아래 Comprehension Check:
Q1. "The cat sat on the ______."
  ○ hat  ○ mat  ○ bat  ○ rat

민지가 "mat"을 선택 → ✅ Correct!

5문제 중 4문제 정답 → 80점
```

**🟡 문제 8: 오답 피드백이 빈약하다**

`explanation` 필드가 있지만, 초등학생에게는:
- "정답은 mat이에요. cat이 mat(매트) 위에 앉았어요" → 한국어 설명 필수
- 오답 선택 시 "다시 읽어보기" 버튼으로 해당 문장 하이라이트

현재 PRD에는 오답 시 어떤 UX인지 명시되어 있지 않음.

**🔴 문제 9: 재시도(Retry) 메커니즘이 없다**

80점을 받았는데, 다시 풀 수 있는가? `readCount`는 있지만
"문제 다시 풀기"와 "텍스트 다시 읽기"를 구분하는 필드가 없음.

---

### Step 5: 학습 완료 & 다음 텍스트

```
민지가 "A Cat and a Hat"을 완료했다.
→ reading_progress에 저장: status='completed', comprehension_score=80
→ 대시보드로 돌아간다.
```

**🔴 문제 10: 완료 보상/동기부여가 없다**

초등학생에게 가장 중요한 것 = **즉각적 보상**
- 완료 후 "축하해요! 🌟" 팝업 → 없음
- 포인트/별/뱃지 시스템 → 없음
- "다음 이야기" 자동 추천 → 없음
- 스트릭 알림 → 없음

`ReadingProgress`에 보상 관련 필드가 전혀 없음.

**🟡 문제 11: "다음에 읽을 텍스트" 동선이 끊긴다**

텍스트 완료 후 → 대시보드 → 다시 Reading → 다시 필터 → 다음 텍스트 클릭
이 4단계가 매번 반복됨.

"다음 텍스트 읽기 →" 버튼으로 바로 이동하는 UX가 필요.

---

### Step 6: 2주 후 — Stage 진급

```
민지가 Stage 1의 20편 중 16편을 완료했다. (80%)
평균 이해도: 75점
→ 진급 조건 충족 (70% 완료, 70점 이상)
→ ???
```

**🔴 문제 12: 진급 알림 & 세레모니가 없다**

`/api/stage/promotion` API는 있지만:
- 언제 이 API가 호출되는가? (매 완료 시? 대시보드 로드 시?)
- 진급 조건 충족 시 어떤 UI가 표시되는가?
- "Stage 2 해금!" 축하 화면이 없음

**🟡 문제 13: Stage 잠금(Locking) 정책이 불명확**

Stage 1을 완료하지 않아도 Stage 5를 읽을 수 있는가?
PRD에 Stage 잠금/해금 정책이 명시되어 있지 않음.

---

## 시뮬레이션 B: 고등 2학년 학생 (준호)

**프로필:** 17세, CEFR B1-B2, 수능 대비, 영어 빈칸추론이 약점, 혼자 공부

---

### Step 1: 가입 & Stage 건너뛰기

```
준호가 가입한다.
→ 대시보드: 모든 Stage 0%
→ "나는 Stage 6 수능 독해가 필요한데..."
→ Stage 1 파닉스부터 시작해야 하나?
```

**🔴 문제 14: 배치 테스트 부재 (재확인, 가장 치명적)**

고등학생이 파닉스부터 시작하는 것은 심각한 UX 실패.
배치 테스트가 없으면:
- Stage 건너뛰기를 허용? → 그러면 진급 시스템이 무의미
- 강제로 Stage 1부터? → 고등학생이 이탈

이것이 PRD의 **가장 큰 결함**.

---

### Step 2: 수능 유형별 연습

```
준호가 Stage 6에 접근한다 (잠금이 없다고 가정).
→ /reading?stage=6&csatType=blank_inference
→ 빈칸추론 8편이 표시된다.

"AI and Human Creativity" 토픽을 선택한다.
```

**🟡 문제 15: 난이도 순서가 없다**

`csatDifficulty` (basic/intermediate/advanced)가 있지만:
- 목록에서 난이도 순 정렬이 되는가?
- "기초 → 중급 → 고급" 추천 경로가 있는가?
- 같은 유형(빈칸추론) 내에서의 학습 순서 가이드 없음

**🟡 문제 16: 수능 실전 환경 시뮬레이션이 없다**

수능은 **시간 제한** 하에 풀어야 함:
- 지문 1개당 약 2분 배정
- 타이머 기능이 없음 (time_spent는 저장하지만 타이머 UI 없음)
- "실전 모드" (타이머 ON, 한 번만 풀기) vs "연습 모드" (무제한, 해설 보기) 구분 없음

---

### Step 3: 지문 읽기 & 문제 풀기

```
500 words 지문을 읽는다.
어휘 패널에 15개 학습 어휘가 있다.

빈칸추론 문제:
"The author argues that the key difference between
AI-generated art and human art lies in the ________."

① intentionality of creative expression
② technical precision of execution
③ aesthetic quality of the output
④ commercial value of the work
```

**🟡 문제 17: 해설이 부족하다**

수능 대비에서 핵심은 **왜 틀렸는지** 이해하는 것:
- 현재: `explanation` 필드 하나에 텍스트 설명
- 필요한 것:
  - 정답 근거가 되는 문장 하이라이트
  - 각 오답이 왜 틀린지 개별 해설
  - 지문 구조 분석 (주제문, 뒷받침 문장, 결론)

**🔴 문제 18: 오답 패턴 분석이 없다**

준호가 빈칸추론 8편 중 5편을 풀었는데:
- "논리 추론" 유형에서 자주 틀린다
- "문맥 파악" 유형은 잘 맞춘다

이런 분석이 없음. `TextQuestion.skill` (literal/inferential/evaluative/creative) 필드가 있지만,
이를 집계해서 "너는 inferential 스킬이 약해"라고 알려주는 분석 화면이 없음.

---

### Step 4: 어휘 학습 연계

```
준호가 "intentionality"라는 단어를 모른다.
→ 어휘 팝업 클릭: 뜻, 발음, 5D 차원
→ "이 단어를 5D 퀴즈로 연습하고 싶다"
```

**🔴 문제 19: 5D 어휘 앱으로의 핸드오프가 없다**

PRD에서 Agent 간 연동을 설명했지만, 실제 UI에서:
- "이 단어를 5D 퀴즈로 학습하기" 버튼이 없음
- 텍스트에서 모른 단어를 모아 5D 앱으로 보내는 기능 없음
- 두 앱이 별도 URL/프로젝트인 경우, 어떻게 이동하는가?

**🟡 문제 20: 단어장(Word Bank) 기능이 없다**

학생이 모르는 단어를 수집하는 개인 단어장이 없다.
`UserVocabulary` 모델은 있지만, Reading Viewer에서 "단어장에 추가" 기능이 없음.

---

### Step 5: Stage 6 → Stage 7 전이

```
준호가 Stage 6을 충분히 연습한 후,
"같은 주제의 더 깊은 학술 지문을 읽고 싶다"

Stage 6 "Cognitive Biases" → Stage 7 "Kahneman 이중 처리 이론"
이 연결이 보이는가?
```

**🟡 문제 21: Stage 6 → 7 토픽 연결이 UI에 드러나지 않는다**

데이터 설계에서 Stage 7은 Stage 6 토픽의 학술적 심화로 설계했지만:
- Reading Viewer에 "이 토픽의 심화 학술 지문 보기 →" 링크가 없음
- Stage 7 텍스트에 `topicNumber`가 같은 Stage 6 텍스트로 역참조 불가
- "관련 텍스트" 추천 기능이 없음

---

## 시뮬레이션 C: 중학교 영어 교사 (김선생님)

**프로필:** 35세, 중2 영어 담당, 학생 30명, 수준별 리딩 자료 필요

---

### Step 1: 교사 가입 & 학급 설정

```
김선생님이 가입한다.
→ role: "teacher" 선택
→ /console (교사 콘솔)로 이동

"우리 반 학생 30명을 등록하고 싶다"
```

**🔴 문제 22: 학급(Class) 모델이 없다**

이것이 교사 기능의 **가장 큰 결함**:
- DB 스키마에 `Class`, `ClassMembership` 테이블이 없음
- 교사가 학생을 그룹화할 수 없음
- "우리 반 학생들의 진도를 보고 싶다" → 불가능
- `/api/analytics/class`가 있지만 어떤 학생이 "우리 반"인지 알 수 없음

**필요한 추가 모델:**
```
Class { id, name, teacherId, gradeLevel, createdAt }
ClassMembership { classId, studentId, joinedAt }
```

**🟡 문제 23: 교사 역할 인증이 불명확**

가입 시 `role: "teacher"`를 자기 스스로 선택하면 학생도 교사를 선택할 수 있음.
- 교사 인증 절차(이메일 도메인? 인증 코드?)가 없음
- 교사가 학생 데이터에 접근하는 권한 체크가 RLS에서 주석 처리됨

---

### Step 2: 텍스트 할당

```
김선생님이 중2 학생들에게 Stage 3-4 텍스트를 할당하고 싶다.
→ "이번 주에 'Robinson Crusoe'를 읽어오세요"
```

**🔴 문제 24: 텍스트 할당(Assignment) 기능이 없다**

교사가 학생에게 특정 텍스트를 "과제"로 할당하는 기능이 PRD에 없음:
- 할당 모델이 없음 (Assignment, TextAssignment)
- 마감일 설정 불가
- 할당된 과제 vs 자유 읽기 구분 불가
- 학생이 과제 완료 여부를 교사가 확인할 수 없음

**필요한 추가 모델:**
```
Assignment { id, classId, textId, teacherId, dueDate, instructions, createdAt }
AssignmentSubmission { assignmentId, studentId, progressId, submittedAt }
```

---

### Step 3: 학급 분석

```
김선생님이 학급 분석 페이지를 연다.
→ /analytics
→ "학급 평균 이해도가 어디에 위치하는지 보고 싶다"
```

**🟡 문제 25: 분석 범위가 제한적**

현재 분석 API:
- 개인별 Stage 진도, 5D 프로필, 이해도 추이
- 학급 평균

**부족한 분석:**
- 특정 텍스트별 학급 정답률 (어떤 지문이 어려웠나?)
- 특정 문제별 오답률 (어떤 문제가 어려웠나?)
- 학생 간 비교 (상위/중위/하위 그룹)
- 어휘 숙달도 히트맵 (어떤 어휘를 학급 대다수가 모르는가?)
- 시간 기반 분석 (이번 주 vs 지난 주)

**🟡 문제 26: 리포트 내보내기가 없다**

교사가 분석 결과를 학부모에게 보내거나 학교에 제출해야 할 때:
- PDF 내보내기 없음
- Excel/CSV 다운로드 없음
- 프린트 최적화 레이아웃 없음

---

### Step 4: 토픽 커스텀

```
김선생님이 Stage 6 토픽 중 "환경" 영역에
"Plastic Pollution in the Ocean"을 추가하고 싶다.
→ /topics 페이지
→ 토픽 추가
```

**🟡 문제 27: 토픽만 추가되고 텍스트는 없다**

`UserTopic` 모델은 토픽 메타데이터만 저장.
실제 지문을 작성/업로드하는 기능이 없음.
- 토픽을 추가해도 해당 토픽의 지문이 자동 생성되는가? (콘텐츠 파이프라인 수동 트리거?)
- 교사가 직접 지문을 붙여넣는 기능이 있어야 하는가?

PRD에서 "사용자 직접 텍스트 업로드 — Out of Scope (v1)"이라고 했지만,
교사의 핵심 니즈와 충돌함.

---

## 종합: PRD 갭 분류

### 🔴 Critical — 핵심 기능 누락 (6건)

| # | 갭 | 영향 | 해당 페르소나 |
|---|---|------|-------------|
| G1 | **배치 테스트(Placement Test) 없음** | 학생이 적절한 Stage를 찾지 못함. 고등학생이 파닉스부터 시작해야 하는 UX 재앙 | 전원 |
| G2 | **온보딩 플로우 없음** | 가입 후 빈 대시보드 → 초등학생 이탈 | 학생(초등) |
| G3 | **학급(Class) 모델 없음** | 교사가 학생을 그룹화할 수 없어 교사 기능 전체가 무용화 | 교사 |
| G4 | **과제 할당(Assignment) 기능 없음** | 교사→학생 과제 부여 불가, 교육 현장 핵심 워크플로 | 교사 |
| G5 | **완료 보상/게이미피케이션 없음** | 초등학생 학습 동기 유지 불가 | 학생(초등) |
| G6 | **5D 앱 핸드오프 UX 없음** | Agent 연동을 설계했지만 실제 UI 동선이 없음 | 전원 |

### 🟡 Important — 중요하지만 v1 우회 가능 (12건)

| # | 갭 | 영향 |
|---|---|------|
| G7 | 텍스트 읽기 순서 가이드 없음 | 파닉스 음소 순서 무시 가능 |
| G8 | Pre-reading 활동 UI 없음 | 읽기 전략 교육 누락 |
| G9 | 삽화 저장/표시 구조 없음 | Stage 1-3 삽화 비율 70-30% 구현 불가 |
| G10 | 듣기 모드(Listen First) 없음 | 파닉스 학습 효과 저하 |
| G11 | 오답 재시도(Retry) 메커니즘 없음 | 학습 기회 제한 |
| G12 | "다음 텍스트" 연속 읽기 동선 없음 | 매번 목록으로 돌아가야 함 |
| G13 | Stage 잠금/해금 정책 불명확 | 자유 접근 vs 순차 접근 미정 |
| G14 | 수능 실전 모드(타이머) 없음 | 시간 관리 연습 불가 |
| G15 | 오답 패턴 분석 없음 | skill별 약점 파악 불가 |
| G16 | 개인 단어장(Word Bank) 없음 | 모르는 단어 수집 불가 |
| G17 | Stage 6→7 토픽 연결 UI 없음 | 학술 심화 경로 보이지 않음 |
| G18 | 리포트 내보내기(PDF/CSV) 없음 | 교사 오프라인 활용 불가 |

### ⚪ Nice-to-have — v2에서 검토 (5건)

| # | 갭 |
|---|---|
| G19 | 읽기 속도(WPM) 측정 |
| G20 | 텍스트 내 하이라이트/메모 기능 |
| G21 | 어휘 복습 SRS(Spaced Repetition) |
| G22 | 학생 간 리더보드/경쟁 요소 |
| G23 | 오프라인 읽기 지원 (PWA) |

---

## 개선안 상세

### 개선 1: 배치 테스트 (Placement Test) 추가

**화면:** `/placement` (가입 직후 리다이렉트)

```
"어디서 시작할지 알아볼까요?"
"간단한 문제 10개만 풀어보세요 (약 5분)"

Q1. [Stage 1 수준] Look at the picture. The cat is on the _____.
Q2. [Stage 2 수준] The fox wanted the grapes because they looked _____.
Q3. [Stage 3 수준] Robinson Crusoe decided to build a shelter because...
Q4. [Stage 4 수준] Marie Curie's discovery of radium led to...
Q5. [Stage 5 수준] The article suggests that renewable energy...
Q6-7. [Stage 6 수준] 빈칸추론/순서배열 (수능 유형)
Q8-10. [Stage 7 수준] 학술 지문 이해

→ 정답 패턴 기반 시작 Stage 결정
→ "당신의 읽기 수준: Stage 3 (CEFR A2-B1)입니다!"
→ Stage Map에서 Stage 3이 하이라이트됨
```

**추가 모델:**
```prisma
model PlacementResult {
  id             String   @id @default(uuid())
  userId         String   @unique @map("user_id")
  assignedStage  Int      @map("assigned_stage")
  score          Float
  responses      Json     // 각 문제별 응답
  completedAt    DateTime @default(now()) @map("completed_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("placement_results")
}
```

**추가 API:**
```
POST /api/placement    — 배치 테스트 결과 저장 + Stage 결정
GET  /api/placement    — 배치 결과 조회
```

---

### 개선 2: 온보딩 플로우

**가입 직후 3단계:**

```
Step 1: 프로필 완성
  "이름이 뭐예요?"
  "몇 학년이에요?" [초3 ~ 고3, 대학생/성인]
  "선생님이에요, 학생이에요?" [학생 | 교사]

Step 2: 배치 테스트
  (개선 1 참조)

Step 3: 첫 텍스트 추천
  "당신에게 딱 맞는 첫 번째 이야기를 골라볼까요?"
  → 배치 결과 Stage의 첫 번째 텍스트 3개 제안
  → [읽으러 가기 →]
```

**추가 필드 (User 모델):**
```prisma
model User {
  // 기존 필드 +
  grade        String?  // 초3, 초4, ..., 고3, 대학, 성인
  onboarded    Boolean  @default(false)
  currentStage Int      @default(1) @map("current_stage")
}
```

---

### 개선 3: 학급 & 과제 모델

```prisma
model Class {
  id         String   @id @default(uuid())
  name       String   // "중2-3반"
  teacherId  String   @map("teacher_id")
  gradeLevel String?  @map("grade_level")
  inviteCode String   @unique @map("invite_code") // 학생 초대 코드
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

  class       Class       @relation(fields: [classId], references: [id], onDelete: Cascade)
  text        ReadingText @relation(fields: [textId], references: [id])
  submissions AssignmentSubmission[]

  @@map("assignments")
}

model AssignmentSubmission {
  id           String    @id @default(uuid())
  assignmentId String    @map("assignment_id")
  studentId    String    @map("student_id")
  progressId   String?   @map("progress_id")
  submittedAt  DateTime? @map("submitted_at")

  assignment Assignment @relation(fields: [assignmentId], references: [id], onDelete: Cascade)

  @@unique([assignmentId, studentId])
  @@map("assignment_submissions")
}
```

**교사 워크플로:**
```
1. 학급 생성 → 초대 코드 발급 (예: "ABC123")
2. 학생에게 코드 공유 → 학생이 코드 입력하여 학급 참가
3. 교사가 텍스트 선택 → "이 반에 할당" → 마감일 설정
4. 학생 대시보드에 "과제" 섹션 표시
5. 교사 콘솔에서 제출 현황 확인
```

---

### 개선 4: 게이미피케이션 기본 요소

```prisma
model Achievement {
  id          String   @id @default(uuid())
  userId      String   @map("user_id")
  type        String   // first_read, stage_complete, streak_7, streak_30, vocab_100, ...
  earnedAt    DateTime @default(now()) @map("earned_at")
  metadata    Json?    // { stage: 1, textTitle: "..." }

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("achievements")
}
```

**뱃지 유형:**
| 뱃지 | 조건 | 표시 |
|------|------|------|
| First Reader | 첫 텍스트 완료 | 🌟 |
| Stage Master 1-7 | Stage 완료 | 🏅 |
| Streak 7 | 7일 연속 학습 | 🔥 |
| Streak 30 | 30일 연속 학습 | 💎 |
| Word Collector 50/100/500 | 학습 어휘 수 | 📚 |
| Perfect Score | 이해도 100점 | ⭐ |

**완료 화면:**
```
┌─────────────────────────────────┐
│                                 │
│     🎉 "A Cat and a Hat"       │
│        읽기 완료!               │
│                                 │
│     이해도: ⭐⭐⭐⭐☆ 80점      │
│     학습 어휘: 4개              │
│     읽기 시간: 3분 22초          │
│                                 │
│     🌟 뱃지 획득: First Reader! │
│                                 │
│     [다음 이야기 →]              │
│     [대시보드로]                 │
└─────────────────────────────────┘
```

---

### 개선 5: Reading Viewer 3단계 플로우

Pre-reading → While-reading → Post-reading을 탭 또는 스텝으로 구현:

```
┌─────────────────────────────────────────────────────┐
│  [1. 읽기 전] → [2. 읽기] → [3. 읽기 후]            │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. 읽기 전 (Pre-reading)                           │
│  ┌─────────────────────────────────────────────┐    │
│  │ 🖼️ [삽화 미리보기]                          │    │
│  │                                             │    │
│  │ "이 그림을 보고 무슨 이야기일까 생각해보세요"  │    │
│  │                                             │    │
│  │ 핵심 어휘 미리 듣기:                         │    │
│  │   🔊 treasure  🔊 island  🔊 map            │    │
│  │                                             │    │
│  │ [읽기 시작하기 →]                            │    │
│  └─────────────────────────────────────────────┘    │
│                                                     │
│  2. 읽기 (While-reading) → 기존 Reading Viewer      │
│                                                     │
│  3. 읽기 후 (Post-reading)                          │
│  ┌─────────────────────────────────────────────┐    │
│  │ 📝 Comprehension Check                      │    │
│  │ 📝 Vocabulary Exercises                     │    │
│  │ 💬 "이 이야기의 교훈은 무엇인가요?" (주관식) │    │
│  │ 📊 결과 & 다음 텍스트 추천                   │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

---

### 개선 6: 삽화 모델 추가

```prisma
model TextIllustration {
  id             String  @id @default(uuid())
  textId         String  @map("text_id")
  imageUrl       String  @map("image_url")    // Supabase Storage
  altText        String? @map("alt_text")
  position       String  @default("top")       // top, inline, paragraph_N
  paragraphIndex Int?    @map("paragraph_index") // inline일 경우 몇 번째 문단 뒤
  orderIndex     Int     @default(0) @map("order_index")

  text ReadingText @relation(fields: [textId], references: [id], onDelete: Cascade)

  @@map("text_illustrations")
}
```

---

## 변경 영향도 요약

### 스키마 추가 (6개 모델)

| 모델 | 용도 | 우선순위 |
|------|------|---------|
| `PlacementResult` | 배치 테스트 결과 | Sprint 1 (Critical) |
| `Class` | 교사 학급 | Sprint 6 |
| `ClassMembership` | 학급-학생 매핑 | Sprint 6 |
| `Assignment` | 과제 할당 | Sprint 6 |
| `AssignmentSubmission` | 과제 제출 | Sprint 6 |
| `Achievement` | 뱃지/업적 | Sprint 3 |
| `TextIllustration` | 삽화 | Sprint 2 |

### User 모델 확장

```diff
model User {
   // 기존 +
+  grade        String?
+  onboarded    Boolean  @default(false)
+  currentStage Int      @default(1)
+
+  placementResult PlacementResult?
+  achievements    Achievement[]
+  teacherClasses  Class[]           @relation("TeacherClasses")
+  studentClasses  ClassMembership[] @relation("StudentClasses")
}
```

### 추가 페이지 라우트

```
+ /placement              — 배치 테스트
+ /onboarding             — 온보딩 3단계
+ /reading/[id]/complete   — 완료 축하 화면
+ /class/[id]              — 학급 상세 (교사)
+ /class/[id]/assignments  — 과제 관리 (교사)
+ /assignments             — 내 과제 목록 (학생)
```

### 추가 API

```
POST /api/placement
GET  /api/placement
POST /api/class              — 학급 생성
POST /api/class/[id]/invite  — 초대 코드 검증 + 참가
GET  /api/class/[id]/students — 학급 학생 목록
POST /api/assignments        — 과제 생성
GET  /api/assignments        — 내 과제 목록
POST /api/achievements       — 뱃지 확인 + 부여
```

### Sprint 계획 수정

| 원래 Sprint | 추가 작업 |
|------------|----------|
| Sprint 1 | + 배치 테스트 + 온보딩 |
| Sprint 2 | + 삽화 모델 + 콘텐츠 파이프라인에 이미지 생성 추가 |
| Sprint 3 | + 완료 화면 + Achievement 시스템 + Pre/While/Post-reading 탭 |
| Sprint 4 | + "다음 텍스트" 연속 동선 + 진급 세레모니 |
| Sprint 5 | (콘텐츠 생성, 변경 없음) |
| Sprint 6 | + Class/Assignment 모델 전체 구현 (기존 교사 기능 확장) |
| Sprint 7 | + 오답 패턴 분석 + 수능 실전 모드 (타이머) |

---

*UX Simulation Review v1.0 — 2026-02-28*
