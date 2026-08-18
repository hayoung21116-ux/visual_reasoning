> ## ⚠️ 이 파일은 구버전입니다
>
> **스코프가 좁혀지기 전(2026-08)의 1단계 정리본입니다.** B-0(Thinking with Images 서베이)과 B-1(CodeVision)의 **원문 대조 내용은 그대로 유효**하므로 인용 근거로 계속 쓸 수 있습니다.
>
> 다만 여기 적힌 **"우리 과제 관점" 판단은 넓은 스코프 기준**이라 현재와 어긋나는 부분이 있습니다 (예: closed-source 논거, abstain 비중).
>
> **현행 파일:** `논문 search (0단계) - rotated OCR.md` · `방법론 분류 (rotated OCR × 소형모델).md`

---

## 1단계 정리 — 핵심 실험 세팅·결과

> 0단계에서 추린 논문 중 **더 깊이 봐야 할 것만** 골라 1단계(실험 세팅·결과)로 내려 정리합니다.
> 0단계 파일: `[구버전] 논문 search (0단계).md`

**표기 규칙**
- 항목마다 **원문 대조 여부를 맨 끝에 명시**합니다. 인용 전에 반드시 확인할 것.
  - **B-0**: 원문 PDF(v3, 57쪽) 대조 완료 — §3·§5·§7의 개별 방법 소개는 미독
  - **B-1**: 원문 PDF 대조 완료 (Figure까지 렌더링해 판독)
- 그래프에서 눈으로 읽은 값은 `~`로 표기합니다 (본문에 수치가 없는 것들).
- 기술 용어는 영어 원어 그대로 씁니다.

**수록 현황**
- [x] B-0. Thinking with Images 서베이 ← **지도. 먼저 읽을 것**
- [x] B-1. Thinking with Programming Vision (CodeVision)
- [ ] A-2. EquiAdapt
- [ ] B-3. DeepEyes
- [ ] B-4. AdaTooler-V
- [ ] (필요할 때 추가)

---

# B-0. Thinking with Images for Multimodal Reasoning: Foundations, Methods, and Future Frontiers

> **B-1보다 먼저 읽으세요.** 서베이라서 실험이 없습니다. 이 항목의 1단계는 "실험 세팅·결과" 대신 **taxonomy·formal definition·challenges를 정확히 파악하는 것**입니다.

## 서지 정보

| 항목 | 내용 |
|---|---|
| 제목 | *Thinking with Images for Multimodal Reasoning: Foundations, Methods, and Future Frontiers* |
| 저자 | Zhaochen Su¹, Peng Xia², Hangyu Guo¹, Zhenhua Liu¹, Yan Ma, Xiaoye Qu, Jiaqi Liu², Yanshu Li¹, Kaide Zeng², Zhengyuan Yang³, Linjie Li³, Yu Cheng⁴, Heng Ji⁵, Junxian He¹, Yi R. (May) Fung¹ |
| 소속 | ¹HKUST, ²UNC-Chapel Hill, ³Microsoft, ⁴CUHK, ⁵UIUC |
| arXiv | 2506.23918**v3** (2025-07-03, cs.CV) — 57쪽 |
| 저장소 | github.com/zhaochen0110/Awesome_Think_With_Images (실시간 갱신) |
| 유형 | **Survey** — 실험·수치 없음 |

**네 가지 기여 (초록 명시):** ① 패러다임의 foundational principles + 3단계 framework 확립 ② 각 단계 핵심 방법 리뷰 ③ 평가 benchmark·응용 지형 분석 ④ challenges와 future directions 제시.

---

## 1. 문제 제기: semantic gap

- 기존 multimodal reasoning은 **textual CoT**에 의존 → 이미 **ceiling**에 도달했다는 게 저자들 진단
- 이미지가 **하나의 정적 feature vector로 encoding**되면 시각 세계의 relational structure가 **flatten**됨
- 그 결과 **superhuman perception을 가진 모델조차 cognitively brittle**해진다

> 이 문장이 우리 논문에 그대로 쓸 만합니다 — "지각 능력이 뛰어나도 인지적으로는 취약하다"가 회전 실패를 설명하는 프레임이 됩니다.

**해결이 여는 3가지 능력** (Figure 2):

| 능력 | 내용 | 예시 |
|---|---|---|
| **Dynamic Perceptual Exploration** | 한 번의 총체적 해석을 넘어 반복적 탐색 | 칼로리 라벨을 zoom-in해서 60 → 160으로 정정 |
| **Structured Visual Reasoning** | 이미지를 **cognitive scratchpad**로 | 기하 문제에 보조선을 그려 넣으면 abstract deduction이 **visual pattern recognition으로 바뀜** |
| **Goal-Oriented Generative Planning** | 생성 능력을 **시뮬레이션 엔진**으로 | 로봇 팔이 컵을 치는 미래 프레임을 생성 → **perceptual self-critique** |

## 2. 3단계 taxonomy

축은 **cognitive autonomy**. 방을 재배치해 큰 소파를 넣는 문제를 러닝 예시로 씁니다.

| Stage | 역할 | 러닝 예시에서 하는 일 | 제약 |
|---|---|---|---|
| **1** Tool-Driven Visual Exploration | tool orchestrator | `object_detector`, `distance_estimator` 호출 → "틈이 1.5m인데 소파는 2.0m 필요" | **미리 정의된 toolset의 정적 능력**에 갇힘 |
| **2** Programmatic Visual Manipulation | **visual programmer** | matplotlib로 2D 평면도를 생성해 배치를 프로그램으로 시험 | **외부 실행 환경 의존** |
| **3** Intrinsic Visual Imagination | visual thinker | 재배치된 방을 **photorealistic 이미지로 직접 생성**해 자기비판 | (아키텍처 병목 해소가 목표) |

### ★ §2.2 "A Note on Non-Linearity" — 반드시 알아야 할 단서

저자들은 **3단계가 선형 진보가 아니라고 명시적으로 못 박습니다.**

> *"These three stages do not represent a strictly linear progression. They are different implementation strategies (the 'How')..."*

근거로 든 예: Stage 3가 가능한 모델이라도 "소파가 문을 통과하나?"를 알려면 **full simulation보다 Stage 1의 `measure` tool 한 번이 훨씬 효율적이고 robust**하다. 그래서 **지능은 peak capability가 아니라 "과제에 맞는 인지 도구를 고르는 능력"에 있다**고 씁니다.

**단, 서베이 안에 긴장이 있습니다.** 초록은 *"spectrum of increasing cognitive autonomy"*라 하고 Figure 1은 *"Intelligence Increasing"*이라 표기하며 Stage 3를 *"the most advanced stage"*라 부릅니다. **위계를 제시하면서 동시에 선형성을 부인**하는 구조예요. 인용할 때 어느 쪽을 쓰는지에 따라 논지가 달라지므로 주의가 필요합니다.

## 3. Formal definition (§2.3)

**Paradigm I — Thinking *about* Images**
```
x_t ~ P(· | x_<t, v, Q; Θ),   v = Φ_V(I)
```
이미지는 한 번 encoding되어 **고정된 context v**가 되고, 추론 중 **수정되지 않습니다.**

**Paradigm II — Thinking *with* Images**
```
z_t ~ P(· | S_t, I, Q; Θ)   where   z_t ∈ T_text ∪ I_vis
```
- `S_t = (z_1, ..., z_{t-1})` — **텍스트와 시각 단계가 섞인** 상태 이력
- `I_vis` — 추론 중 도입·수정 가능한 중간 시각 산출물의 공간

핵심 차이는 **state history가 동적**이라는 것. `z_t ∈ I_vis`는 ① 외부 tool의 출력(bounding box) ② 코드로 생성한 시각화(보조선) ③ 모델이 내재적으로 생성한 이미지(예측 프레임) 셋 다 될 수 있습니다.

> **우리 설계를 이 표기로 쓸 수 있습니다.** 앞단 모델이 회전을 보정하면 그건 `z_1 ∈ I_vis`이고, 본 모델은 `S_2`부터 Paradigm I로 동작합니다. **"z_1만 외부 모델이 담당하는 hybrid"**로 형식화하면 positioning이 깔끔해집니다.

