## Visual Tool Reasoning 방법론 — rotated OCR × ≤3B 적합성 분류

> **과제 전제:** ① 회전된 문서의 OCR을 살리는 것 ② 학습 대상은 **3B 이하 소형 모델** ③ 얼린 본 모델의 **내부 feature 접근 가능**
>
> visual tool reasoning 계열 방법론들이 이 조건에서 쓸 만한지 판정합니다. 원문 미확인(초록·검색 요약 기반)이므로 채택 전 대조가 필요합니다.

---

## 1. ★ 가장 중요한 논문

### Seeing Straight: Document Orientation Detection for Efficient OCR (arXiv 2511.04161v2)

Goswami 외 (OLA Electric · Krutrim AI). **원문 대조 완료.**

**구조 — 놀랍도록 단순합니다.**

| 항목 | 내용 |
|---|---|
| 백본 | **Phi-3.5-Vision의 vision encoder** (= CLIP ViT-L/14), standalone fine-tune |
| 입력 처리 | **dynamic cropping** — 이미지를 K개 crop으로 나눠 각각 encoder 통과 |
| 특징 추출 | 각 crop의 **CLS 토큰을 뽑아 K개 평균** |
| 분류 헤드 | **2층 FFN** (GELU, hidden = D/2, dropout 0.2) + softmax CE |
| 클래스 | **12-class = 30° 간격**(0°,30°,…,330°). 4-class는 C만 4로 바꿈 |
| 규모 | **약 304M 파라미터, H100에서 0.5초** |

**성능**

| | ORB-En | ORB-Indic |
|---|---|---|
| 12-class | **98.00** | **96.71** |
| 4-class | 96.81 | 92.68 |

**★ VLM은 문서에서도 방향을 못 맞힙니다** (이게 우리 motivation의 핵심)

| 모델 | 4-class (random 25%) | 12-class (random 8.3%) |
|---|---|---|
| **Ours** | **96.81** | **98.00** |
| LLaMA-4 Maverick 17B | 65.49 | 32.00 |
| GPT-4o | 59.58 | 34.00 |
| Gemini-2.5 Flash | 39.55 | 22.00 |
| **Gemini-2.5 Pro** | **34.11** | 24.00 |
| Gemma-3 27B | 30.57 | 16.00 |

**Gemini-2.5 Pro가 4지선다에서 34%입니다.** 텍스트가 가득한 문서인데도요.

**다운스트림 OCR** (SROIE field-level, 4-class)

| 모델 | 정방향 | 회전 | **보정 후** |
|---|---|---|---|
| docTR | 64.54 | **15.63** | **63.11** (≈4배) |
| Tesseract | 49.06 | 24.06 | 48.99 (≈2배) |
| Gemini-2.5 Pro | 62.82 | 46.61 | 60.66 |
| **Qwen2.5-VL** | 72.62 | **68.16** | 72.26 |
| **Gemini-2.5 Flash** | 70.10 | **69.67** | 70.10 |

**보정 후 성능이 정방향과 거의 같아집니다.** 다만 **Qwen2.5-VL과 Gemini-2.5 Flash는 애초에 별로 안 떨어집니다** — 이미 부분적 rotation robustness가 있다는 뜻이라, 이 모델 계열을 쓰면 4-class 헤드룸이 작습니다.

> 저자 결론: *"여러 모델이 coarse(4-class) 회전에는 부분적 invariance를 보이지만, **fine-grained 방향 변동에서는 모든 파이프라인이 일관되게 무너진다.**"* → **12-class 이상이 실제 격차가 있는 구간입니다.**

**Ablation** (4-class): 단층 헤드+cropping 90.66 / 다층 헤드+cropping없음 92.65 / **둘 다 96.81** — 두 요소가 상보적.

**★★ Failure Analysis — reasoning 방식을 정당화할 가장 좋은 근거**

저자들이 오분류를 직접 분석했습니다.

- **ORB-Indic 45건** (타밀·힌디·텔루구 위주): **90°↔270°, 0°↔180°, 240°↔120° 혼동**
  - 원인: *"텍스트가 문서의 대부분을 차지하지 않을 때"*, *"텍스트 배치가 비균일할 때"*
