# EFL Reading Roadmap & Text DB 구축 계획

> 비영어권(한국) EFL 학습자를 위한 단계별 리딩 로드맵
> 파닉스 동화 → 초등 영어동화 → 수능 지문 → 미국 Liberal Arts 학술 지문

---

## 1. 전체 프레임워크 개요

### 1.1 단계 구조 (7-Stage Reading Ladder)

```
Stage 7 ─ Academic Discourse        ─ CEFR C1    ─ 대학/성인
Stage 6 ─ CSAT & Critical Reading   ─ CEFR B2+   ─ 고등 2-3
Stage 5 ─ Future World & Careers    ─ CEFR B2    ─ 고등 1-2
Stage 4 ─ Great Lives (위인전)       ─ CEFR B1+   ─ 중등 2-3
Stage 3 ─ Classic Stories           ─ CEFR B1    ─ 중등 1-2
Stage 2 ─ Aesop's Fables           ─ CEFR A2    ─ 초등 5-6
Stage 1 ─ Phonics Readers          ─ CEFR A1    ─ 초등 3-4
```

### 1.2 핵심 매개변수 매핑

| Stage | CEFR | Lexile | 어휘 수준 | 문장 길이 | 지문 길이 | 대상 연령 |
|-------|------|--------|-----------|-----------|-----------|-----------|
| 1 | Pre-A1~A1 | <200L | ~500 words | 3-7 words | 30-80 words | 초3-4 (9-10세) |
| 2 | A1~A2 | 200-450L | ~1,500 words | 6-10 words | 80-150 words | 초5-6 (11-12세) |
| 3 | A2~B1 | 450-650L | ~2,500 words | 8-14 words | 150-300 words | 중1-2 (13-14세) |
| 4 | B1 | 650-800L | ~3,000 words | 10-16 words | 250-400 words | 중2-3 (14-15세) |
| 5 | B1+~B2 | 800-950L | ~3,500 words | 12-18 words | 300-500 words | 고1-2 (16-17세) |
| 6 | B2~B2+ | 950-1100L | ~4,500 words | 14-22 words | 400-700 words | 고2-3 (17-18세) |
| 7 | C1 | 1100L+ | 5,000+ words | 18-30 words | 600-1200 words | 대학/성인 |

---

## 2. Stage별 상세 설계

---

### Stage 1: Phonics Readers (20권)

**대상:** 초등 3-4학년 / CEFR Pre-A1~A1
**목표:** 음소-문자 대응(Grapheme-Phoneme Correspondence) 습득, 디코딩 자동화

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 단어 수/페이지 | 5-15 words |
| 총 단어 수/권 | 30-80 words |
| 문장 구조 | SV, SVO 단순문 |
| 어휘 | CVC(cat, dog, run), sight words(the, is, a) |
| 반복률 | 핵심 패턴 3회 이상 반복 |
| 삽화 비율 | 70-80% (그림이 텍스트를 보완) |

#### 20권 구성안

| # | 포커스 음소 | 제목 (예시) | 핵심 패턴 | 단어 수 |
|---|------------|------------|-----------|---------|
| 1 | short a | "A Cat and a Hat" | CVC: -at family | 30 |
| 2 | short e | "The Red Hen" | CVC: -en, -ed family | 35 |
| 3 | short i | "Big Pig's Dig" | CVC: -ig, -in family | 35 |
| 4 | short o | "Hot Dog on a Log" | CVC: -og, -ot family | 40 |
| 5 | short u | "The Bug in the Mug" | CVC: -ug, -un family | 40 |
| 6 | long a (a_e) | "Jake's Lake" | CVCe pattern | 45 |
| 7 | long i (i_e) | "Mike Rides a Bike" | CVCe pattern | 45 |
| 8 | long o (o_e) | "Rose and the Stone" | CVCe pattern | 50 |
| 9 | consonant blends (bl, cl) | "Blue Clams" | Initial blends | 50 |
| 10 | consonant blends (st, sp) | "Star Spin" | Initial blends | 50 |
| 11 | digraphs (sh, ch) | "Chip and the Ship" | Digraphs | 55 |
| 12 | digraphs (th, wh) | "The White Whale" | Digraphs | 55 |
| 13 | -ing endings | "Singing in the Ring" | Verb + -ing | 55 |
| 14 | -ed endings | "The Baked Cake" | Past tense basics | 60 |
| 15 | ee, ea | "The Green Bee Sees" | Long vowel teams | 60 |
| 16 | oo, ou | "Moon and Spoon" | Vowel teams | 60 |
| 17 | ar, or | "Far Star, Short Horse" | R-controlled vowels | 65 |
| 18 | ow, aw | "The Cow Saw Snow" | Diphthongs | 65 |
| 19 | 복합 패턴 1 | "The Knight's Bright Light" | Silent letters + blends | 70 |
| 20 | 복합 패턴 2 | "My Friend's Great Day" | 종합 패턴 | 80 |

#### 문법 요소
- be동사 (is, am, are)
- 현재 단순형 (I see, He runs)
- 명령문 (Look! Run! Stop!)
- This is / That is
- 관사 (a, the)