## 4. ★ Unique Challenges (§2.4) — 4개

우리 과제와 직결되는 절인데 이전 정리에서 통째로 빠져 있었습니다.

| # | 이름 | 내용 |
|---|---|---|
| 1 | **Computational Cost** | 이미지 하나가 수천 개 visual patch로 분해됨. 중간 시각 단계 하나가 major computational event이고 multi-step은 **곱셈적 부담**. "token explosion"이 시각적 숙고의 깊이에 **hard ceiling**을 만듦 |
| 2 | **Information Density** | *"One Flawed Image versus a Thousand Wrong Words"* |
| 3 | **Architectural Divide** | vision과 language를 분리한 modular 설계가 **풍부한 공간 정보를 순차 포맷으로 번역**해야 해서 fine-grained detail을 잃음. 의도 형성 ↔ 결과 지각의 dynamic loop를 저해 |
| 4 | **Cross-Task Generalization** | 하나의 visual strategy가 모든 과제에 맞지 않음. 기하=constructive, 세부 진단=analytical, 미로=simulative. **"다양한 전략을 유지하고 과제에 맞는 걸 고르는 meta-policy" 학습이 미해결 과제** |

### Challenge 2를 특히 주목해야 합니다 ★

저자들의 설명: zoom-in tool이 제품 대신 **아래 나무 탁자에 잘못 초점**을 맞추면, 모델은 그냥 실패하는 게 아니라 **나뭇결과 니스 광택에 대한 그럴듯한 분석을 새로 시작**합니다. 이 최초의 실수가 **false ground truth**를 세우고, 이후 모든 숙고를 오염시킵니다.

> **이게 앞단 모델 방식의 최대 리스크를 서베이가 직접 언어화한 것입니다.** 앞단이 회전을 잘못 판정하면 본 모델은 "틀린 전제 위에서 내적으로 일관된 추론"을 합니다. 0단계 A-5의 `94 → 59.1 → 75.8`에서 완전 회복이 안 되는 이유가 여기 있습니다. **우리 논문에서 error propagation을 다룰 때 인용할 근거가 이 문단입니다.**

## 5. 평가 지형 (§6)

**주의 — 이전 정리를 정정합니다.** 서베이는 특정 benchmark 목록을 나열하지 않고 **7개 도메인으로 분류**합니다.

| # | 도메인 |
|---|---|
| 1 | Mathematical Reasoning (기하 문제가 핵심) |
| 2 | STEM |
| 3 | Perception Reasoning |
| 4 | Code Generation |
| 5 | Chart and Table Reasoning |
| 6 | Real-world Applications |
| 7 | Puzzles and Games |

> **회전·orientation을 다루는 도메인은 없습니다.** B-1 CodeVision의 문제 설정은 이 7개 어디에도 깔끔하게 안 들어갑니다 — 굳이 넣으면 Perception Reasoning인데, 서베이가 상정한 건 "시각적 착시 판별" 류입니다.

## 6. ★ Future Directions (§8) — 우리 주장과 겹치는지 확인

**이전 정리에서 "반드시 확인" 항목으로 남겨뒀던 절입니다. 확인 결과 겹치는 게 있습니다.**

### §8.1 Computational and Cognitive Efficiency

- "cognitive efficiency" = **과제에 맞는 만큼만 노력을 쓰는 것**. 사람이 암산할지 종이를 꺼낼지 정하는 것에 비유
- **"불필요한 visual step에 페널티를 주는 reward system으로 학습시킬 수 있다"** ← B-1의 Inappropriate Tool Use Penalty가 정확히 이것
- "느린 외부 tool 대신 **작은 내장 모듈**을 두고 latent space에서 동작하게" ← 방향은 다르지만 **모듈 분리 발상 자체는 이미 제시됨** (단, *내장* 모듈이지 앞단 분리가 아님)

### §8.3 Novel Benchmarks

- 현재 benchmark는 최종 답만 검사 → **"올바른 이유 없이 정답"(shortcut)** 이 가능하다는 지적
- **necessity test 방법론 제안:** *중간 시각 단계를 제거하고도 정답에 도달하는지 보라*

> 이건 방법론으로 바로 차용할 수 있습니다. "앞단 보정을 껐을 때 성능이 떨어지지 않으면 그 보정은 불필요했다"로 옮기면 됩니다.

### §8.5.1 Thinking with Images v.s. Agentic Frameworks ★★

positioning에 직접 쓸 수 있는 절입니다. 저자들이 둘을 네 축으로 구분합니다.

| 축 | Thinking with Images | Agent Framework |
|---|---|---|
| 목적 | **Deliberation** — 추론의 fidelity·depth | **Execution** — 외부 과제 완수 |
| vision의 역할 | **cognitive workspace** (mental sketchpad) | **환경 지각**의 매체 |
| 중간 단계 | **cognitive artifacts** (내적 구성물) | **executable instructions** (API 호출 등) |
| 관계 | \= **cognitive engine** | \= 그 engine이 들어가는 **machine** |

결론: *"Thinking with Images를 agent라는 기계 안의 인지 엔진으로 보라."* 배타적이 아니라 상보적이라는 것.

### §8.5.2 Blueprint for a Unified Visual Thinker ★★★ — 우리 abstain 아이디어와 겹칩니다

Figure 5의 청사진입니다. **핵심 부품이 Metacognitive Controller**입니다.

```
입력 → [Metacognitive Controller]  ── "Thinking with Images가 필요한가?"
                 │
      필요없음 ──┴──> Direct Encode → Textual Inference → 답
      필요함   ────> Visual Cognitive Workspace (Stage 1/2/3)
                        ↑ 반복 정제·피드백 루프 ↓
                     Action & Output
```

저자들의 원칙: *"항상 가장 복잡한 추론에 기대는 모델이 아니라, **과제에 맞는 인지 도구를 지능적으로 고르는 동적 시스템**"*. 마지막 문장은 **"진짜 visual thinker의 열쇠는 상상하는 능력이 아니라 가장 효과적·효율적인 경로를 고르는 meta-reasoning"**입니다.

> **이것이 0단계에서 "빈 자리"로 꼽았던 회전 abstain 정책과 개념적으로 겹칩니다.** 서베이가 이미 "쓸지 말지 판단하는 controller"를 청사진으로 제시했고, §8.1은 "불필요한 시각 단계에 페널티"를 제안했으며, B-1은 그걸 penalty로 부분 구현했습니다.
>
> **→ abstain을 novelty로 주장하면 안 됩니다.** 남은 자리는 훨씬 좁습니다: ① **회전에 특화된** 판정(방향이 정의되지 않는 이미지 식별) ② 그 판정 능력 **자체를 평가하는 프로토콜** — 서베이도 B-1도 이건 안 했습니다.
>
> **다만 결정적 차이가 하나 있습니다.** 서베이의 Metacognitive Controller는 **통합 모델 내부의 부품**입니다. 우리 구상은 **본 모델 외부의 독립 모듈**이고 본 모델은 freeze입니다. §8.5.1의 어휘를 빌리면 — 서베이는 engine 내부를 설계하고, 우리는 **engine을 갈아끼울 수 있는 machine**을 설계하는 것입니다.

---

## 7. ★ 회전은 이 서베이에 사실상 없습니다 (검증 완료)

57쪽 전문 검색 결과입니다.

| 단어 | 등장 횟수 |
|---|---|
| `rotate` / `rotation` | **1회** |
| `orientation` | **0회** |
| `canonical` | **0회** |
| `equivariance` | **0회** |

유일한 1회는 §4.3 Categories of Composable Operations에서 **tool 목록의 예시**로 나옵니다:

> *"action-oriented operations such as geometric transformations (**e.g., rotate, resize**) and, critically, drawing functions..."*

**즉 회전은 "할 수 있는 연산 중 하나"로 스쳐 지나갈 뿐, 문제로 취급되지 않습니다.** 서베이가 강조하는 건 오히려 **drawing functions**(보조선 긋기)예요 — "critically"라는 부사가 거기 붙어 있습니다.