- ORB-En-SROIE 2건 · FUNSD 1건: *"텍스트 영역 주변의 과도한 padding"*, *"중심에서 벗어난 텍스트"*
- **SynthDog 25건**: 합성 문서의 레이아웃 변동성에 민감

**실패 지점이 정확히 "텍스트가 희소하거나 배치가 비정형일 때"입니다.** CLS 토큰 평균은 전역 통계라 이런 경우에 약할 수밖에 없어요. **레이아웃 의미를 읽는 reasoning 모델이 이길 수 있는 자리**입니다.

**저자가 명시한 future work:** *"arbitrary-angle rotation"* — 임의 각도는 스스로 미해결로 남겨뒀습니다.

**재현 시 장애물:** 분류기를 **비공개 문서 데이터셋**으로 학습시켰습니다. ORB는 평가용이라 학습 데이터는 직접 구해야 합니다.

**ORB 구성**

| | 규모 | 출처 |
|---|---|---|
| ORB-En | 897 | SROIE(347) + SynthDog(500) + FUNSD(50) |
| ORB-Indic | 966 | Wikisource **Level-4 검증 페이지**, 11개 언어 |

⚠️ 이미지를 **디지털 회전**으로 만들었습니다(A-4처럼 실물 촬영이 아님) → resampling artifact 통제는 안 돼 있습니다.

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
- **Adaptive-CoF** ★ *(구 Chain-of-Focus, v3에서 개명)* — **AGAR reward가 그룹 내에 직답 성공이 있을 때만 tool 사용을 깎는 구조**라, 정방향 샘플을 빼지 않고도 shortcut을 막고 abstain을 자연스럽게 학습시킬 수 있다.
- **OpenThinkIMG** ★ — **Qwen2-VL-2B에서 base 29.56 → SFT 45.67 → V-ToolRL 59.39**를 보인 유일한 소형 검증 사례이자, tool interface·궤적 생성·RL 환경이 갖춰진 오픈소스 인프라다.
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

**① TIR-Bench — agentic SFT가 direct SFT를 압도한다** *(원문 대조 완료)*

Rotated OCR을 사례로 두 학습 전략을 비교했습니다. OCRDataset에서 1k/5k/10k/15k 샘플을 회전시켜 **Qwen2.5-VL-7B를 전체 파라미터 5 epoch fine-tune**.

- **Direct SFT**: 회전 이미지 → 정답 텍스트 직접 매핑
- **Tool-Use SFT**: 먼저 회전 각도를 출력 → 복원된 이미지를 원 context와 concat → 그 위에서 텍스트 읽기

**Figure 6b 판독 결과:**

| 데이터 | Direct SFT | Tool-use SFT |
|---|---|---|
| 1k | ~0.44 | ~0.735 |
| 5k | ~0.44 | ~0.825 |
| 10k | ~0.44 | ~0.85 |
| 15k | ~0.445 | ~0.848 |

**Direct SFT는 데이터를 15배로 늘려도 0.44에서 평평합니다.** 저자 표현으로 *"단순히 데이터를 늘리는 것은 image-based reasoning이 필요한 과제에서 효과가 없다."*

Loss 곡선(15k)도 극적입니다 — Tool-use SFT는 0.78에서 시작해 **약 20 iteration 만에 0.02로 급락**, Direct SFT는 0.55에서 250 iteration에 걸쳐 계단식으로 내려갑니다.

> **⚠️ 다만 catastrophic forgetting은 저자의 *가설*입니다.** 원문은 *"fine-tuning it directly on rotated data **may** cause forgetting"*, *"**which may explain** the faster loss reduction"*으로 두 번 hedge하고, **다른 과제에서의 성능 저하를 측정한 실험은 없습니다.**
>
> **인용할 때는 scaling 결과를 쓰세요** — 그건 측정된 사실입니다. forgetting은 "저자들이 제시한 설명"으로만 언급해야 합니다.

**② ToolsRL — rotate reward가 놀랍도록 단순하다** *(원문 대조 완료)*

회전/뒤집기의 per-state reward는 **이진 지시함수 하나**입니다.

