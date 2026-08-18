## Visual Tool Reasoning 방법론 — rotated OCR × ≤3B 적합성 분류

> **과제 전제:** ① 회전된 문서의 OCR을 살리는 것 ② 학습 대상은 **3B 이하 소형 모델** ③ 얼린 본 모델의 **내부 feature 접근 가능**
>
> visual tool reasoning 계열 방법론들이 이 조건에서 쓸 만한지 판정합니다. 원문 미확인(초록·검색 요약 기반)이므로 채택 전 대조가 필요합니다.

---

## 1. ★ 가장 중요한 논문

### Seeing Straight: Document Orientation Detection for Efficient OCR (arXiv 2511.04161)

**우리 스코프를 거의 그대로 다룬 선행 연구입니다.**

| 항목 | 내용 |
|---|---|
| 구조 | **앞단 rotation 분류기 → OCR 모델** (얼린 다운스트림) |
| 앞단 | **Phi-3.5-Vision의 vision encoder** + dynamic image cropping, **12-class 회전 분류**로 fine-tune |
| 성능 | 회전 식별 **98% / 96%** |
| OCR 개선 | closed-source **최대 20%**, open-weights **최대 4배** |
| 벤치마크 | **OCR-Rotation-Bench (ORB)** — 1,863장. ORB-En + **ORB-Indic**(11개 인도계 저자원 언어) |

**이 논문의 두 얼굴:**

- ✅ **자산:** 우리 구조(앞단 + 얼린 본 모델)가 rotated OCR에서 실제로 큰 효과가 있다는 걸 이미 증명해줍니다. 그리고 함께 공개한 **ORB가 rotated OCR 전용 평가셋**이라 평가 문제까지 해결됩니다.
- ⚠️ **위협:** *"문서 회전을 앞단에서 고쳐 OCR을 살린다"* 는 아이디어 자체는 **이미 나왔고, 98%로 거의 풀렸습니다.** 그것도 **소형 모델의 vision encoder에 분류 헤드**를 붙인, 아주 단순한 방법으로요.

**→ 그래서 우리가 답해야 할 질문이 바뀝니다:** *"앞단을 붙일까?"*가 아니라 **"분류기로 98%가 나오는데 왜 reasoning 모델이 필요한가?"**입니다. §4에 답 후보를 정리했습니다.

> 이 논문은 canonicalization 계열과 사실상 같은 구조이고, **A-4(AMR)와 마찬가지로 본 모델의 vision encoder feature를 재활용**합니다. 본 과제도 feature 접근이 가능하므로 이 설계를 그대로 따라갈 수 있습니다.

---

## 2. 3B 제약이 만드는 분류 기준

문헌에서 확인된 소형 모델의 능력 경계입니다.

- 2B 미만 SLM도 **단일 턴 벤치마크(GSM8K·MMLU)에서는 대형과 비슷**하지만, **tool 호출·장기 계획·에러 복구의 핵심인 다단계 추론에서 크게 뒤집니다.**
- 반면 **구조화된 tool calling**(JSON schema 강제, guided decoding, validator-first)을 쓰면 **격차가 상당히 메워집니다.**
- **open-ended long-horizon**은 8B·30B에서도 어렵습니다.

여기에 CodeVision 원문에서 확인한 증거를 더하면:

> **CodeVision(7B) Figure 16** — SFT cold start 없이 RL만 돌리자 **tool turn이 step 30에서 0으로 붕괴**하고 끝까지 0이었습니다. 저자 설명: *"code generation의 방대하고 비구조적인 action space 때문에 pure RL exploration으로는 유용한 정책을 못 찾는다."* **7B에서 이랬습니다. 3B는 더 심할 수밖에 없습니다.**

**→ 분류 기준 3가지**

| 기준 | 3B 친화 | 3B 위험 |
|---|---|---|
| **출력 형태** | 고정 tool + 구조화된 인자 | **자유 코드 생성** |
| **호출 길이** | 1~2턴 | 수십 턴 long-horizon |
| **학습** | SFT cold start **또는** tool-supervised RL(§3-B ②) | 맨바닥 RL-only 부트스트랩 |

---

## 3. 분류 결과

### ✅ 적합 — 3B에서 바로 시도 가능