**함의 두 가지:**
1. 2025년 7월 시점에 **이 분야는 회전을 문제로 인식하지 못했습니다.** 5개월 뒤 B-1이 그걸 중심 소재로 끌어올린 겁니다. B-1의 소재 발굴이 실제로 새로웠다는 증거.
2. **canonicalization 계열(0단계 A그룹)은 단어 하나도 등장하지 않습니다.** 두 학문 계열의 단절이 정량적으로 확인됐습니다. 우리 논문이 다리를 놓는다는 주장의 근거로 쓸 수 있습니다.

---

## 8. 우리 과제 관점 — 정리

### taxonomy 위 좌표

| 0단계 항목 | 위치 |
|---|---|
| B-5 OpenThinkIMG | Stage 1 |
| B-3 DeepEyes | Stage 1 |
| B-5 Visual Sketchpad | Stage 2 |
| B-2 Thyme | Stage 2 |
| **B-1 CodeVision** | **Stage 2** |
| C-2 Visual Program Distillation | Stage 2 |
| **A그룹 전체 (canonicalization)** | **언급 자체가 없음 (0회)** |

### 결론 4가지

**① B-1의 패러다임은 새롭지 않지만 소재는 새롭습니다.** Stage 2는 이미 정의돼 있었고 Thyme도 거기 있습니다. 그러나 **회전은 서베이가 문제로 보지 않았습니다.** B-1의 기여는 소재 발굴 + reward 설계입니다.

**② abstain 아이디어는 선점됐습니다 (부분적으로).** §8.5.2 Metacognitive Controller + §8.1 penalty 제안. 남은 건 회전 특화 판정과 그 평가 프로토콜.

**③ error propagation은 서베이가 직접 언어화해줬습니다.** §2.4 Challenge 2(Information Density)가 앞단 방식의 최대 리스크입니다. 우리 논문이 이걸 **정면으로 다루면** 오히려 강점이 됩니다 — "우리는 이 위험을 알고 측정했다".

**④ 직교 축 프레임은 여전히 유효하고, 오히려 강화됐습니다.** 서베이의 축은 본 모델의 cognitive autonomy입니다. §8.5.1이 "cognitive engine vs machine"이라는 어휘까지 제공하므로, **"우리는 engine을 교체 가능하게 만드는 machine 층위의 기여"**라고 쓸 수 있습니다.

---

## 9. 원문 대조 상태

**arXiv 2506.23918v3 (57쪽) 전문 대조 완료.** 초록·§1~§9·Figure 1·2·5 캡션·전문 단어 검색까지 확인했습니다.

이전 버전(동반 GitHub 저장소 기반)에서 **정정된 것 2가지:**
- ❌ ~~benchmark로 m&m's, CoMT, ChartMuseum, TIR-Bench를 든다~~ → 실제로는 **7개 도메인 분류**. 이 이름들은 저장소에만 있고 논문엔 ChartMuseum만 1회
- ❌ ~~Stage 1→2→3이 지향해야 할 방향으로 제시된다~~ → §2.2에서 **선형 진보가 아니라고 명시**. 단 초록·Figure 1은 위계를 시사해 내부 긴장 있음

아직 안 읽은 부분: §3(Stage 1 상세), §5(Stage 3 상세), §7(Applications) 각 절의 개별 방법 소개. 필요해지면 요청 주세요.

---

# B-1. Thinking with Programming Vision: Towards a Unified View for Thinking with Images

> 0단계에서 "CodeVision"으로 적어뒀던 논문입니다. **CodeVision은 논문 제목이 아니라 method name**입니다.

## 서지 정보

| 항목 | 내용 |
|---|---|
| 제목 | *Thinking with Programming Vision: Towards a Unified View for Thinking with Images* |
| Method | **CodeVision** |
| 저자 | Zirun Guo¹², Minjie Hong¹, Feng Zhang², Kai Jia², Tao Jin¹ |
| 소속 | ¹ Zhejiang University, ² ByteDance (BandAI) |
| arXiv | 2512.03746v1 (2025-12-03, cs.CV) |
| 학회 | CVPR 2026 |
| 코드 | github.com/ByteDance-BandAI/CodeVision |
| 연락 | zrguo.cs@gmail.com |

---

## 0. 한 눈에

- **진단:** SOTA MLLM이 rotation/flip에 취약. 어떤 변형이 걸렸는지 맞히는 5지선다에서 **최고가 GPT-5의 50.5%**(random 20%, 사람 100%), 다운스트림 성능은 단순 변형만으로 **최대 80%까지** 하락.
- **처방:** tool registry를 고정하지 말고 **코드 자체를 universal tool interface**로 삼는다.
- **학습:** SFT(약 5K, GPT-5로 생성) → RL(약 40K, **GRPO** + dense multi-component reward).
- **평가:** OCRBench·ChartQAPro에 5가지 변형을 걸어 augment + 신규 **MVToolBench**.
- **결과:** CodeVision-7B가 transformed OCRBench 평균 **73.4** (base 56.0, **+17.4**), MVToolBench **60.1** (Gemini2.5-Pro 32.6).

---

## 1. 문제 제기

저자들이 지적하는 기존 thinking-with-images의 세 가지 한계:

| # | 한계 | 내용 |
|---|---|---|
| ① | **tool의 necessity** | 대부분 `crop`에 집중. 그런데 **tool을 써도 정확도 이득이 2–5%에 불과**하고, tool 없는 RL로도 비슷한 성능이 나옴. 즉 기존 task가 tool의 필요성을 제대로 시험하지 못함 |
| ② | **flexibility·scalability** | tool name과 argument를 수동 지정해야 함. **tool 이름만 바꿔도(`crop`→`zoomin`) 재학습이 필요**할 정도로 brittle |
| ③ | **multi-turn·multi-tool** | 대부분 single tool 또는 한 turn 내 소수 tool만 지원. multi-turn을 다루는 연구도 대부분 **crop을 반복**할 뿐, 서로 다른 tool을 조합하지 않음 |

### Diagnostic (Figure 1)

tool이 **정말로 필요한** 상황을 만들기 위해 저자들이 주목한 것이 orientation입니다. 근거로 든 실제 시나리오가 구체적입니다 — landscape/portrait 촬영으로 생기는 잘못된 orientation, 그리고 **텍스트 인식을 방해하는 mirrored selfie**.

- 여러 도메인의 이미지 **200장**
- 5가지 변형 중 하나를 uniform하게 적용
- **어떤 변형이 걸렸는지 맞히는 5-way 객관식** (따라서 random guess = 20%)

**Figure 1 판독 결과:**

| 모델 | 정확도 |
|---|---|
| **Human** | **100%** |
| GPT-5 | **50.5%** |
| GPT-4o | 43.5% |
| Qwen3-VL-235B-Thinking | 36.5% |
| Gemini2.5-Pro | **33.5%** |
| Qwen2.5-VL-72B | 32.0% |
| InternVL3.5-241B | 29.0% |
| *(Random Guess)* | *20%* |

**이 표에서 읽어야 할 것:**
- **최고 성능이 GPT-5의 50.5%** — random guess의 2.5배에 불과합니다. 사람이 100%를 "쉽게" 얻는 과제에서요.
- **Gemini2.5-Pro가 33.5%로 GPT-4o(43.5%)보다 낮습니다.** 그런데 Table 1의 다운스트림 성능에서는 Gemini2.5-Pro가 압도적 1위(OCRBench avg 62.6 vs GPT-4o 52.7)입니다. **즉 "변형을 명시적으로 식별하는 능력"과 "변형된 이미지에서 task를 푸는 능력"이 서로 어긋납니다.**
- InternVL3.5-241B는 29.0%로 거의 random 수준인데, Table 1에서도 가장 취약합니다(avg 45.9, Rot180에서 32.4).

> **우리 과제에 직접 쓸 수 있는 논거:** 이 diagnostic은 **"본 모델은 자기가 어떤 변형을 당했는지 모른다"**를 정량화한 것입니다. B-3(DeepEyes)가 주장하는 "본 모델이 스스로 하게 하면 된다"에 대한 반박 근거가 여기 있습니다 — **판단 능력 자체가 random guess 근처인데 RL로 무엇을 emerge시킬 것인가.** 0단계에서 세운 반박 논거("canonical prior가 본 모델 안에 없다")를 뒷받침하는 숫자입니다.
>
> 다만 Gemini의 역전 현상은 **주의해서 써야 합니다.** "식별 못 함 → task 실패"라는 단선적 인과가 아니라는 반례이기 때문입니다. 오히려 우리 쪽에 유리하게 해석할 수도 있습니다 — 명시적 식별과 무관하게 성능이 나오는 경로가 있다면, 앞단에서 아예 정방향으로 되돌려 주는 것이 더 확실한 처방이라는 논리.