#### 한국인 EFL 학습자 특화 고려
- /r/ vs /l/ 구분 집중 (권 17-18에 배치)
- /f/ vs /p/ 구분 (권 5, 12에 포함)
- /v/ vs /b/ 구분
- 한국어에 없는 음소 클러스터(bl, cl, str) 추가 연습

---

### Stage 2: Aesop's Fables (20권)

**대상:** 초등 5-6학년 / CEFR A1~A2
**목표:** 서사 구조 이해(Beginning-Middle-End), 도덕적 교훈 파악, 기본 독해 전략 습득

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 총 단어 수/편 | 80-150 words |
| 문장 구조 | SVO, SVC + and/but 접속 |
| 어휘 | 고빈도 1,500단어 이내 + 5-8개 새 어휘/편 |
| 시제 | 과거 시제 중심 (Once upon a time...) |
| 삽화 비율 | 50-60% |

#### 20편 구성안

| # | 이야기 | 핵심 교훈 | 어휘 포커스 | 단어 수 |
|---|--------|----------|------------|---------|
| 1 | The Fox and the Grapes | 합리화(sour grapes) | reach, high, sweet, sour | 80 |
| 2 | The Tortoise and the Hare | 꾸준함 | slow, steady, race, rest | 90 |
| 3 | The Boy Who Cried Wolf | 정직 | village, shepherd, lie, trust | 100 |
| 4 | The Ant and the Grasshopper | 근면 | summer, winter, work, play | 90 |
| 5 | The Lion and the Mouse | 작은 도움의 가치 | trap, net, chew, free | 100 |
| 6 | The Dog and His Shadow | 욕심 | bridge, river, shadow, drop | 90 |
| 7 | The Crow and the Pitcher | 지혜 | thirsty, pebble, rise, clever | 100 |
| 8 | The North Wind and the Sun | 부드러움의 힘 | blow, warm, coat, gentle | 110 |
| 9 | The City Mouse and Country Mouse | 가치관 | city, country, danger, peace | 120 |
| 10 | The Goose That Laid Golden Eggs | 탐욕의 대가 | golden, egg, greedy, patient | 110 |
| 11 | The Wolf in Sheep's Clothing | 겉과 속 | disguise, pretend, careful | 110 |
| 12 | The Fox and the Stork | 배려 | invite, dinner, narrow, deep | 120 |
| 13 | The Bundle of Sticks | 단결 | together, break, strong, weak | 100 |
| 14 | The Frog and the Ox | 허풍 | bigger, burst, size, compare | 110 |
| 15 | The Milkmaid and Her Pail | 현실 집중 | dream, spill, plan, careful | 120 |
| 16 | The Lion's Share | 불공평한 힘 | share, divide, fear, power | 120 |
| 17 | Belling the Cat | 실행의 어려움 | bell, idea, brave, plan | 130 |
| 18 | The Peacock and the Crane | 외모 vs 능력 | beautiful, feather, fly, useful | 130 |
| 19 | The Oak and the Reed | 유연함 | storm, bend, stand, break | 140 |
| 20 | The Wolf and the Crane | 감사 | bone, throat, help, reward | 150 |

#### 문법 요소 (A1→A2 전이)
- 과거 단순형 (규칙/불규칙): walked, ran, said
- and, but, so 접속사
- because 이유절 (기초)
- 비교급: bigger, stronger, faster
- 의문사 질문: Who? What? Why?
- There was / There were

#### 학습 활동 연계
- **Pre-reading:** 표지 예측, 핵심 어휘 사전학습
- **While-reading:** 이야기 순서 배열, 등장인물 감정 추적
- **Post-reading:** 교훈 토론(L1 허용), 간단한 문장 완성 writing

---

### Stage 3: Classic Stories (30권)

**대상:** 중등 1-2학년 / CEFR A2~B1
**목표:** 확장된 서사 이해, 인물 심리 추론, 문맥 추론(contextual guessing) 전략

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 총 단어 수/편 | 150-300 words (축약 리텔링) |
| 문장 구조 | 복문(when, because, if) 도입 |
| 어휘 | 2,000-2,500단어 + 8-12개 새 어휘/편 |
| 장르 다양성 | 모험, 판타지, 미스터리, 유머, 감동 |
| 삽화 비율 | 30-40% |

#### 30편 구성안 (6개 하위 카테고리)

**A. 세계 명작 동화 (6편)**
| # | 작품 | 원작 | 단어 수 |
|---|------|------|---------|
| 1 | The Ugly Duckling | H.C. Andersen | 180 |
| 2 | The Little Mermaid | H.C. Andersen | 220 |
| 3 | Hansel and Gretel | Brothers Grimm | 200 |
| 4 | Cinderella | Charles Perrault | 220 |
| 5 | Jack and the Beanstalk | English folktale | 200 |
| 6 | The Emperor's New Clothes | H.C. Andersen | 240 |

