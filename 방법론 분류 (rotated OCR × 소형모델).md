## Visual Tool Reasoning 방법론 — rotated OCR × ≤3B 적합성 분류

> **설계 전제**
> ① **얼린 본 모델 + 독립 앞단** 구조. 앞단은 **약 3B VLM**을 학습시킨다
> ② 본 모델의 vision encoder가 **ViT 계열이 아니므로** 내부 feature에 의존하는 설계는 쓸 수 없다
> ③ 여러 본 모델로 **확장 가능**해야 하므로 앞단은 본 모델에 종속되지 않는다
> ④ 회전 각도는 **임의 각도(continuous)**를 목표로 한다
>
> 이 조건에서 visual tool reasoning 계열 방법론들이 쓸 만한지 판정합니다.

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

> **CodeVision(7B) Figure 16** — SFT cold start 없이 RL만 돌리자 **tool turn이 step 30에서 0으로 붕괴**하고 끝까지 0이었습니다. 저자 설명: *"code generation의 방대하고 비구조적인 action space 때문에 pure RL exploration으로는 유용한 정책을 못 찾는다."*
>
> **ReVPT(3B)도 독립적으로 같은 현상을 보고합니다** — R1-Zero 방식을 시도했다가 *"tool을 쓰려는 성향이 점진적으로 감소"*하는 걸 관찰하고 cold start를 도입했습니다. 이유는 *"**처리된 이미지로 추론하는 것이 모델 초기 학습 데이터로부터의 distribution shift**"*였다는 것 — **A-2(EquiAdapt)의 misalignment 문제와 같은 이야기**입니다.
>
> **7B와 3B 양쪽에서, 자유 코드 생성과 고정 tool 양쪽에서 확인된 현상입니다. cold start는 필수로 봐야 합니다.**

**→ 분류 기준 3가지**

| 기준 | 3B 친화 | 3B 위험 |
|---|---|---|
| **출력 형태** | 고정 tool + 구조화된 인자 | **자유 코드 생성** |
| **호출 길이** | 1~2턴 | 수십 턴 long-horizon |
| **학습** | SFT cold start **또는** tool-supervised RL(§3-B ②) | 맨바닥 RL-only 부트스트랩 — **7B(CodeVision)·3B(ReVPT) 양쪽에서 붕괴 확인** |

---

## 3. 분류 결과

### ✅ 적합 — 3B에서 바로 시도 가능

- **Seeing Straight** ★ *(분류 파이프라인)* — 문서 회전 → OCR을 그대로 다룬 **가장 가까운 선행 연구**로 **성능 참조점**이 된다. 단 ViT feature에 의존하고 12-class에 묶여 있어 **아키텍처는 차용 불가**(§1).
- **Adaptive-CoF** ★ *(구 Chain-of-Focus, v3에서 개명)* — **AGAR reward가 그룹 내에 직답 성공이 있을 때만 tool 사용을 깎는 구조**라, 정방향 샘플을 빼지 않고도 shortcut을 막고 abstain을 자연스럽게 학습시킬 수 있다.
- **OpenThinkIMG** ★ — **Qwen2-VL-2B에서 base 29.56 → SFT 45.67 → V-ToolRL 59.39**를 보인 유일한 소형 검증 사례이자, tool interface·궤적 생성·RL 환경이 갖춰진 오픈소스 인프라다.
- **Beacon** — *"어려운 예제에서 tool로 얻은 이득이 **쉬운 예제에서 생긴 손해로 대부분 상쇄된다**"*는 발견이 **정방향 문서에서의 성능 보존을 반드시 측정해야 한다**는 경고를 준다. gain을 reward로 쓰는 발상도 유효하되 base가 8B라 3B는 미검증.
- **ToolsRL** ★ — **rotate·flip이 이미 tool 목록에 있고**, 2단계 curriculum이 비싼 trajectory 없이 tool 사용법을 가르쳐 **3B의 cold start 부담을 덜어준다**.
- **ReVPT** ★ — **Qwen2.5-VL-3B에서 CV-Bench +9.03%**를 낸 3B 직접 검증이자, *"tool의 유용성은 모델 능력과 non-monotonic 관계이고 **3B 같은 자원 제약 모델에서 가장 크다**"*는 결론으로 우리 규모 선택을 뒷받침한다.

---

## 3-B. 주요 논문 확인 사항

혼동하기 쉬운 논문들의 실제 내용입니다.