> **우리 과제와의 연결:** RotBench가 "회전을 *맞히지* 못한다"를 보였다면, 이 논문은 거기에 더해 **"회전되면 다운스트림 task 성능이 무너진다"(Table 1)**를 보입니다. Intro에서 "인식 실패 → 성능 실패"로 이어 쓰기 좋습니다.
>
> 저자 주장 중 우리가 그대로 쓸 수 있는 문장: **"cropping에 비해 canonical orientation 복원이 downstream recognition·reasoning에 훨씬 더 필수적(strictly more necessary)"**. 앞단 canonicalization 모델의 존재 이유를 정당화하는 논거입니다.

---

## 2. 방법: code-as-tool

OpenAI o3에서 착안해 **코드 자체를 unified tool**로 취급합니다. 모델이 필요한 image operation을 코드로 작성해 호출합니다.

- hand-crafted tool name/argument 명세가 사라짐
- 고정 registry에 묶이지 않고 **사실상 무한한 tool 집합**에 접근

저자들은 code-as-tool 패러다임 자체는 선행 연구(Program-of-Thoughts, Thyme)에 있었다고 인정하면서, **자신들의 RL 설계와 데이터 구축을 통해** 세 가지 이점을 새로 끌어냈다고 주장합니다.

| 이점 | 내용 |
|---|---|
| **Emergence of new tools** | RL 학습 데이터에 **없던 tool**을 스스로 호출해 새 문제를 품 |
| **Efficiency** | 한 번의 실행 안에서 여러 tool을 chaining |
| **Robustness** | runtime error message와 출력을 읽고 코드를 수정 → failure recovery, OOD generalization |

---

## 3. Cold Start (SFT)

### 3.1. 데이터 구축

**데이터 소스** (도메인 다양화 목적)

| 도메인 | 데이터셋 |
|---|---|
| handwriting | IAM (Marti & Bunke 2002), CASIA (Liu et al. 2011) |
| in-the-wild OCR/VQA | HierText (Long et al. 2022) |
| table·chart | UniChart (Masry et al. 2023) |
| math reasoning | We-Math 2.0 (Qiao et al. 2025) |

**Task type (5종)** — 샘플마다 ground-truth 답과 target type을 담은 metadata를 만들고, 정해둔 비율로 type을 sampling한 뒤 그에 맞춰 tool을 uniform sampling합니다.

| type | 설계 의도 |
|---|---|
| `single-tool` | tool 하나로 해결 |
| `multi-tool` | 서로 다른 tool 조합 |
| `multi-crop` | **단조 축소·공간 연속**인 crop window를 강제해 coarse-to-fine zoom-in을 시뮬레이션 |
| `error-handling` | 잘못된 tool·code error·runtime error를 **의도적으로 발생**시키고, error log를 읽어 코드 수정·재시도 (제한된 retry 후 fallback) |
| `no-tool` | tool이 필요 없는 경우 |

crop type은 원본 데이터셋에 annotate된 텍스트 영역 중 **면적이 전체의 0.01% 이하**인 것만 고릅니다. crop 없이는 못 푸는 상황을 보장하려는 것.

**Metadata-conditioned image transformation** — 데이터 생성이 **역방향**입니다. metadata를 먼저 뽑고, 선택된 tool에 맞춰 canonical(무변형) 이미지를 **변형해서** 모델의 초기 관찰을 만듭니다. 예: 필요한 tool이 `rotate-180`이면 원본을 180° 돌린 걸 입력으로 준다. 이렇게 하면 tool 호출이 **필연적으로 필요**해집니다.

**생성 모델은 GPT-5** — 질문 + (변형된) 이미지 + metadata를 주면 GPT-5가 turn마다 reasoning trace와 action을 냅니다. tool을 호출하면 controlled runtime에서 실행해 이미지를 갱신하고, **결과를 canonical 원본과 비교**해 일치하면 정답 tool call로 표시, 아니면 궤적을 버리거나 교정합니다.

> **주목할 설계 tension:** Figure 1에서 "GPT-5도 어떤 변형인지 못 맞힌다"고 해놓고, SFT 데이터 생성은 GPT-5로 합니다. 저자들도 이걸 인지하고 있고 — *"even state-of-the-art models struggle to reliably determine which tool is needed, so we guide data generation with structured metadata and automatic checks"* — **정답 변형을 자기들이 걸었으니 알고 있다**는 점을 이용해 metadata로 우회합니다. 우리가 데이터를 만들 때도 그대로 쓸 수 있는 트릭입니다.

최종 규모: **약 5,000개** 고품질 SFT 예제 (reasoning + action sequence + answer 정렬).

### 3.2. 학습

multi-turn 대화를 interleaved dialogue로 포맷하고, **user token과 tool-return token을 mask out**합니다. assistant의 chain-of-thought와 tool-call token만 loss에 기여:

```
L_SFT(θ) = − Σ_t m_t · log p_θ(y_t | x, y_<t)
```

`m_t ∈ {0,1}`는 assistant reasoning/tool-call token에서만 1.

**SFT 중에는 tool을 online 실행하지 않습니다.** 데이터 구축 때 캐시해둔 결과를 그대로 context로 씁니다. runtime variance를 없애고 표준 causal LM 학습으로 환원하려는 것.

---

## 4. Reinforcement Learning

### 4.1. 데이터

SFT 소스에 더해 reasoning-heavy·perception 샘플을 추가: **DocVQA** (Mathew et al. 2021), MCTS-guided selection (Wang et al. 2025c), **TextVQA** (Singh et al. 2019).

**Difficulty filtering** — 후보마다 여러 rollout을 뽑아, 전부 맞거나(all-correct) 전부 틀린(all-incorrect) degenerate 샘플을 제거. policy 개선에 신호가 있는 항목만 남깁니다.

**must-use tool 필드** — 각 항목에 반드시 써야 할 tool을 annotate:

```
{ rotate90, rotate180, rotate270, flip-horizontal, flip-vertical, crop }
```

crop이 필요한 항목엔 target region의 bounding box도 붙입니다. tool이 필요 없으면 `None`.

최종 규모: **약 40,000개** RL 학습 항목.

### 4.2. Reward 설계

저자들이 밝히는 실무적 동기가 솔직합니다 — **training collapse와 비정상적인 tool-call rate가 자주 발생**해서 dense reward가 필요했다는 것.

궤적 `τ = (s₁,a₁,...,s_T,a_T)`에 대해:

```
R_total(τ) = R_outcome(τ) + β₁ Σ_t R_strategy(a_t) − β₂ P_cost(τ)
```

#### (a) Outcome Reward
- `r_acc ∈ {0,+1}` — 최종 답의 terminal accuracy
- `r_fmt ∈ {0,+1}` — `<think>`/`<answer>` 태그 형식

#### (b) Strategy Shaping — 두 축

**must-use tools `S_req`**
- `N = |S_req|`개 tool 각각에 **1/N**의 reward budget 배정
- `rotate`/`flip` 같은 categorical tool: 처음 올바르게 쓰면 **1/N 일회성 보너스**
- `crop`: 연속적 품질 척도가 있으므로 **IoU에 비례**. 단 **직전 최고 IoU 대비 개선분만** 보상 (다단계 refinement 유도)
- **완전 일치 보너스**: `S_req`의 tool 전체를 **지정된 순서대로, 불필요·오류 step 없이** 실행했을 때만 부여
  - single-tool에서는 "즉시 정확한 호출"과 "시행착오 끝에 도달"을 구분해 정밀도를 유도
  - multi-tool에서는 optimal solution path 학습을 유도