**B. 모험 이야기 (6편)**
| # | 작품 | 원작 | 단어 수 |
|---|------|------|---------|
| 7 | Robinson Crusoe (excerpt) | Daniel Defoe | 250 |
| 8 | Treasure Island (excerpt) | R.L. Stevenson | 260 |
| 9 | Gulliver's Travels (Lilliput) | Jonathan Swift | 250 |
| 10 | Around the World in 80 Days (excerpt) | Jules Verne | 270 |
| 11 | The Jungle Book (Mowgli) | Rudyard Kipling | 260 |
| 12 | The Swiss Family Robinson (excerpt) | J.D. Wyss | 280 |

**C. 미스터리 & 추리 (5편)**
| # | 작품 | 원작 | 단어 수 |
|---|------|------|---------|
| 13 | Sherlock Holmes: The Red-Headed League | Conan Doyle | 280 |
| 14 | Sherlock Holmes: The Speckled Band | Conan Doyle | 290 |
| 15 | The Phantom of the Opera (excerpt) | Gaston Leroux | 260 |
| 16 | The Prince and the Pauper (excerpt) | Mark Twain | 270 |
| 17 | The Secret Garden (excerpt) | F.H. Burnett | 280 |

**D. 판타지 & SF (5편)**
| # | 작품 | 원작 | 단어 수 |
|---|------|------|---------|
| 18 | Alice in Wonderland (excerpt) | Lewis Carroll | 270 |
| 19 | The Wizard of Oz (excerpt) | L. Frank Baum | 260 |
| 20 | Peter Pan (excerpt) | J.M. Barrie | 280 |
| 21 | A Christmas Carol | Charles Dickens | 290 |
| 22 | The Time Machine (excerpt) | H.G. Wells | 300 |

**E. 감동 & 성장 (4편)**
| # | 작품 | 원작 | 단어 수 |
|---|------|------|---------|
| 23 | The Happy Prince | Oscar Wilde | 280 |
| 24 | Little Women (excerpt) | L.M. Alcott | 290 |
| 25 | Anne of Green Gables (excerpt) | L.M. Montgomery | 290 |
| 26 | Oliver Twist (excerpt) | Charles Dickens | 300 |

**F. 한국 & 아시아 이야기 (4편)**
| # | 작품 | 출처 | 단어 수 |
|---|------|------|---------|
| 27 | The Woodcutter and the Fairy | 한국 전래 | 200 |
| 28 | Kongjwi and Patjwi | 한국 전래 | 230 |
| 29 | Momotaro (The Peach Boy) | 일본 전래 | 210 |
| 30 | Journey to the West (excerpt) | 중국 고전 | 280 |

#### 문법 요소 (A2→B1 전이)
- 과거진행형: was/were + -ing
- when/while/because/if 종속절
- 현재완료 (경험): have been, have seen
- 비교급/최상급 확장
- 간접 의문문 도입: "Do you know where...?"
- 관계대명사 (기초): who, which, that
- 수동태 (기초): was made, was found

---

### Stage 4: Great Lives — 위인전 (30권)

**대상:** 중등 2-3학년 / CEFR B1~B1+
**목표:** 논픽션 독해, 시간순 서사, 원인-결과 관계 파악, 문화적 배경 이해

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 총 단어 수/편 | 250-400 words |
| 문장 구조 | 복합문, 분사구문 도입 |
| 어휘 | 2,500-3,000단어 + 10-15개 도메인 어휘/편 |
| 텍스트 유형 | 전기문(biography) |
| 삽화/사진 | 20-30% |

#### 30편 구성안 (6개 하위 카테고리)

**A. 과학자 & 발명가 (6편)**
| # | 인물 | 핵심 주제 | 단어 수 |
|---|------|----------|---------|
| 1 | Marie Curie | 과학적 헌신, 성 차별 극복 | 300 |
| 2 | Albert Einstein | 창의적 사고, 상대성 이론 | 320 |
| 3 | Thomas Edison | 실패와 도전, 발명 정신 | 280 |
| 4 | Ada Lovelace | 최초의 프로그래머, 여성 선구자 | 300 |
| 5 | Nikola Tesla | 교류전기, 비전의 힘 | 310 |
| 6 | Jane Goodall | 동물 보호, 과학적 관찰 | 320 |

**B. 인권 & 사회 운동가 (6편)**
| # | 인물 | 핵심 주제 | 단어 수 |
|---|------|----------|---------|
| 7 | Martin Luther King Jr. | 비폭력 저항, 시민권 | 350 |
| 8 | Nelson Mandela | 인종 차별 극복, 용서 | 360 |
| 9 | Malala Yousafzai | 교육권, 용기 | 330 |
| 10 | Mahatma Gandhi | 비폭력, 독립 운동 | 340 |
| 11 | Rosa Parks | 시민 불복종, 일상의 용기 | 300 |
| 12 | Frederick Douglass | 자유, 자기교육의 힘 | 350 |