```
R_rotflip(s_t) = 1[ o(I_t) = o* ]  ∈ {0, 1}
```

현재 이미지의 방향 `o(I_t)`가 목표 방향 `o*`와 같으면 1. 끝입니다. *(비교: zoom-in은 ModF1에 w_fp=0.1, w_fn=1.0 비대칭 가중을 줘야 했습니다.)*

Stage 1의 최종 reward는 두 성분의 평균입니다:

```
R_stage1 = ½( R_global + R_answer ) + R_format
  R_global = max_t R_task(s_t)          # 전 궤적 최대 → 탐색 장려
  R_answer = R_task(s_answer)           # <answer>가 참조한 상태만 → 관련성 확보
```

global만 쓰면 무관한 step도 보상받고, answer만 쓰면 탐색이 죽습니다. 둘을 섞어 균형을 잡습니다.

**★ Ablation이 결정적입니다** (DocVQA-RF 기준):

| 방식 | 정확도 |
|---|---|
| Accuracy reward만 | 62.6 |
| **Tool-Sup + Accuracy를 *동시에*** | **58.1** ⚠️ |
| Global tool-sup만 | 60.3 |
| Answer tool-sup만 | 65.4 |
| **ToolsRL (2단계 curriculum)** | **77.3** |

**tool reward와 accuracy reward를 동시에 최적화하면 accuracy 단독보다 오히려 나쁩니다(58.1 < 62.6).** 저자가 말한 "optimization conflict"의 실증이고, **단계를 나누는 것이 선택이 아니라 필수**라는 뜻입니다.

> **★★ 그리고 회전 학습에 결정적인 함정이 하나 있습니다 (부록 D.3).**
>
> Stage 1에는 **회전/뒤집힌 샘플만** 넣어야 합니다. 정방향 문서를 섞으면 모델이 **"그냥 정방향이라 가정하고 바로 답하기"라는 shortcut**을 배웁니다 — 정방향 subset에서는 그게 통하니까요. 저자들은 이걸 reward hacking으로 분류하고, **회전 샘플만으로 제한해 active perception을 강제**했습니다.
>
> **이건 abstain과 정면으로 충돌합니다.** "돌릴 필요 없으면 안 돌린다"를 배우려면 정방향 샘플이 필요한데, 그걸 넣으면 shortcut이 생깁니다. → **Stage 1은 회전만, Stage 2에서 정방향을 섞는 분리 설계**가 필요해 보입니다.

*(base는 Qwen2.5-VL-7B 하나뿐입니다. 3B 실험은 없습니다.)*

**③ ReVPT — 소형에서 tool-use RL이 된다는 증거**

**"2B 스케일에서 특히 큰 향상"**을 보고합니다. CodeVision Fig 16의 붕괴와 상충하는 것처럼 보이지만 아닙니다 — **ReVPT는 고정 tool 4개**를 쓰고 CodeVision은 자유 코드 생성이었어요. **§2의 분류 기준(고정 tool은 3B 친화, 자유 코드는 위험)을 오히려 뒷받침합니다.** *(원문 미대조)*

*(분류 결과는 §3 참조.)*

---

## 4. 스코프를 OCR로 좁힐 때의 득실 ★

> **정정:** 이전 판에서 *"OCR로 좁히면 문제가 쉬워진다"*고 적었는데, **원문 대조 결과 틀렸습니다.** 아래로 대체합니다.

### 얻는 것 — 모호성이 사라진다

- **정답이 잘 정의됩니다.** 하늘·텍스처와 달리 텍스트 문서는 정방향이 유일하게 정해져요. 그래서 **abstain 문제의 비중이 크게 줄어듭니다.**
- **검증 신호가 공짜입니다.** 회전을 제대로 맞히면 OCR 결과가 급격히 좋아지므로, reward도 평가도 여기서 나옵니다.
- **실용적 가치가 명확합니다.** 스캔·촬영 문서의 방향 보정은 실제 파이프라인의 표준 전처리입니다.

### 잃지 않는 것 — 난이도는 그대로다

**"텍스트가 있으면 읽기 방향으로 90°/270°가 구분되니 쉬워진다"는 추론은 VLM에게는 성립하지 않습니다.** Seeing Straight Table 7이 반례입니다.