**suggested tools bonus** — 미리 정의할 수 없는 유용한 tool을 발굴하기 위한 장치. code-as-tool이라 임의 함수·라이브러리를 부를 수 있다는 점을 활용합니다.

- 현재 policy에서 **K = 8** 궤적 수집
- optional tool을 쓴 그룹 `G_tool` / 안 쓴 그룹 `G_notool`으로 분할
- tool 그룹의 정확도가 더 높고 **no-tool 그룹이 K개 중 최대 1번만 성공**하면, 그 tool이 도움된다고 판정

```
r_nec = max(0, mean_{i∈G_tool}(r_acc^i) − mean_{i∈G_notool}(r_acc^i))
```

이 `r_nec`를 해당 optional tool을 쓴 **성공 궤적 전부**에 보너스로 더합니다. 추가로 "optional tool을 최소 하나 쓰고 + 최종 답이 맞으면" 주는 per-trajectory 보너스도 있습니다.

#### (c) Constraint Penalties — reward hacking 방지

저자들이 실제로 관찰한 hacking 행동을 막는 guardrail 3종. 각 penalty `p ∈ {0,+1}`이고 `P_cost`는 이들의 합입니다.

| penalty | 막으려는 행동 |
|---|---|
| **Turn Limit** | `R_strategy`를 최대화하려고 **rotate90/180/270을 순차적으로 다 호출**하는 행동. 첫 action이 이미 문제를 풀었어도 그럼. `\|S_req\|+1` turn(에러 처리용 여유 1턴)을 넘는 tool-call turn에 penalty |
| **Poor Reasoning** | 답은 맞았지만 crop의 **IoU < 0.1** — 시각적 근거 없이 맞힌 경우 |
| **Inappropriate Tool Use** | `S_req = None`(변형 없음)인데 orientation tool을 쓰는 행동. 정상 이미지를 돌리면 난이도만 올라감 |

> **B-4(AdaTooler-V)와의 관계:** 0단계에서 "회전 abstain 정책"을 빈 자리로 꼽았는데, **Inappropriate Tool Use Penalty가 그 아이디어를 이미 부분적으로 가져갔습니다.** 다만 이건 *학습 시 penalty*일 뿐, "돌릴지 말지"를 판단하는 **명시적 정책이나 그 판단 자체에 대한 평가는 없습니다.** 빈 자리가 좁아졌지만 사라지진 않았습니다 (§9 참고).

---

## 5. 평가 세팅

### 5.1. Benchmark 구성

| 목적 | Benchmark |
|---|---|
| crop (single-tool) | V*, HRBench4k, HRBench8k |
| **orientation robustness** | **OCRBench**, **ChartQAPro**에 5가지 변형 적용 |
| **multi-tool** | **MVToolBench** (신규) |

**5가지 transformation:** rotate 90° / 180° / 270°, horizontal flip, vertical flip

OCRBench와 ChartQAPro를 고른 이유가 명확합니다 — OCRBench는 **perception-critical**(글자를 정확히 지각해야 성공), ChartQAPro는 **reasoning-heavy**(모든 문자를 완벽히 인식할 필요는 없음). 두 성격을 나눠 보려는 것.

> **Scope 주의:** flip 2종이 포함돼 있습니다. flip은 rotation group이 아니라 **reflection**이라, canonicalization 이론틀(A-1/A-2)에서 다루는 group이 SO(2)에서 **O(2)로 넓어집니다.** 우리 과제를 rotation만으로 한정할지 flip까지 포함할지가 scope 결정 지점입니다. **Table 1을 보면 이게 단순한 형식 문제가 아닙니다** (§6 참고).

### 5.2. MVToolBench 구축

**HierText** 데이터셋 기반. word/line/paragraph 단위 bounding box와 텍스트가 annotate돼 있습니다. 3단계로 만듭니다.

1. **Data filtering** — annotation 면적이 전체 이미지의 **0.01% 미만**인 것만 남김 (word-level 기준). 목적 두 가지: ① 난이도 보장 ② **crop tool 의존성 강제**
2. **Question generation** — text recognition("‘Busy’로 시작하는 line은 뭐라고 쓰여 있나"), counting("‘Television’으로 끝나는 문단에 ‘a’가 몇 번 나오나"), 문단 정보 검색형 QA 등을 programmatic하게 생성
   - **핵심 설계 원칙: 위치 단서를 일절 주지 않음.** "왼쪽 단어" 같은 표현도, bounding box 좌표도 안 줌 → 모델이 **자기 grounding 능력으로** 영역을 찾아야 함
3. **Multi-tool augmentation** — 각 이미지에 5가지 변형 중 하나를 랜덤 적용. **각 변형 비율을 동일**하게 맞춰 balanced. 결과적으로 **orientation 보정 + crop을 둘 다** 해야 풀림

### 5.3. Implementation

**Backbone 3종:** Qwen2.5-VL-7B, Qwen3-VL-8B-Thinking, Qwen3-VL-32B-Thinking
**RL 알고리즘: GRPO** (Shao et al. 2024)

| 단계 | 설정 |
|---|---|
| SFT | 2 epochs, batch 128, lr **5e-6**, cosine scheduler, warmup ratio 0.05 |
| RL | 2 epochs, batch 64, lr **1e-6**, **8 rollouts/sample**, KL coef **0.001** |

**Reward 계수:** format **0.1** / strategy **1.0** (must-use 1.0, suggested 0.2) / constraint penalty **0.5**

---

## 6. 결과

### Table 1 — transformed OCRBench & ChartQAPro

`Avg`는 **Source 포함 6개 컬럼의 평균**입니다 (Qwen2.5-VL-7B: (86.4+70.2+58.0+71.7+32.4+17.0)/6 = 56.0으로 검산됨).

**OCRBench (perception-critical)**

| Model | Source | Rot90 | Rot180 | Rot270 | Hori | Verti | **Avg** |
|---|---|---|---|---|---|---|---|
| GPT-4o | 83.1 | 61.9 | 46.6 | 63.7 | 48.1 | 12.9 | 52.7 |
| Gemini2.5-Pro | 87.1 | 68.2 | 67.9 | 71.5 | 39.4 | 41.3 | 62.6 |
| Qwen2.5-VL-32B | 85.0 | 64.5 | 45.6 | 64.8 | 25.5 | 8.7 | 49.0 |
| Qwen2.5-VL-72B | 88.5 | 72.6 | 58.3 | 73.5 | 35.1 | 18.2 | 57.7 |
| Qwen3-VL-30B-Thinking | 87.6 | 72.1 | 60.5 | 66.6 | 49.3 | 31.8 | 61.3 |
| Qwen3-VL-235B-Thinking | 88.8 | 76.4 | 71.0 | 74.3 | 45.9 | 23.8 | 63.4 |
| InternVL3.5-30B | 88.7 | 45.8 | 7.2 | 45.8 | 5.9 | 6.2 | 33.3 |
| InternVL3.5-241B | 92.3 | 58.2 | 32.4 | 57.6 | 25.6 | 9.4 | 45.9 |
| Thyme | 86.3 | 67.0 | 51.9 | 67.8 | 27.2 | 13.3 | 52.3 |
| Qwen2.5-VL-7B | 86.4 | 70.2 | 58.0 | 71.7 | 32.4 | 17.0 | 56.0 |
| **CodeVision-7B** | 87.2 | 72.3 | 73.1 | 75.2 | 65.1 | 67.4 | **73.4** |
| Qwen3-VL-8B-Thinking | 82.4 | 64.8 | 57.6 | 58.4 | 35.7 | 14.5 | 52.2 |
| **CodeVision-8B** | 83.5 | 78.6 | 77.4 | 76.7 | 68.7 | 67.3 | **75.4** |
| Qwen3-VL-32B-Thinking | 86.6 | 67.8 | 64.3 | 66.4 | 35.1 | 13.8 | 55.7 |
| **CodeVision-32B** | 87.8 | 82.8 | 79.3 | 81.1 | 77.7 | 68.3 | **79.5** |

**ChartQAPro (reasoning-heavy)**