**C. 예술가 & 음악가 (5편)**
| # | 인물 | 핵심 주제 | 단어 수 |
|---|------|----------|---------|
| 13 | Leonardo da Vinci | 다재다능, 호기심 | 350 |
| 14 | Frida Kahlo | 고통과 예술, 정체성 | 330 |
| 15 | Ludwig van Beethoven | 역경 극복, 청각 장애 | 320 |
| 16 | BTS / K-pop Pioneers | 글로벌 문화 영향, 팀워크 | 340 |
| 17 | Walt Disney | 상상력, 실패 후 성공 | 300 |

**D. 탐험가 & 개척자 (5편)**
| # | 인물 | 핵심 주제 | 단어 수 |
|---|------|----------|---------|
| 18 | Amelia Earhart | 도전 정신, 성별 장벽 | 320 |
| 19 | Neil Armstrong | 달 착륙, 인류의 도약 | 330 |
| 20 | Ernest Shackleton | 리더십, 극한 생존 | 350 |
| 21 | Jang Bogo (장보고) | 해상 무역, 한국 역사 | 300 |
| 22 | Sacagawea | 문화 중재, 탐험 기여 | 310 |

**E. 현대 혁신가 (4편)**
| # | 인물 | 핵심 주제 | 단어 수 |
|---|------|----------|---------|
| 23 | Steve Jobs | 디자인 철학, 혁신 | 360 |
| 24 | Elon Musk | 미래 비전, 도전 | 370 |
| 25 | 반기문 (Ban Ki-moon) | 외교, 글로벌 리더십 | 340 |
| 26 | Wangari Maathai | 환경 운동, 풀뿌리 변화 | 330 |

**F. 한국 위인 (4편)**
| # | 인물 | 핵심 주제 | 단어 수 |
|---|------|----------|---------|
| 27 | 세종대왕 (King Sejong) | 한글 창제, 백성 사랑 | 350 |
| 28 | 이순신 (Admiral Yi) | 전략, 헌신 | 360 |
| 29 | 유관순 (Yu Gwan-sun) | 독립 운동, 용기 | 320 |
| 30 | 김연아 (Yuna Kim) | 세계 무대, 집중과 노력 | 300 |

#### 문법 요소 (B1~B1+)
- 현재완료 진행: has been working
- 과거완료: had already finished before...
- 가정법 2형 도입: If she had not studied...
- 관계부사: where, when, why
- 분사구문 (기초): Born in 1867, ...
- 접속부사: however, therefore, moreover

---

### Stage 5: Future World & Careers (20권)

**대상:** 고등 1-2학년 / CEFR B1+~B2
**목표:** 정보텍스트(expository) 독해, 주장-근거 구분, 비판적 사고 도입

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 총 단어 수/편 | 300-500 words |
| 문장 구조 | 복합-복문, 삽입절, 분사구문 |
| 어휘 | 3,000-3,500단어 + 전문용어 15-20개/편 |
| 텍스트 유형 | 정보문, 설명문, 인터뷰 |
| 시각 자료 | 인포그래픽, 차트 포함 |

#### 20편 구성안 (4개 하위 카테고리)

**A. AI & 테크놀로지 직업 (5편)**
| # | 토픽 | 단어 수 |
|---|------|---------|
| 1 | AI Engineer: Building Tomorrow's Intelligence | 350 |
| 2 | Data Scientist: Reading the World Through Numbers | 380 |
| 3 | Cybersecurity Expert: Digital Guardians | 400 |
| 4 | UX Designer: Making Technology Human | 370 |
| 5 | Robotics Engineer: Machines That Move | 390 |

**B. 지속가능 미래 직업 (5편)**
| # | 토픽 | 단어 수 |
|---|------|---------|
| 6 | Climate Scientist: Predicting Our Planet's Future | 400 |
| 7 | Renewable Energy Engineer: Powering Change | 420 |
| 8 | Urban Farm Designer: Growing Food in Cities | 380 |
| 9 | Ocean Conservation Biologist: Saving the Seas | 410 |
| 10 | Circular Economy Consultant: Zero Waste World | 430 |

**C. 의료 & 바이오 직업 (5편)**
| # | 토픽 | 단어 수 |
|---|------|---------|
| 11 | Genetic Counselor: Decoding Our DNA | 420 |
| 12 | Telemedicine Doctor: Healthcare Without Borders | 400 |
| 13 | Biomedical Engineer: Where Biology Meets Design | 440 |
| 14 | Mental Health App Developer: Digital Wellbeing | 410 |
| 15 | Pandemic Preparedness Specialist | 450 |

**D. 크리에이티브 & 글로벌 직업 (5편)**
| # | 토픽 | 단어 수 |
|---|------|---------|
| 16 | Content Creator: Storytelling in the Digital Age | 380 |
| 17 | Space Tourism Guide: The Final Frontier | 420 |
| 18 | Global Education Consultant: Learning Without Borders | 440 |
| 19 | Social Entrepreneur: Profit with Purpose | 460 |
| 20 | Metaverse Architect: Building Virtual Worlds | 480 |