- **Seeing Straight** ★ *(tool reasoning이 아니라 분류 파이프라인)* — 우리 스코프(문서 회전 → OCR)를 그대로 다룬 **가장 가까운 선행 연구**이고 **소형 모델 vision encoder + 분류 헤드만으로 98%**를 냈으므로, **가장 먼저 재현해 성능 상한을 잡아야 할 baseline**이다(§1·§4).
- **Chain-of-Focus** — 고정 crop/zoom 구조라 출력이 단순하고, tool만 rotate로 바꾸면 그대로 최소 baseline이 된다.
- **OpenThinkIMG** — 표준 tool interface와 RL 환경이 이미 갖춰져 있어 밑바닥부터 만들 필요가 없다.
- **Beacon** ★ — "tool이 실제로 도움이 됐는가"를 학습 신호로 삼는데, rotated OCR에서는 **회전 전후 OCR 정확도 차이**로 그걸 teacher 없이 공짜로 잴 수 있다 → *"부를까 말까"*가 아니라 **"어느 각도가 맞았나"의 graded reward**로 전용 가능.
- **ToolsRL** ★ — **rotate·flip이 이미 tool 목록에 있고**, 2단계 curriculum이 비싼 trajectory 없이 tool 사용법을 가르쳐 **3B의 cold start 부담을 덜어준다**.
- **ReVPT** — 고정 tool 4개를 GRPO로 학습해 **"2B 스케일에서 특히 큰 향상"**을 보고한 사례라, 소형에서 tool-use RL이 실제로 된다는 직접 증거다.

### ⚠️ 조건부 — 3B에서 위험, 보완 필요

- **DeepEyes** — "SFT 없이 RL만으로 된다"를 **7B에서** 검증한 것이라, 3B에서 그대로 믿고 가면 위험하다 → **SFT cold start 필수**.
- **Pixel Reasoner** — 방법 자체보다 *"RL이 text-only local optimum으로 붕괴한다"*는 문제의식이 3B에서 더 중요하다 → 개념만 차용.
- **Visual Sketchpad** — training-free지만 **강한 base 모델을 전제**하므로 3B에서 sketch 품질을 기대하기 어렵다.

### ❌ 부적합 — 3B에서 비권장

- **Thyme / PyVision** — 자유 Python 코드 생성은 3B에서 붕괴 위험이 큰데(§2), **회전은 애초에 4~6지선다라 그 유연성이 필요 없다** — 대가만 치르고 이득이 없다.
- **Mini-o3** — 수십 step long-horizon은 문헌상 8B·30B에서도 어려운 영역이다.
- **VC-Tooler** — multi-tool composition과 unseen tool 일반화가 목표인데, **우리 tool은 1~2개**라 풀 문제 자체가 없다.
- **Act Wisely / Metis** — tool 과용 억제(98%→2%)가 목표인데 rotated OCR은 거의 매 입력에 회전이 필요해서 **억제할 과용 자체가 없다**.

### 🤔 참고만

- **FaithEyes** — "잘못된 tool output이 추론을 오염시킨다"는 **문제의식은 우리에게도 실재**하지만(앞단이 잘못 회전시키면 본 모델이 틀린 전제 위에서 추론), **judge를 쓰는 해법은 불필요**하다 — OCR 결과 자체가 검증 신호이므로.
- **TextCall** — *"tool 결과 이미지 없이 tool-call만으로 gain이 나는가"*를 분석하는데, **회전은 돌린 이미지를 다시 봐야 글자를 읽으므로 성립할 수 없다** → 역으로 *"우리 과제는 tool result가 필수인 케이스"*라는 근거로 인용 가능.
- **AgenticOCR** — "**4B가 GRPO로 OCR tool 호출을 학습한다**"는 실현 가능성 증거로는 유효하지만, tool이 zoom-and-ocr이고 목표가 RAG token 절감이라 **회전과는 무관하다**.
- **Jigsaw-R1** — 회전이 아니라 jigsaw 퍼즐 연구지만 **"정답이 공짜로 나오는 rule-based reward"라는 설계가 회전과 동일**하고, 그 결론(*"명시적 reasoning 없이도 학습·일반화되며, 복잡한 추론 패턴은 emergent가 아니라 pre-existing"*)은 **우리 reasoning 방식에 불리한 증거**다 → §4 참조.

### 📊 방법론이 아니라 평가셋

- **TIR-Bench** ★ — 13개 task 중 **Rotated OCR이 명시적으로 포함**되고 최고 성능이 46%라 포화되지 않아, **다운스트림 평가셋으로 바로 쓸 수 있다**.
- **ORB** ★ *(Seeing Straight 부속)* — 1,863장의 rotated OCR 전용 평가셋이고 **ORB-Indic으로 11개 저자원 언어까지 커버**해, 우리 과제에 가장 정확히 맞는 평가셋이다.

---

## 3-B. 주요 논문 확인 사항

혼동하기 쉬운 논문들의 실제 내용입니다.