| Model | Source | Rot90 | Rot180 | Rot270 | Hori | Verti | **Avg** |
|---|---|---|---|---|---|---|---|
| GPT-4o | 50.3 | 40.9 | 33.8 | 41.3 | 33.1 | 25.3 | 37.4 |
| Gemini2.5-Pro | 66.8 | 63.8 | 59.7 | 61.6 | 45.6 | 58.2 | 59.3 |
| Qwen2.5-VL-32B | 39.5 | 29.7 | 26.0 | 30.7 | 22.6 | 17.6 | 27.7 |
| Qwen2.5-VL-72B | 38.2 | 29.5 | 27.7 | 30.3 | 23.8 | 20.3 | 28.3 |
| Qwen3-VL-30B-Thinking | 50.2 | 38.7 | 36.5 | 38.2 | 31.3 | 29.0 | 37.3 |
| Qwen3-VL-235B-Thinking | 56.9 | 45.0 | 43.4 | 46.9 | 35.1 | 26.0 | 42.2 |
| InternVL3.5-30B | 37.2 | 27.2 | 18.0 | 27.9 | 17.5 | 18.0 | 24.3 |
| InternVL3.5-241B | 45.9 | 31.5 | 24.4 | 34.7 | 22.5 | 21.3 | 30.0 |
| Thyme | 30.3 | 23.8 | 20.4 | 22.9 | 17.8 | 14.4 | 21.6 |
| Qwen2.5-VL-7B | 37.3 | 23.4 | 22.2 | 23.7 | 19.5 | 20.1 | 24.4 |
| **CodeVision-7B** | 39.1 | 30.8 | 29.8 | 31.4 | 30.1 | 29.0 | **31.7** |
| Qwen3-VL-8B-Thinking | 47.2 | 32.8 | 30.0 | 31.9 | 21.3 | 13.9 | 29.5 |
| **CodeVision-8B** | 50.3 | 39.0 | 38.1 | 39.2 | 39.7 | 38.0 | **40.7** |
| Qwen3-VL-32B-Thinking | 52.3 | 39.9 | 38.3 | 42.0 | 26.1 | 18.7 | 36.2 |
| **CodeVision-32B** | 57.4 | 54.4 | 53.6 | 54.7 | 52.8 | 53.1 | **54.3** |

### Table 2 — single-tool & multi-tool

| Model | V* | HRBench4k | HRBench8k | **MVToolBench** |
|---|---|---|---|---|
| GPT-4o | 67.9 | 65.0 | 60.1 | 8.5 |
| Gemini2.5-Pro | 83.8 | **86.2** | **85.1** | 32.6 |
| Qwen2.5-VL-32B | 81.9 | 73.8 | 70.5 | 16.4 |
| Qwen3-VL-30B-Thinking | 81.2 | 77.8 | 71.3 | 23.7 |
| Qwen3-VL-235B-Thinking | 85.9 | 84.3 | 76.6 | 30.1 |
| Thyme | 82.2 | 77.0 | 72.0 | 24.2 |
| Qwen2.5-VL-7B | 74.6 | 69.4 | 67.5 | 18.1 |
| **CodeVision-7B** | 83.7 | 75.6 | 72.2 | **60.1** |
| Qwen3-VL-8B-Thinking | 77.5 | 72.4 | 68.1 | 19.7 |
| **CodeVision-8B** | 82.4 | 77.1 | 73.4 | **62.7** |
| Qwen3-VL-32B-Thinking | 84.8 | 82.1 | 74.8 | 28.6 |
| **CodeVision-32B** | **86.2** | 84.3 | 76.1 | **65.4** |

### ★ 여기서 반드시 읽어야 할 것

0단계에서 "최우선 확인 항목"으로 잡았던 transformation별 분해가 나왔습니다. **저자들이 본문에서 강조하지 않은 패턴들**이 보입니다.

**(1) 취약성은 rotation이 아니라 flip에 몰려 있다.**

Qwen2.5-VL-7B 기준 rotation은 58.0~71.7인데 **Hori 32.4, Verti 17.0**입니다. InternVL3.5-30B는 더 극단적입니다 — Rot180 **7.2**, Hori **5.9**, Verti **6.2**. Introduction의 "최대 80% 성능 하락"은 사실상 **flip과 Rot180이 만들어낸 숫자**입니다.

**(2) CodeVision-7B의 이득도 대부분 flip에서 나온다.**

| 변형 | Qwen2.5-VL-7B | CodeVision-7B | Δ |
|---|---|---|---|
| Rot90 | 70.2 | 72.3 | **+2.1** |
| Rot180 | 58.0 | 73.1 | +15.1 |
| Rot270 | 71.7 | 75.2 | **+3.5** |
| Hori | 32.4 | 65.1 | **+32.7** |
| Verti | 17.0 | 67.4 | **+50.4** |

**rotation 90°/270°에서의 개선은 +2~4%p에 불과합니다.** 평균 +17.4라는 헤드라인 숫자는 flip이 끌어올린 것입니다. 32B에서는 rotation 이득도 커지지만(67.8→82.8), **7B 스케일에서 rotation은 여전히 잘 안 됩니다.**

> **이게 우리 과제에 결정적입니다.** RotBench가 지적한 "90°와 270°를 구분 못 한다"는 문제를 **CodeVision은 사실상 해결하지 못했습니다.** 소형 모델 스케일에서 rotation은 여전히 열려 있고, 정확히 거기가 우리 자리입니다. 논문에 이 분해 표를 그대로 인용해 "선행 연구의 이득은 reflection에 편중돼 있고 rotation은 미해결"이라고 쓸 수 있습니다.

**(3) reasoning-heavy task에서는 방법이 훨씬 덜 통한다.**

ChartQAPro에서 CodeVision-7B는 **31.7**로, base(24.4) 대비 +7.3에 그치고 **Gemini2.5-Pro(59.3)에 크게 못 미칩니다.** OCRBench(+17.4)와 대비됩니다. orientation 보정이 perception에는 직접 효과가 있지만 reasoning에는 병목이 다른 데 있다는 뜻입니다.

**(4) untransformed 이미지에서 성능 저하(regression)는 없다.**

Source 컬럼: 86.4→87.2 (7B), 82.4→83.5 (8B), 86.6→87.8 (32B). ChartQAPro도 37.3→39.1, 47.2→50.3, 52.3→57.4. **소폭 상승**입니다. Inappropriate Tool Use Penalty가 작동한 것으로 보입니다. — "tool을 붙이면 정상 이미지가 망가진다"는 반론은 이 논문 상대로는 안 통합니다.

### 학습 곡선 (Figure 5, 7, 17)

**Figure 5 — RL 학습 곡선 (700 step)**

| 성분 | 시작 → 끝 |
|---|---|
| Accuracy | ~0.28 → **~0.72** |
| Strategy | ~0.22 → **~0.62** |
| Penalty | ~0.09 → step 200 이후 **~0에 수렴** |

Penalty가 초반에 빠르게 0으로 떨어지는 게 눈에 띕니다. reward hacking 억제가 학습 초기에 완료된다는 뜻.