#### 문법 요소 (B1+→B2)
- 수동태 확장: is being developed, has been established
- 관계대명사 계속적 용법: ..., which means...
- 가정법: If AI were to..., If we invested more in...
- 화법 전환: Experts say that...
- 분사구문 확장: Having graduated from..., Faced with...
- 도치 구문 도입: Not only... but also...

---

### Stage 6: CSAT & Critical Reading (수능 고난이도 지문)

**대상:** 고등 2-3학년 / CEFR B2~B2+
**목표:** 수능 유형별 독해 전략, 추론적 이해, 요약 능력, 구조 분석

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 총 단어 수/편 | 400-700 words |
| 문장 구조 | 복합 복문, 도치, 강조구문, 삽입, 동격 |
| 어휘 | 4,000-5,000단어 + 추상 어휘 |
| 텍스트 유형 | 논설문, 설명문, 수필 |
| Readability | Flesch-Kincaid Grade 10-12 |

#### 수능 유형 매핑

| 유형 | 출제 비중 | 핵심 전략 | 편수 |
|------|----------|----------|------|
| 빈칸추론 | 최고난도 | 논리적 추론, 문맥 파악 | 8 |
| 순서배열 | 고난도 | 담화 구조, 연결사 | 6 |
| 문장삽입 | 고난도 | 응집성, 지시어 추적 | 6 |
| 주제/요지 | 중난도 | 중심생각, 주제문 | 5 |
| 어휘추론 | 중난도 | 문맥적 의미 파악 | 5 |
| 요약문 완성 | 고난도 | 핵심 논지 압축 | 5 |
| 장문독해 | 복합 | 복합 스킬 | 5 |

#### 40개 토픽 프레임워크 (사용자 제공 토픽 + 확장)

현대사회 핵심 이슈를 8개 대영역, 각 5개 소주제로 구성:

**영역 1: AI & 기술 (5 topics)**
1. AI and Human Creativity: Competition or Collaboration?
2. Algorithmic Bias: When Machines Discriminate
3. Digital Privacy in the Age of Surveillance
4. The Ethics of Autonomous Weapons
5. Brain-Computer Interfaces: Merging Mind and Machine

**영역 2: 환경 & 지속가능성 (5 topics)**
6. Climate Tipping Points: The Point of No Return
7. Biodiversity Loss: The Sixth Mass Extinction
8. Greenwashing: When Green Claims Mislead
9. Environmental Justice: Who Bears the Burden?
10. Degrowth vs. Green Growth: Rethinking Progress

**영역 3: 정의 & 윤리 (5 topics)**
11. Restorative Justice: Healing Over Punishment
12. The Ethics of Gene Editing (CRISPR)
13. Animal Rights and Moral Consideration
14. Intergenerational Justice: Obligations to the Future
15. Cancel Culture and Free Speech

**영역 4: 심리 & 인지과학 (5 topics)**
16. Cognitive Biases: Why We Think Irrationally
17. The Psychology of Misinformation
18. Emotional Intelligence in the Workplace
19. The Paradox of Choice: Too Many Options
20. Grit and Growth Mindset: Beyond Talent

**영역 5: 경제 & 사회 (5 topics)**
21. Universal Basic Income: Utopia or Illusion?
22. The Gig Economy: Freedom or Exploitation?
23. Behavioral Economics: Nudging Better Decisions
24. Inequality and Social Mobility
25. The Attention Economy: Your Focus as Currency

**영역 6: 교육 & 언어 (5 topics)**
26. Multilingualism and Cognitive Advantages
27. The Hidden Curriculum: What Schools Really Teach
28. Digital Literacy: Navigating Information Overload
29. Standardized Testing: Measuring What Matters?
30. Lifelong Learning in a Rapidly Changing World

**영역 7: 문화 & 커뮤니케이션 (5 topics)**
31. Cultural Appropriation vs. Cultural Exchange
32. The Power of Narrative: How Stories Shape Reality
33. Echo Chambers and Filter Bubbles
34. Visual Literacy in the Age of Deepfakes
35. The Globalization of Language: English as Lingua Franca

**영역 8: 과학 & 철학 (5 topics)**
36. The Nature of Consciousness
37. Scientific Reductionism vs. Holism
38. The Fermi Paradox: Where Is Everybody?
39. Epistemology: How Do We Know What We Know?
40. The Ship of Theseus: Identity and Change

#### 문법 요소 (B2~B2+)
- 도치: Not until ~ did..., Never before has...
- 강조구문: It is ~ that...
- 동격: The idea that..., The fact that...
- 삽입구: — which experts call "X" —
- 가정법 복합: Had it not been for...
- 무생물 주어 구문: This discovery enabled...
- 양보절: Despite/Although/While...

---

### Stage 7: Academic Discourse (학술 지문)

**대상:** 고등 3학년 ~ 대학 / CEFR C1
**목표:** 미국 Liberal Arts 수준 학술 지문 이해, 비판적 분석, 학문적 글쓰기 기초