| 논문 | 실제 내용 |
|---|---|
| **TIR-Bench** (2511.01833) | 13개 task 중 **"Rotated OCR"이 명시적으로 포함**. 22개 MLLM 평가에서 **최고 46%**. tool 있는 모델(o3·o4-mini·PyVision)이 크게 앞섬 |
| **ToolsRL** (2604.19945, CVPR'26) | *Visual Reasoning through Tool-supervised RL* (Amazon). tool에 **rotate·flip 명시 포함**. Qwen2.5-VL-7B |
| **ReVPT** (2509.01656) | *Reinforced Visual Perception with Tools*. GRPO + 4개 tool(detection·zoom·edge·depth). **"2B 스케일에서 특히 큰 향상"** |
| **AgenticOCR** (2602.24134) | ⚠️ **이름과 달리 rotation 타깃이 아님.** *Parsing Only What You Need for **Efficient RAG***. tool이 **zoom-and-ocr**이고 목표는 **visual token 예산 절감**. 4B/8B·GRPO·OCR은 맞지만 **회전과 무관** |
| **Jigsaw-R1** (2505.23590) | ⚠️ **회전이 아니라 jigsaw 퍼즐.** 섞인 패치의 원래 위치 인덱스를 맞히는 과제로, rotation prediction과는 다른 pretext task. 다만 결론이 중요(§4) |

### 여기서 나오는 핵심 근거 3가지 ★

**① TIR-Bench — 우리 접근을 정면으로 뒷받침하는 발견**

> *"이미지 방향을 먼저 복원하고 텍스트를 출력하는 **agentic 방식은, 회전된 데이터로 모델을 직접 fine-tuning할 때 생기는 catastrophic forgetting을 피한다**."*

**지금까지 나온 것 중 "본 모델을 얼리고 앞단을 두는" 설계에 대한 가장 직접적인 실증 근거입니다.** 그동안 근거가 비용·적용가능성이었는데, 여기서 **성능상의 이유**가 생겼습니다 — 직접 학습시키면 다른 능력이 망가진다는 것.

**② ToolsRL — SFT cold start를 우회할 수 있는 경로**

2단계 curriculum으로 **tool 습득과 답 최적화를 분리**합니다.

```
Stage 1 : ground-truth 기반 per-tool reward로 tool 사용법만 학습
Stage 2 : GRPO로 답 정확도 최적화 (학습된 tool을 자유 호출)
```

핵심은 **"비싼 curated tool-use trajectory가 필요 없다"**는 점입니다. **회전은 정답 각도를 우리가 직접 걸어서 만들기 때문에 per-tool reward가 공짜로 나옵니다.** → §2에서 "3B는 SFT cold start 필수"라고 했는데, **ToolsRL Stage 1이 더 싼 대안**이 될 수 있습니다.

**③ ReVPT — 소형에서 tool-use RL이 된다는 증거**

**"2B 스케일에서 특히 큰 향상"**을 보고합니다. CodeVision Fig 16의 붕괴와 상충하는 것처럼 보이지만 아닙니다 — **ReVPT는 고정 tool 4개**를 쓰고 CodeVision은 자유 코드 생성이었어요. **§2의 분류 기준(고정 tool은 3B 친화, 자유 코드는 위험)을 오히려 뒷받침합니다.**

*(분류 결과는 §3 참조.)*

---

## 4. 스코프를 OCR로 좁힐 때의 득실 ★

**반드시 인지하고 가셔야 할 트레이드오프입니다.**

### 얻는 것

- **문제가 쉬워집니다.** 텍스트는 읽기 방향이 있어서 **90°와 270°가 명확히 구분됩니다.** 자연 장면에서 어려웠던 부분이 사라져요.
- **검증 신호가 공짜로 생깁니다.** 회전을 제대로 맞히면 OCR 결과가 급격히 좋아집니다. reward·평가 모두 여기서 나옵니다.
- **실용적 가치가 명확합니다.** 스캔·촬영 문서의 방향 보정은 실제 파이프라인의 표준 전처리 단계입니다.

### 잃는 것 — 이게 중요합니다

**RotBench의 "90°/270°를 구분하는 모델이 하나도 없다"는 논거를 더 이상 쓸 수 없습니다.** 그 문제는 **방향 단서가 약한 자연 이미지**에서 발생합니다. 텍스트가 있으면 읽기 방향이 곧 정답이라 애초에 어렵지 않아요.

즉 **스코프를 OCR로 좁히는 것은 가장 어려운 케이스를 피해 가는 선택**입니다. 나쁜 선택이 아니라 **정당한 범위 설정**이지만, *"아무도 90°/270°를 못 구분한다"*를 motivation으로 쓰면서 *"우리는 문서 OCR만 다룬다"*고 하면 **논리가 어긋납니다.** 심사에서 바로 지적받을 지점입니다.

### 그래서 motivation을 어디서 가져올 것인가

RotBench 대신 이쪽이 맞습니다:

- **Seeing Straight의 수치** — 회전만으로 open-weights OCR 성능이 **1/4 토막**이 납니다. 실용적 심각성이 여기 있습니다.
- **CodeVision Table 1** — OCRBench에서 Qwen2.5-VL-7B가 Rot180에서 86.4 → **58.0**으로 떨어집니다.
- **VLM-RobustBench** — "의미적으로 강하지만 공간적으로 취약하다".

### 그리고 "왜 분류기가 아니라 reasoning인가"에 답해야 합니다

**⚠️ 이 질문이 생각보다 무겁습니다. 독립된 두 증거가 "reasoning이 필요 없을 수도 있다"를 가리킵니다.**

| 출처 | 내용 |
|---|---|
| **Seeing Straight** | 문서 회전을 **분류 헤드만으로 98%** 달성 |
| **Jigsaw-R1** | 같은 성격(rule-based 정답이 있는 공간 과제)에서 *"**명시적 reasoning 없이도** 학습·일반화된다"*, 그리고 *"복잡한 추론 패턴은 **emergent가 아니라 pre-existing**"* |

**Jigsaw-R1의 두 번째 결론이 특히 3B에 불리합니다.** 추론 패턴이 학습으로 생겨나는 게 아니라 **이미 있던 게 드러나는 것**이라면, 그 패턴이 애초에 약한 3B 모델에서는 RL을 돌려도 나타나지 않을 수 있습니다.

**→ 그러니 reasoning 방식을 밀려면 "정확도가 더 높다"가 아니라 다른 축에서 이유를 대야 합니다.** 후보:

1. **abstain** — 12-class 분류기는 "판단 보류"를 표현할 수 없습니다. 방향이 애매한 문서(도표만 있는 페이지, 회전 대칭 양식)에서 강제로 한 클래스를 고릅니다.
2. **설명 가능성** — 왜 그 각도로 판단했는지 근거를 남길 수 있습니다. FaithEyes·TreeBench의 문제의식과 연결됩니다.
3. **일반화** — 분류기는 학습 도메인(문서)에 갇힙니다. reasoning 모델은 간판·손글씨·장면 텍스트로 넓히기 쉽습니다. **ORB가 문서 중심이라는 점이 곧 빈틈입니다.**
4. **다중 tool** — 회전 + crop이 함께 필요한 경우(작은 글씨가 기울어져 있는 경우). 분류기는 회전밖에 못 합니다.

**3번과 4번이 가장 방어하기 좋아 보입니다.** 1번은 매력적이지만 §6-2에서 봤듯 이미 부분 선점돼 있고, 2번은 "그래서 성능이 오르나?"라는 반문에 약합니다.

---

## 5. 권장 실험 경로

**3B·회전·OCR 제약에 맞춰 압축한 경로**입니다.

```
[0] 진단 — 본 모델 vision encoder에 회전 정보가 있는가?  ← 먼저
     └ feature probing. A-4/Seeing Straight의 전제가 우리 모델에서도 성립하는지

[1] baseline 2종 비교
     ├ (a) 분류 헤드         ← Seeing Straight 재현. 성능 상한 확인용
     └ (b) 고정 tool + 추론   ← Chain-of-Focus / OpenThinkIMG 구조, rotate로 교체

[2] tool 사용법 부트스트랩    ← 맨바닥 RL은 3B에서 붕괴 (CodeVision Fig 16)
     ├ (a) SFT cold start
     └ (b) ToolsRL Stage 1 — 정답 각도를 우리가 알고 있으므로 per-tool reward가 공짜

[3] RL — reward에 OCR 결과를 직접 사용
     └ Beacon식 tool-induced gain = 회전 전후 OCR 정확도 차이

[4] (선택) abstain — 방향이 애매한 문서를 섞어 평가
     └ 문서 스코프에서는 비중이 작음. 필요해지면 간단한 penalty로 시작
```

**[0]과 [1a]를 먼저 해야 합니다.** [1a]에서 분류기가 이미 95%+를 낸다면 reasoning 방식의 정당화가 §4의 3·4번으로 좁혀지고, 못 낸다면 그 자체가 reasoning 방식이 필요한 근거가 됩니다. **어느 쪽이 나와도 방향이 정해집니다.**

**자유 코드 생성(Thyme/PyVision) 경로는 이 스코프에서 시도할 이유가 없습니다.**

---

## 참고

- Seeing Straight: arxiv.org/abs/2511.04161
- Act Wisely / Metis: arxiv.org/abs/2604.08545 · github.com/Accio-Lab/Metis
- Beacon: arxiv.org/abs/2607.28595
- 나머지는 원 자료의 References 참조

*이 문서는 초록·검색 요약 기반이며 원문 대조 전입니다. 특히 Seeing Straight·ToolsRL·TIR-Bench는 본 과제와 직결되므로 **원문 확보가 필요합니다.***