*(사소한 흠: 캡션은 "outcome, strategy, and total rewards"라고 적었는데 범례는 Accuracy/Strategy/**Penalty**입니다. total 곡선은 없습니다.)*

**Figure 7 — emergent tool 사용 성공률** ★

본문은 "consistent upward trend"라고 서술하지만, **실제 곡선은 점진적 상승이 아니라 phase transition에 가깝습니다.**

- step 0 ~ **270**: **0.0에서 완전히 평탄** (emergent tool 사용 성공이 사실상 없음)
- step **270 ~ 350**: **0.0 → ~0.65로 급등**
- step 350 ~ 1100: ~0.7–0.8에서 진동

즉 emergence가 **특정 시점에 갑자기 켜집니다.** 학습을 270 step 전에 끊었다면 이 능력을 아예 못 봤을 것이라는 뜻이고, RL 예산 산정에 직접 영향을 주는 관찰입니다.

**Figure 17 — 학습 중 benchmark 정확도 (1000 step)**

| Benchmark | 궤적 |
|---|---|
| OCRBench-Rot90 | ~54 → **step 200에서 ~46으로 하락** → 400에서 ~72.5 → 이후 **~74.5–76에서 평탄** |
| ChartQAPro-Hori | ~31 → 400에서 ~38.2 → 이후 ~38–40에서 진동 |
| MVToolBench | ~30 → 500에서 ~59.5 → 1000에서 ~62.5 |

두 가지 관찰:

1. **OCRBench-Rot90에 초기 하락 구간이 있습니다** (step 100→200에서 54→46). SFT 체크포인트에서 RL로 넘어가는 전이 구간의 불안정성으로 보입니다. 저자들은 언급하지 않습니다.
2. **저자의 "no signs of plateauing" 주장은 그래프와 잘 맞지 않습니다.** Limitations에서 "성능이 계속 오르며 plateau 조짐이 없다 → scaling하면 더 좋아진다"고 적었는데, 실제로는 세 곡선 모두 step 400~500 이후 뚜렷하게 완만해집니다. MVToolBench는 500→1000의 500 step 동안 59.5→62.5로 3%p 오르는 데 그칩니다. **상승이 멈춘 건 아니지만 "포화 조짐이 없다"고 하긴 어렵습니다.** scaling으로 rotation 문제가 해결될 거라는 기대는 이 그래프로는 뒷받침되지 않습니다.

---

## 7. Ablation (Table 3)

CodeVision-7B 기준.

| Model | OCRBench Rot180 | OCRBench Verti | ChartQAPro Rot180 | ChartQAPro Hori | V* | MVToolBench |
|---|---|---|---|---|---|---|
| Qwen2.5-VL-7B | 70.2 | 17.0 | 23.4 | 19.5 | 74.6 | 18.1 |
| Qwen2.5-VL-7B-**SFT only** | **57.0** | 35.8 | 23.2 | 20.9 | **71.7** | 26.6 |
| **CodeVision-7B (full)** | 72.3 | 67.4 | 30.8 | 30.1 | 83.7 | 60.1 |
| w/o Strategy Reward | 60.9 | 61.5 | 24.6 | 28.9 | 78.5 | **50.7** |
| w/o Penalty | 68.3 | 66.3 | 24.0 | 24.3 | **71.2** | 55.9 |

**읽을 점 세 가지:**

1. **SFT만 하면 오히려 나빠지는 항목이 있다.** OCRBench Rot180이 70.2 → **57.0**, V*가 74.6 → **71.7**로 base보다 떨어집니다. 성능을 만드는 건 RL이고, SFT는 어디까지나 bootstrap입니다. (저자들은 이 표를 "SFT 단독의 회귀"라는 각도로는 논평하지 않습니다.)
2. **Strategy reward 제거의 타격이 가장 크다** — MVToolBench 60.1 → 50.7. outcome reward만으로는 복잡한 tool-use 전략을 못 배운다는 근거.
3. **Penalty 제거는 V*에 치명적** — 83.7 → 71.2. reward hacking(불필요한 crop 반복, 이미 올바른 이미지 회전)이 실제로 성능을 갉아먹습니다.

### Figure 15 — reward 성분별 학습 동역학

| 패널 | CodeVision (full) | w/o Strategy | w/o Penalty |
|---|---|---|---|
| (a) Penalty Term | ~0.05에서 평탄 | ~0.2까지 상승 | **~0.35까지 상승**, 변동 극심 |
| (b) Entropy | 0.8 → **~0.2**로 꾸준히 하강 | ~0.45에서 정체 | ~0.42에서 정체 |
| (c) **Tool Turns** | **~1.0에서 평탄** | step 300 부근 **~2.5–3.0으로 급등** | **~3.5–4.0으로 급등** |

**(c)가 결정적입니다.** penalty를 빼면 tool turn이 **3.5~4회로 폭증**합니다. §4.2에서 저자가 서술한 reward hacking — "rotate90/180/270을 순서대로 다 호출해서 strategy reward를 긁어모으는" 행동 — 이 그래프로 확인됩니다. full model은 **평균 1턴**을 유지합니다.

또 하나: **ablated run들은 step 400~450에서 곡선이 끊깁니다** (full은 600까지). 학습이 붕괴해 중단된 것으로 보입니다. 저자들이 서두에 언급한 "training collapse가 자주 발생한다"는 서술과 맞물립니다.

### Figure 16 — cold start의 필요성 ★

SFT 없이 base에서 바로 RL을 돌린 경우(`w/o SFT`), 본문은 "의미 있는 개선에 실패"라고만 적었지만 **그래프는 훨씬 극적입니다.**

| 패널 | w. SFT | w/o SFT |
|---|---|---|
| (a) Entropy | 0.78 → ~0.19 (600 step) | 0.62 → ~0.3, **step ~230에서 run 종료** |
| (b) **Tool Turns** | ~1.4 → ~0.95 유지 | **step ~30에서 0.0으로 붕괴, 이후 계속 0** |
| (c) Accuracy Reward | 0.3 → **~0.7** | **~0.15–0.2에서 평탄** |
| (d) Strategy Reward | 0.2 → **~0.58** | **0.0에서 평탄** |

**SFT 없는 모델은 tool을 아예 부르지 않게 됩니다.** turn이 0으로 붕괴하고 strategy reward가 0에 고정된다는 건, "성능이 덜 올랐다"가 아니라 **tool use라는 행동 자체가 소멸했다**는 뜻입니다. RL이 "tool을 안 쓰는 게 이득"이라는 local optimum으로 즉시 수렴한 것입니다.

> **우리 과제에 매우 중요합니다.** 앞단에 소형 reasoning 모델을 두고 tool call을 학습시키려면, **pure RL로는 부트스트랩이 안 됩니다.** SFT cold start가 선택이 아니라 필수 조건이라는 걸 이 그림이 보여줍니다. 모델이 작을수록 더 심할 가능성이 높고요. 실험 계획에 SFT 데이터 구축 비용을 반드시 반영해야 합니다.

---

## 8. Case study와 저자가 밝힌 한계

**Case study (Figure 6, 11~14)**

| Figure | 내용 |
|---|---|
| 6 | error recovery. `flip-horizontal`을 잘못 호출 → 실행 결과 보고 오류 인지 → `rotate-90`으로 교정 |
| 9 / 10 | emergent tool. RL 데이터에 없던 contrast·grayscale을 한 turn에 chaining. Fig 10은 **5개 tool(brightness↑, contrast↑, crop, rotate90, sharpness)**을 자발적으로 조합, 그중 3개는 RL 데이터에 없던 것 |
| 11 | **reward hacking 실패 사례** (penalty 없이 학습). `rotate90`으로 이미 교정했는데도 strategy reward를 더 얻으려 추가 orientation tool을 호출 → 이미 맞던 이미지를 망가뜨리고 최종 실패 |
| 12 | 성공 사례. crop이 불완전함을 인지하고 2차 crop으로 refine |
| 13 | "성공했지만 비효율". crop이 길고 좁은 띠 모양이라 무관한 영역을 많이 포함 — 과도하게 보수적인 safe cropping 경향 |
| 14 | **실패 사례.** orientation 교정과 대략적 위치 추론은 성공했으나 **정밀 좌표 예측에서 실패** — crop이 목표 영역 바로 옆으로 빗나감 |

**Figure 8 — emergent tool word cloud 판독** ★

RL 학습 중 발굴된 tool 전체 목록입니다:

```
brightness_down, brightness_up, contrast_up, autocontrast,
grayscale, gaussian_blur, smooth, sharpen, sharpness,
edge_detect, resize
```

**여기서 결정적인 관찰이 하나 나옵니다 — 11개가 전부 photometric(밝기·대비·블러·샤프닝·엣지) 연산이고, `resize` 하나만 geometric입니다.** 그마저도 크기 변환일 뿐입니다.

**임의 각도 rotation도, shear도, perspective 보정도 emerge하지 않았습니다.** code-as-tool은 원리상 `image.rotate(37)`을 못 부를 이유가 없는데도요. 즉 **"무한한 toolset"이라는 주장이 geometry 축에서는 실현되지 않았습니다.** 모델이 새로 발견한 건 전부 "픽셀 값을 바꾸는" 연산이지 "픽셀 위치를 바꾸는" 연산이 아닙니다.

> 이건 §9-④(continuous rotation이 빈 자리) 주장의 **직접적 증거**입니다. 저자들의 emergence 사례를 그대로 인용하면서 "단, geometric transformation은 emerge하지 않았다"고 지적할 수 있습니다.

**저자가 명시한 Limitations**

1. **Tool 다양성** — 의도적으로 orientation correction + cropping으로 tool을 한정했음. 향후 multi-image tool use(비교·병합), 그리고 Python 표준 라이브러리를 넘어 **API endpoint만 노출하는 custom tool**(검색 엔진, generative model)로 확장 필요
2. **Process supervision** — 현재는 "must-use" 목록에 의존. 더 유연한 "beneficial tool" 프레임으로 확장 필요
3. **Scaling** — 학습이 아직 saturate되지 않았음. policy entropy에 탐색 여지가 남아 있고, Figure 17에서 성능이 계속 오르며 plateau 조짐이 없음

그리고 case study에서 드러난 실질적 한계: **fine-grained coordinate prediction이 여전히 약함.** high-level reasoning과 coarse localization은 잘 하지만 정밀 bounding box 생성이 안 됩니다.

---

## 9. 우리 과제 관점 — 무엇이 남았나

### 이 논문이 가져간 것

- "MLLM이 orientation 변화에 취약하다"는 **진단**과 그 정량화(Table 1)
- 회전 보정을 tool call로 푸는 **접근 자체**
- 변형된 OCRBench/ChartQAPro라는 **평가 프로토콜**
- **MVToolBench**라는 multi-tool benchmark
- "정상 이미지는 건드리지 말라"는 abstain 아이디어의 **일부** (Inappropriate Tool Use Penalty)

0단계의 "경쟁 논문" 판단은 맞습니다. 문제 정의 수준에서 상당 부분 선점됐습니다.

### 그래도 남아 있는 자리 (근거와 함께)

**① rotation은 실제로 안 풀렸다** ★가장 강함

§6-(2)의 분해가 근거입니다. **CodeVision-7B의 Rot90 개선은 +2.1, Rot270은 +3.5에 불과**합니다. 평균 +17.4는 flip이 만든 숫자입니다. RotBench의 "90°/270° 구분 실패"가 **이 논문에서도 그대로 재현**됩니다. 우리가 rotation에 집중할 정당성이 여기서 나옵니다.

**② 본 모델을 freeze할 수 없다**

| 축 | CodeVision | 우리 구상 |
|---|---|---|
| 학습 대상 | 본 모델 전체 SFT+RL | 본 모델 **freeze**, 앞단만 |
| 본 모델 교체 | 모델마다 **전체** 재학습 | 앞단만 재학습 |
| 학습 비용 | 7B/8B/32B 전체 | 소형 앞단만 |
| 전제 조건 | 본 모델이 코드 생성 가능해야 함 | 무관 |

> **⚠ closed-source 논거는 조건부입니다.** 예전에 여기 "GPT-5·Gemini에는 CodeVision을 못 쓰지만 우리는 된다"고 적어뒀는데, **우리가 본 모델 feature를 쓰기로 하면 우리도 못 씁니다.** A-4형(feature 활용)을 택하면 앞단이 작아지고 학습이 싸지는 대신 white-box 전용이 되고, A-2형(독립 네트워크)을 택하면 그 반대입니다. **어느 쪽을 택할지가 주장 범위를 정합니다** — 상세는 `논문 비교 (A-1 vs A-2 vs A-4).md` §5.
>
> 다만 **freeze 자체의 이득은 어느 쪽이든 남습니다** — 본 모델을 건드리지 않으니 다른 능력이 퇴화하지 않고, 학습 비용이 앞단으로 국한됩니다.

**③ resampling artifact를 통제하지 않았다**

논문 전체에 artifact 관련 언급이 **없습니다.** 데이터·benchmark 모두 digital transformation으로 만들어졌습니다. A-4(Artificial Mental Rotation)의 경고 — 디지털 회전은 사람 눈에 안 보이는 알고리즘 고유의 artifact를 남기고, 모델이 그걸 shortcut으로 배울 수 있다 — 에 그대로 노출돼 있습니다.

특히 **flip에서 이득이 압도적으로 큰 것(+50.4)이 의심스럽습니다.** flip은 rotation과 달리 interpolation이 없는 무손실 pixel 재배열이라 artifact 특성이 다릅니다. 모델이 semantic이 아니라 **artifact 패턴으로 변형 종류를 맞히고 있을 가능성**이 있습니다. → **"실제 촬영 회전 vs 디지털 회전"에서 CodeVision을 재평가하는 것만으로 반증 실험이 성립합니다.**

**④ discrete 4방향만 다룬다**

must-use tool set이 `{rotate90, rotate180, rotate270, flip-h, flip-v, crop}` 6개로 닫혀 있습니다. **임의 각도(예: 37°) rotation은 다루지 않습니다.** 실제 사진의 기울어짐은 연속적인데, 여기서 continuous rotation이 통째로 빈 자리입니다.

그리고 **Figure 8이 이 주장의 결정적 증거입니다.** 저자들이 자랑하는 emergent tool 11개가 전부 photometric 연산이고 geometric은 `resize` 하나뿐입니다. code-as-tool이 원리상 `image.rotate(37)`을 부를 수 있는데도 **RL은 새로운 geometry를 한 번도 발견하지 못했습니다.** "무한한 toolset"이라는 프레임이 geometry 축에서는 작동하지 않은 것이고, 이건 우연이 아니라 **must-use tool set을 4방향으로 닫아둔 process supervision의 직접적 귀결**로 보입니다. 저자들 스스로 Limitations에서 "must-use 목록 의존"을 한계로 인정합니다.

**⑤ abstain은 penalty일 뿐 policy가 아니다**

Inappropriate Tool Use Penalty는 학습 중 신호일 뿐, **"이 이미지는 방향이 정의되지 않으니 건드리지 않는다"를 판단하는 명시적 정책도, 그 판단 능력에 대한 평가도 없습니다.** A-5가 지적한 "방향이 애매한 이미지"(하늘, 텍스처) 케이스는 다뤄지지 않습니다. benchmark가 전부 텍스트 중심(OCRBench, HierText)이라 **방향이 항상 잘 정의되는 데이터만 씁니다.** B-4의 Tool Benefit Score를 회전에 재정의하는 아이디어는 여전히 유효합니다.

### 실험 설계에 바로 반영할 것

- **평가 지표를 transformation별로 쪼개서 보고할 것.** 평균 하나로 뭉치면 이 논문처럼 flip 이득이 rotation 실패를 가립니다.
- **Source 컬럼(untransformed)을 반드시 함께 보고할 것.** regression 없음을 증명하는 게 앞단 모델 방식의 필수 방어선입니다.
- **perception task와 reasoning task를 분리할 것.** OCRBench/ChartQAPro 이분법은 잘 만든 설계라 그대로 차용할 만합니다.
- **SFT cold start를 예산에 반드시 포함할 것.** Figure 16이 보여주듯 pure RL은 tool use 행동 자체를 소멸시킵니다(tool turn → 0). 앞단 모델이 소형일수록 더 심할 가능성이 높습니다. "RL만으로 앞단을 학습시킨다"는 계획은 이 그림 하나로 반박됩니다.
- **RL 예산을 최소 300 step 이상 잡을 것.** Figure 7에서 emergent tool 사용이 step 270 부근에서야 phase transition으로 켜집니다. 그 전에 끊으면 능력이 없다고 오판하게 됩니다.
- **방향이 애매한 이미지를 평가셋에 넣을 것.** 이 논문의 benchmark는 전부 텍스트 중심(OCRBench, ChartQAPro, HierText)이라 방향이 항상 잘 정의됩니다. 하늘·텍스처·추상 패턴을 넣으면 선행 연구가 측정하지 않은 축이 열립니다.

---

## 출처

원문 PDF 직접 대조 완료 (arXiv 2512.03746v1, 20쪽). 본문·Table 1~3·Implementation Details·Limitations 전부 확인.
Figure 1·5·7·8·15·16·17은 해당 페이지를 200 dpi로 렌더링해 직접 판독했습니다. 그래프에서 눈으로 읽은 값은 `~`로 표기했고, 정밀한 값이 필요하면 원 그림을 다시 볼 것.