#### 텍스트 사양
| 항목 | 기준 |
|------|------|
| 총 단어 수/편 | 600-1200 words |
| 문장 구조 | 학술적 문체, 다층 복문, hedging 표현 |
| 어휘 | 5,000+ 단어 + Academic Word List |
| 텍스트 유형 | 학술 에세이, 강의 발췌, 연구 요약 |
| 출처 표기 | APA/MLA 인용 포함 |

#### Stage 6 토픽의 학술적 심화

Stage 6의 40개 토픽 중 특히 학문적 깊이가 있는 주제를 선별하여 학술 지문으로 확장:

| 학문 영역 | Stage 6 연계 토픽 | 학술적 접근 |
|----------|------------------|------------|
| Philosophy | #11 Restorative Justice | Rawls의 정의론 vs 공동체주의 |
| Philosophy | #36 Consciousness | Chalmers의 Hard Problem |
| Philosophy | #39 Epistemology | 과학적 실재론 논쟁 |
| Psychology | #16 Cognitive Biases | Kahneman의 이중 처리 이론 |
| Psychology | #20 Grit & Mindset | Dweck/Duckworth 연구 비판적 검토 |
| Economics | #21 UBI | 핀란드 실험 + 경제 모델링 |
| Economics | #23 Behavioral Economics | Thaler/Sunstein의 넛지 이론 |
| Sociology | #22 Gig Economy | 노동의 미래 + 불안정 노동 |
| Sociology | #24 Inequality | Piketty의 자본론 핵심 논지 |
| Political Science | #15 Cancel Culture | Mill의 자유론 + 현대 적용 |
| Environmental Science | #6 Tipping Points | 기후 시스템 피드백 루프 |
| Environmental Science | #9 Env. Justice | 환경 인종주의 사례 연구 |
| Linguistics | #26 Multilingualism | 이중언어 인지 이점 연구 |
| Linguistics | #35 English as Lingua Franca | 언어 제국주의 vs 실용주의 |
| Computer Science | #1 AI and Creativity | 생성AI의 저작권 쟁점 |
| Computer Science | #2 Algorithmic Bias | 공정성 측정 지표 |
| Bioethics | #12 Gene Editing | 생식세포 편집의 윤리적 프레임 |
| Media Studies | #34 Deepfakes | 정보 생태계와 신뢰 |
| Education | #29 Standardized Testing | Washback effect 연구 |
| Cognitive Science | #19 Paradox of Choice | Schwartz vs Iyengar 실험 |

#### 문법 요소 (C1)
- Hedging: It appears that..., One might argue..., tend to, seem to
- 명사화: The implementation of..., The assumption that...
- 양보와 반론: While it is true that..., Granted, ...; however, ...
- 복합 관계절: ..., a phenomenon which researchers attribute to...
- 학술적 수동태: It has been widely demonstrated that...
- 인용 동사: contend, posit, assert, refute, corroborate

---

## 3. Text DB 스키마 설계

### 3.1 핵심 테이블 구조