| 논문 | 실제 내용 |
|---|---|
| **TIR-Bench** (2511.01833) | 13개 task 중 **"Rotated OCR"이 명시적으로 포함**. 22개 MLLM 평가에서 **최고 46%**. tool 있는 모델(o3·o4-mini·PyVision)이 크게 앞섬 |
| **ToolsRL** (2604.19945, CVPR'26) | *Visual Reasoning through Tool-supervised RL* (Amazon). tool에 **rotate·flip 명시 포함**. base Qwen2.5-VL-7B |
| **ReVPT** (2509.01656) | *Reinforced Visual Perception with Tools*. GRPO + 4개 tool(detection·zoom·edge·depth). base **Qwen2.5-VL-3B/7B-Instruct** |

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

**③ ReVPT — 3B에서 tool-use RL이 된다는 직접 증거**

**Qwen2.5-VL-3B**에서 CV-Bench +9.03%를 냅니다. CodeVision Fig 16의 붕괴와 상충하는 것처럼 보이지만 아닙니다 — **ReVPT는 고정 tool 4개**를 쓰고 CodeVision은 자유 코드 생성이었어요. **§2의 분류 기준(고정 tool은 3B 친화, 자유 코드는 위험)을 뒷받침합니다.** 학습 파이프라인은 아래에서 정리합니다.

### ★ 3B급 두 선례의 학습 파이프라인

우리가 실제로 따라 할 수 있는 유일한 소형 tool-RL 레시피 두 개입니다. **둘 다 "cold-start SFT → tool RL" 2단계로 동일**합니다.

| | **OpenThinkIMG** (2505.08617) | **ReVPT** (2509.01656) |
|---|---|---|
| **베이스 모델** | Qwen2-VL-2B-Instruct | Qwen2.5-VL-**3B**-Instruct / 7B-Instruct |
| **Stage 1 — cold start** | **Qwen2-VL-72B**로 tool 사용 궤적 합성 → 필터링 → SFT | **GPT-4.1**로 궤적 합성 → SFT |
| **Stage 2 — RL** | **V-ToolRL** (GRPO). tool 호출 결과 **이미지를 궤적에 되먹여** 최종 답 정확도로 최적화 | **GRPO**. 고정 tool 4개 위에서 답 정확도로 최적화 |
| **tool 구성** | POINT, DrawH/VLine, ZoomInSubplot, SegmentRegionAroundPoint (chart 특화). 각 tool을 **분산 서비스로 띄워 표준 interface로 통일** | object detection · zoom-in · edge detection · depth estimation (**4개로 의도적으로 축소**) |
| **성적** | 29.56 → SFT 45.67 → **59.39** (chart reasoning) | CV-Bench **3B +9.03%**, 7B +9.44% |

**① OpenThinkIMG의 핵심 아이디어 — "V", 즉 tool 출력 이미지를 다시 넣는 것**

같은 RL을 tool 출력을 **텍스트로만** 받아 돌리면 51.63, **이미지로 되먹이면** 59.39입니다. **차이 +7.76이 이 논문의 실질**이에요. 회전은 돌린 결과를 눈으로 확인해야 하는 과제라 **이 성분은 우리에게 필수**입니다.

**② ReVPT의 핵심 아이디어 — tool을 늘리지 않는 것**

저자들은 초기에 보조선·하이라이팅·**rotation**까지 넣었다가 활용률이 극히 낮아 **4개로 잘라냈습니다**. 이유는 *"소형 모델은 제한된 world knowledge 때문에 여러 tool을 동시에 학습하기 어렵다"*. 3B에서 성공한 유일한 사례가 **tool을 최소화한 사례**라는 점을 그대로 받아들여야 합니다.

**③ 왜 둘 다 cold start를 넣었나 — 같은 이유입니다**

ReVPT는 R1-Zero 방식을 먼저 시도했다가 **tool 사용 성향이 점진적으로 소멸**하는 걸 관찰하고 SFT를 도입했습니다. 저자 설명은 *"처리된 이미지로 추론하는 것이 모델의 초기 학습 데이터로부터의 distribution shift"*. **§2의 CodeVision(7B) 붕괴와 같은 현상이 3B에서 재현된 것**이고, 두 논문이 독립적으로 cold start를 필수로 결론지었습니다.

**④ 파라미터 학습 범위 — OpenThinkIMG는 vision encoder까지 전부 엽니다** *(저장소 확인)*

공개 저장소(`OpenThinkIMG/OpenThinkIMG`)의 학습 코드에는 `freeze`·`requires_grad`·`vision_tower` 관련 로직이 **하나도 없고**, 파라미터 선택을 TRL의 `get_peft_config(model_args)`에 위임합니다. README의 공식 실행 커맨드에도 `--use_peft`나 LoRA 인자가 없고 **DeepSpeed ZeRO-3 full-parameter** 세팅입니다(SFT `lr 2e-5`, RL `lr 1e-6`).
→ **ViT + merger + LLM 전체를 푸는 full fine-tuning**입니다. 소형 tool-RL에서 "encoder는 얼리고 LLM만 튜닝"하는 절약형 레시피의 검증된 선례는 없다는 뜻입니다.

> ⚠️ **재현 주의 2가지.** ⓐ 논문이 표기한 경로 `src/open_r1/*.py`는 저장소에 없고 실제 코드는 `r1_v/open_r1/`에 있습니다. ⓑ `sft.py`는 `Qwen2VLForConditionalGeneration` 로드 줄이 주석 처리되고 **`Qwen2_5_VLForConditionalGeneration`이 활성화**돼 있어, 본문의 Qwen2-VL-2B backbone과 공개 코드가 어긋납니다.

**⑤ 우리 과제와의 결정적 차이 — 둘 다 tool 호출을 *줄이는* 방향입니다**

OpenThinkIMG는 학습이 진행되며 tool 호출 비율이 **0.63 → 0.10~0.12로 급감**하고, ReVPT는 활용률 낮은 tool을 제거했습니다. 두 과제 모두 tool이 "가끔만 필요한" 성격이었기 때문입니다. **rotated OCR은 반대로 거의 항상 tool이 필요**하므로, 이 레시피를 그대로 쓰면 reward가 tool 사용을 억누르는 쪽으로 작동합니다 — reward 설계에서 반드시 통제해야 할 지점입니다.

---

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

설계 전제상 답이 세 가지로 정리됩니다.

| 근거 | 내용 |
|---|---|
| **① 적용 불가** | Seeing Straight은 본 모델의 **ViT CLS 토큰**이 전제. 본 과제의 본 모델은 ViT 계열이 아니라 **그 설계 자체를 쓸 수 없다** |
| **② 확장 불가** | 본 모델 feature에 배선하면 모델을 바꿀 때마다 재학습해야 한다. **독립 앞단만이 재사용 가능** |
| **③ 구조적 한계** | 12-class 분류는 **임의 각도를 표현할 수 없다.** 클래스를 늘려도(예: 360-way) 각도 간 순서·거리 정보가 사라진다 — Seeing Straight 저자도 arbitrary-angle을 **future work로 남겨뒀다** |

**①②는 "분류기가 나빠서"가 아니라 "우리 환경에 안 맞아서"입니다.** 논문에서는 이 구분을 명확히 해야 합니다 — Seeing Straight을 깎아내리는 게 아니라, **적용 조건이 다르다**고 써야 방어됩니다.

**③이 가장 강한 학술적 논거**입니다. 분류 프레임 자체의 한계이고, 선행 연구가 스스로 인정한 빈틈이니까요.

여기에 §1의 failure analysis가 보조 근거로 붙습니다 — 분류기가 실패하는 지점이 *"텍스트가 희소하거나 배치가 비정형일 때"*로 특정돼 있고, 그건 **전역 통계(CLS 평균)의 구조적 약점**입니다.

> **⚠️ 반대 증거도 알고 있어야 합니다.** Jigsaw-R1은 같은 성격(rule-based 정답이 있는 공간 과제)에서 *"명시적 reasoning 없이도 학습·일반화되며, 복잡한 추론 패턴은 emergent가 아니라 pre-existing"*이라고 보고합니다. 3B에서 reasoning 능력이 저절로 생기길 기대하면 안 됩니다.

---

## 5. 권장 실험 경로

```
[1] baseline — 임의 각도 회귀/분류 헤드
     └ Seeing Straight의 성능 참조점을 우리 조건(독립 모델·임의 각도)에서 재현
        ※ 본 모델 feature를 쓸 수 없으므로 앞단은 처음부터 독립 학습

[2] tool 사용법 부트스트랩   ← 맨바닥 RL은 3B에서 붕괴 (CodeVision 7B · ReVPT 3B)
     ├ (a) SFT cold start
     └ (b) ToolsRL Stage 1 — 정답 각도를 우리가 걸었으므로 per-tool reward가 공짜

[3] RL — reward에 OCR 결과를 직접 사용
     └ 회전 전후 OCR 정확도 차이 = tool-induced gain (Beacon식)

[4] 정방향 보존 검증
     └ 회전 불필요한 문서에서 성능이 떨어지지 않는가 (Beacon·A-2가 경고한 지점)
```

**[1]이 방향을 정합니다.** 임의 각도에서 단순 헤드가 어디까지 가는지 알아야 reasoning 방식의 이득을 주장할 수 있습니다.

### 임의 각도로 가면 달라지는 것 3가지

| | 4방향일 때 | 임의 각도일 때 |
|---|---|---|
| **reward** | ToolsRL의 이진 지시함수 `1[o=o*]` 로 충분 | **각도 오차 기반 연속 reward 필요.** 37°를 40°로 맞힌 것과 130°로 맞힌 건 다르게 평가돼야 함 |
| **resampling** | 90/180/270·flip은 **무손실** 픽셀 재배열 | **interpolation이 실제로 일어남** → VLM-RobustBench가 정량화한 기하 변형 손실(최대 34%p)이 실제 비용이 됨 |
| **평가셋** | ORB 4-class로 충분 | ORB **12-class**(30° 간격)가 최대 해상도. 그 사이 각도는 **직접 만들어야 함** |

**resampling 비용은 이제 반드시 측정해야 합니다.** 앞단의 순이득 = (방향 교정 이득) − (resampling 손실)이고, 임의 각도에서는 후자가 0이 아닙니다.

---

## 참고

- Seeing Straight: arxiv.org/abs/2511.04161
- Act Wisely / Metis: arxiv.org/abs/2604.08545 · github.com/Accio-Lab/Metis
- Beacon: arxiv.org/abs/2607.28595
- 나머지는 원 자료의 References 참조

*이 문서는 초록·검색 요약 기반이며 원문 대조 전입니다. 특히 Seeing Straight·ToolsRL·TIR-Bench는 본 과제와 직결되므로 **원문 확보가 필요합니다.***