| 모델 | 4-class 문서 회전 (random 25%) |
|---|---|
| Gemini-2.5 Pro | **34.11** |
| Gemma-3 27B | 30.57 |
| GPT-4o | 59.58 |

**텍스트가 가득한 문서인데도 Gemini-2.5 Pro가 4지선다에서 34%입니다.** 그리고 Seeing Straight의 failure analysis에서 **90°↔270° 혼동이 실제로 관찰**됩니다(ORB-Indic 45건).

**→ 그러니 motivation을 포기할 필요가 없습니다.** 다만 **출처를 RotBench에서 Seeing Straight으로 바꾸면 됩니다.** 문서 도메인에서 직접 측정된 수치라 오히려 더 강합니다.

### 정말로 주의할 것 — headroom

문제는 난이도가 아니라 **개선 여지**입니다.

| | 정방향 | 회전 | 하락폭 |
|---|---|---|---|
| docTR | 64.54 | 15.63 | **−48.9** |
| **Qwen2.5-VL** | 72.62 | **68.16** | **−4.5** |
| Gemini-2.5 Flash | 70.10 | 69.67 | −0.4 |

**Qwen 계열을 본 모델로 쓰면 4-class에서 헤드룸이 4.5%p뿐입니다.** 큰 개선 폭은 회전에 취약한 전통 OCR 엔진(docTR·Tesseract)에서 나온 숫자예요.

**→ 대응 두 가지:**
1. **fine-grained(12-class 이상)로 갈 것.** 저자들도 *"coarse 회전에는 부분적 invariance가 있지만 fine-grained에서는 모든 파이프라인이 무너진다"*고 명시합니다.
2. **본 모델을 신중히 고를 것.** 이미 rotation-robust한 모델을 쓰면 개선을 보여주기 어렵습니다.

### 그리고 "왜 분류기가 아니라 reasoning인가"

**⚠️ 이 질문이 무겁습니다. 독립된 두 증거가 "reasoning이 필요 없을 수도 있다"를 가리킵니다.**

| 출처 | 내용 |
|---|---|
| **Seeing Straight** | 문서 회전을 **304M 분류기만으로 98%**, H100에서 0.5초 |
| **Jigsaw-R1** | 같은 성격(rule-based 정답이 있는 공간 과제)에서 *"**명시적 reasoning 없이도** 학습·일반화된다"*, *"복잡한 추론 패턴은 **emergent가 아니라 pre-existing**"* |

Jigsaw-R1의 두 번째 결론이 특히 3B에 불리합니다 — 추론 패턴이 학습으로 **생겨나는** 게 아니라 **이미 있던 게 드러나는** 것이라면, 그게 약한 3B에서는 RL을 돌려도 안 나타날 수 있습니다.

**→ 그러니 "정확도가 더 높다"로는 안 됩니다.** 다른 축에서 이유를 대야 하고, **원문 대조로 후보가 좁혀졌습니다:**

1. **★ 분류기가 실패하는 지점을 공략** — Seeing Straight의 오분류 원인이 *"텍스트가 문서의 대부분을 차지하지 않을 때"*, *"텍스트 배치가 비균일할 때"*, *"과도한 padding·중심 이탈"*입니다. **CLS 토큰 평균은 전역 통계라 구조적으로 여기에 약합니다.** 레이아웃을 의미적으로 읽는 모델이라면 이길 수 있고, **실패 사례가 이미 특정돼 있어 실험 설계가 쉽습니다.**
2. **★ 임의 각도(continuous rotation)** — Seeing Straight이 **future work로 명시**한 미해결 항목입니다. 12-class는 30° 단위라 그 사이 각도는 못 다룹니다.
3. **다중 tool** — 회전 + crop이 함께 필요한 경우. 분류기는 회전밖에 못 합니다.
4. 설명 가능성 — *"그래서 성능이 오르나?"*에 약합니다. 단독 논거로는 부족.

**1번과 2번이 가장 강합니다.** 둘 다 **선행 연구가 스스로 남긴 빈틈**이라 방어하기 좋습니다.

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