```typescript
// ============================================
// Reading Text Database Schema
// ============================================

interface ReadingText {
  id: string                    // UUID
  stage: 1 | 2 | 3 | 4 | 5 | 6 | 7

  // === 기본 메타데이터 ===
  title: string
  subtitle?: string
  category: string              // "phonics" | "fable" | "classic" | "biography" | "career" | "csat" | "academic"
  subcategory: string           // e.g., "science_inventors", "adventure", "ai_tech"
  topicArea?: string            // Stage 6-7: 8개 대영역 중 하나
  topicNumber?: number          // Stage 6-7: 1-40

  // === 텍스트 본문 ===
  content: string               // 전체 텍스트
  contentHtml?: string          // 서식 포함 HTML 버전
  paragraphs: Paragraph[]       // 문단 분리 데이터

  // === 난이도 지표 ===
  cefrLevel: 'Pre-A1' | 'A1' | 'A2' | 'B1' | 'B1+' | 'B2' | 'B2+' | 'C1'
  lexileScore: number           // Lexile 지수
  fleschKincaidGrade: number    // FK 학년 지수

  // === 텍스트 분석 ===
  wordCount: number
  sentenceCount: number
  avgSentenceLength: number     // words per sentence
  avgWordLength: number         // syllables per word
  uniqueWordCount: number
  lexicalDensity: number        // unique words / total words

  // === 어휘 데이터 ===
  targetVocabulary: VocabItem[] // 학습 대상 어휘
  vocabularyLevel: {
    k1Percentage: number        // 고빈도 1,000단어 비율
    k2Percentage: number        // 1,001-2,000단어 비율
    k3Percentage: number        // 2,001-3,000단어 비율
    awlPercentage: number       // Academic Word List 비율
    offListPercentage: number   // 목록 외 어휘 비율
  }

  // === 문법 태그 ===
  grammarFeatures: GrammarTag[] // 포함된 문법 요소
  keyStructures: string[]       // 핵심 문장 구조 (학습 포인트)

  // === 학습 활동 ===
  comprehensionQuestions: Question[]
  vocabularyExercises: Exercise[]

  // === 수능 연계 (Stage 6) ===
  csatType?: CsatQuestionType
  csatDifficulty?: 'basic' | 'intermediate' | 'advanced'

  // === 메타 정보 ===
  author?: string               // 원작자
  originalWork?: string         // 원작 제목
  illustrationRatio?: number    // 삽화 비율 (0-1)
  audioAvailable: boolean       // 음성 파일 존재 여부
  koreanTranslation?: string    // 한국어 번역 (참고용)

  // === 5D 어휘학습 연동 ===
  fiveDimensions: {
    semantic: string[]          // 의미 관련 핵심 어휘
    contextual: string[]        // 문맥 사용 핵심 어휘
    form: string[]              // 형태 변화 핵심 어휘
    relational: string[]        // 관계어 핵심 어휘
    pragmatic: string[]         // 화용적 핵심 어휘
  }

  createdAt: string
  updatedAt: string
}

interface Paragraph {
  index: number
  text: string
  wordCount: number
  keyIdea?: string              // 문단 핵심 아이디어 (Stage 4+)
}

interface VocabItem {
  word: string
  partOfSpeech: string
  cefrLevel: string
  definition: string
  definitionKo: string          // 한국어 뜻
  exampleSentence: string       // 지문 내 예문
  phonetic: string              // IPA 발음기호
  audioUrl?: string
  // 5D 연동
  dimension: 'semantic' | 'contextual' | 'form' | 'relational' | 'pragmatic'
}

interface GrammarTag {
  feature: string               // e.g., "past_simple", "passive_voice"
  cefrLevel: string
  exampleFromText: string       // 지문 내 해당 문장
  explanation?: string
  explanationKo?: string
}

type CsatQuestionType =
  | 'blank_inference'           // 빈칸추론
  | 'sentence_order'            // 순서배열
  | 'sentence_insertion'        // 문장삽입
  | 'main_idea'                // 주제/요지
  | 'vocabulary_inference'      // 어휘추론
  | 'summary_completion'        // 요약문 완성
  | 'long_passage'              // 장문독해

interface Question {
  id: string
  type: 'multiple_choice' | 'true_false' | 'short_answer' | 'open_ended'
  question: string
  questionKo?: string
  options?: string[]
  correctAnswer: string
  explanation: string
  explanationKo?: string
  skill: 'literal' | 'inferential' | 'evaluative' | 'creative'
  // 수능 유형 연계
  csatType?: CsatQuestionType
}

interface Exercise {
  type: 'fill_blank' | 'matching' | 'word_form' | 'synonym_antonym' | 'context_clue'
  items: ExerciseItem[]
}

interface ExerciseItem {
  prompt: string
  answer: string
  distractors?: string[]
}
```

### 3.2 보조 테이블

```typescript
// 학습 진도 추적
interface ReadingProgress {
  userId: string
  textId: string
  status: 'not_started' | 'in_progress' | 'completed'
  startedAt?: string
  completedAt?: string
  comprehensionScore?: number   // 0-100
  vocabularyScore?: number      // 0-100
  timeSpent: number             // seconds
  readCount: number             // 재독 횟수
  notes?: string
}

// 스테이지 진급 기준
interface StagePromotion {
  stage: number
  requiredTextsCompleted: number        // 최소 완료 텍스트 수
  requiredAvgComprehension: number      // 최소 평균 이해도
  requiredVocabularyMastery: number     // 최소 어휘 숙달도
  cefrTestScore?: number               // CEFR 테스트 점수 (선택)
}

// 사용자 제공 토픽 관리 (Stage 6-7)
interface UserTopic {
  id: string
  areaCode: string              // "ai_tech", "environment", etc.
  areaName: string
  topicNumber: number           // 1-40
  title: string
  titleKo: string
  description: string
  keywords: string[]
  relatedAcademicField?: string // Stage 7 연계 학문 분야
  userId: string                // 토픽 제공 사용자
  createdAt: string
}
```

---

## 4. 단계간 연결 & 스캐폴딩 전략

### 4.1 Stage 전이 브릿지

각 Stage 전환 시 **3-5편의 브릿지 텍스트**를 제공:

```
Stage 1 → 2 브릿지: 파닉스 패턴이 포함된 짧은 이솝이야기
Stage 2 → 3 브릿지: 교훈이 있는 짧은 명작 요약
Stage 3 → 4 브릿지: 클래식 작품 속 실제 인물 이야기
Stage 4 → 5 브릿지: 위인의 직업 세계를 현대적으로 재해석
Stage 5 → 6 브릿지: 직업 관련 사회적 이슈를 논설 형식으로
Stage 6 → 7 브릿지: 수능 지문의 핵심 논지를 학술적으로 확장
```

### 4.2 어휘 연속성 매트릭스

| 방향 | 설명 |
|------|------|
| 수직 연결 | Stage N에서 학습한 어휘가 Stage N+1에서 재등장 (spaced repetition) |
| 수평 연결 | 같은 Stage 내 다른 텍스트에서 동일 어휘 다른 맥락으로 재사용 |
| 5D 연결 | 각 어휘의 5차원(Semantic, Contextual, Form, Relational, Pragmatic) 데이터 축적 |

### 4.3 스캐폴딩 장치

| Stage | 지원 장치 |
|-------|----------|
| 1-2 | 삽화(70-80%), 반복 패턴, L1 어휘 주석 |
| 2-3 | 삽화(50%), 어휘 사전학습, 줄거리 예측 |
| 3-4 | 삽화(30%), 문단별 핵심 아이디어, 타임라인 |
| 4-5 | 인포그래픽, 배경지식 박스, 어휘 맵 |
| 5-6 | 텍스트 구조 다이어그램, 논리 연결사 하이라이트 |
| 6-7 | 학술 어휘 주석, 논증 구조 분석, 인용 해석 가이드 |

---

## 5. 전체 콘텐츠 수량 요약

| Stage | 카테고리 | 편수 | 총 단어 수 (예상) |
|-------|---------|------|------------------|
| 1 | Phonics Readers | 20 | ~1,100 |
| 2 | Aesop's Fables | 20 | ~2,200 |
| 3 | Classic Stories | 30 | ~7,500 |
| 4 | Great Lives (위인전) | 30 | ~9,900 |
| 5 | Future World & Careers | 20 | ~8,200 |
| 6 | CSAT Critical Reading | 40 | ~22,000 |
| 7 | Academic Discourse | 20 | ~18,000 |
| **Bridge** | 전이 브릿지 | ~20 | ~4,000 |
| **합계** | | **~200편** | **~72,900 words** |

---

## 6. 구현 로드맵

### Phase 1: 기반 구축 (2주)
- [ ] DB 스키마 최종 확정 및 Supabase 테이블 생성
- [ ] 텍스트 분석 도구 구축 (단어 수, 문장 길이, Lexile 자동 계산)
- [ ] 어휘 분석 파이프라인 (K1/K2/K3/AWL 분류)
- [ ] 5D 어휘학습 앱과의 연동 API 설계

### Phase 2: Stage 1-2 콘텐츠 (3주)
- [ ] Phonics Readers 20편 작성 + 음성 녹음
- [ ] Aesop's Fables 20편 작성 (축약 리텔링)
- [ ] 각 편의 어휘/문법 태깅
- [ ] 이해도 문제 및 어휘 연습 생성

### Phase 3: Stage 3-4 콘텐츠 (4주)
- [ ] Classic Stories 30편 축약 리텔링 작성
- [ ] Great Lives 30편 위인전 작성
- [ ] 문단별 핵심 아이디어 태깅
- [ ] 브릿지 텍스트 작성

### Phase 4: Stage 5-6 콘텐츠 (4주)
- [ ] Future Careers 20편 작성
- [ ] CSAT 지문 40편 작성 (토픽별)
- [ ] 수능 유형별 문제 생성 (유형당 5-8문항)
- [ ] 사용자 토픽 입력 시스템 구축

### Phase 5: Stage 7 & 통합 (3주)
- [ ] Academic Discourse 20편 작성
- [ ] 전체 Stage 간 어휘 연속성 검증
- [ ] 스캐폴딩 장치 구현
- [ ] 진급 기준 & 학습 진도 추적 시스템

### Phase 6: AI 연동 (2주)
- [ ] LLM 기반 적응형 난이도 조절
- [ ] 5D 어휘 코치 → 리딩 추천 파이프라인
- [ ] 자동 문항 생성 (문항생성 Agent 연동)
- [ ] 학습 분석 리포트 (분석 Agent 연동)

---

## 7. 기술 스택 & 도구

| 영역 | 도구 |
|------|------|
| 텍스트 분석 | `text-readability` (Lexile/FK), `compromise` (NLP) |
| 어휘 분류 | BNC/COCA 빈도 목록, Academic Word List |
| 문법 태깅 | SpaCy / Stanford NLP (dependency parsing) |
| 콘텐츠 생성 | GPT-4o (초안) + 전문가 검수 |
| TTS | OpenAI TTS / ElevenLabs (음성 파일) |
| DB | Supabase (PostgreSQL) + Neo4j (어휘 그래프 연동) |
| 프론트엔드 | Next.js + shadcn/ui + Recharts |

---

## 8. 품질 관리 기준

### 텍스트 품질 체크리스트

- [ ] **난이도 적합성:** CEFR 레벨에 맞는 어휘/문법 사용
- [ ] **길이 기준 준수:** Stage별 목표 단어 수 ±15% 이내
- [ ] **문화적 적합성:** 한국 EFL 학습자에게 적절한 내용
- [ ] **성별/인종 균형:** 위인전, 직업 소개 등에서 다양성 확보
- [ ] **정확성:** 역사적 사실, 과학적 정보의 정확성 검증
- [ ] **저작권:** 원작 축약 시 공공 도메인 확인 또는 원작 리텔링
- [ ] **5D 연동:** 각 텍스트에서 5D 차원별 학습 포인트 명시
- [ ] **한국어 지원:** 핵심 어휘 한국어 뜻, 문화적 배경 주석

---

*문서 작성: 2026-02-28*
*AI TESOL 멀티에이전트 프로젝트 연계*
