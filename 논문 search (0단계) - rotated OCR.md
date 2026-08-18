## rotated OCR × 소형모델 — 0단계 조사

> **스코프:** 얼린 본 모델(≤3B) 앞에 모듈을 두어 **회전된 문서의 OCR**을 살리는 것.
> `방법론 분류 (rotated OCR × 소형모델).md`에서 **✅ 적합** 판정된 논문만 추렸습니다.
> 각 논문을 **0단계**(기존 문제점 한 줄 + 핵심 아이디어 한 줄 + 쉬운 설명 한 문단 + 아이디어 씨앗)로 정리합니다.

**신뢰도 표기**
- `✅` **원문 PDF 대조 완료**
- `✔` 초록·저장소로 확인 (원문 미대조)
- `~` 2차 자료 기반, 미검증 — 채택 전 확인 필요

---

## 어떻게 나눴는지 (구축 순서 기준)

| 그룹 | 역할 | 논문 |
|---|---|---|
| **P** | **앞단 파이프라인** — 우리가 만들 것의 직접 선례 | Seeing Straight |
| **T** | **tool 구조** — 회전을 어떻게 tool로 붙일까 | Adaptive-CoF, OpenThinkIMG |
| **R** | **학습 방법** — 3B에서 어떻게 가르칠까 | ToolsRL, ReVPT, Beacon |

---

# P. 앞단 파이프라인

## P-1. Seeing Straight: Document Orientation Detection for Efficient OCR (arXiv 2511.04161v2) `✅` ★ **가장 중요**

- **기존 문제점:** 스캔·촬영 문서의 방향 보정은 실제 파이프라인의 필수 전처리인데, **회전된 문서에서 OCR 성능이 무너진다.**
- **핵심 아이디어:** 본 모델 앞에 **경량 회전 분류기**를 두되, **Phi-3.5-Vision의 vision encoder(CLIP ViT-L/14)를 재활용**하고 dynamic cropping으로 뽑은 **CLS 토큰들을 평균내 2층 FFN**에 넣어 **12-class(30° 간격) 분류**로 fine-tune한다. **약 304M, H100에서 0.5초.**

문서를 스캔하거나 사진으로 찍으면 방향이 틀어지는 일이 흔합니다. 사람은 종이를 돌려 보면 그만이지만 OCR 모델은 그걸 못 해요. 이 논문은 **"읽기 전에 먼저 똑바로 세우는 작은 모듈"**을 붙였습니다. 새 모델을 만든 게 아니라 **이미 있는 소형 VLM의 눈(vision encoder)을 빌려다** 분류 헤드만 얹은 겁니다.

**결과:** 회전 식별 12-class **98.00 / 96.71**, 4-class 96.81 / 92.68. 다운스트림 OCR은 docTR가 15.63 → **63.11**(≈4배), Tesseract가 24.06 → 48.99(≈2배). 부산물로 **ORB** 공개 — 1,863장(ORB-En 897 + **ORB-Indic 966**, 11개 언어).

**★ VLM은 문서에서도 방향을 못 맞힙니다.** 4-class(random 25%)에서 **Gemini-2.5 Pro 34.11**, Gemma-3 27B 30.57, GPT-4o 59.58. 텍스트가 가득한 문서인데도요.

**→ 아이디어 씨앗:**
1. **가장 먼저 재현해 성능 상한을 잡아야 할 baseline입니다.** 다만 분류기를 **비공개 문서 데이터셋**으로 학습시켰으므로 학습 데이터는 직접 구해야 합니다.
2. **★ 실패 지점이 특정돼 있습니다.** 오분류 원인이 *"텍스트가 문서의 대부분을 차지하지 않을 때"*, *"텍스트 배치가 비균일할 때"*, *"과도한 padding·중심 이탈"*입니다(ORB-Indic 45건에서 **90°↔270° 혼동** 관찰). **CLS 토큰 평균은 전역 통계라 구조적으로 여기에 약합니다** — reasoning 모델이 이길 자리이고, 실패 사례가 이미 알려져 있어 실험 설계가 쉽습니다.
3. **★ 저자가 future work로 "arbitrary-angle rotation"을 명시**했습니다. 12-class는 30° 단위라 그 사이 각도를 못 다룹니다.
4. **headroom 주의.** 회전에 원래 취약한 docTR·Tesseract에서 큰 개선이 나온 것이고, **Qwen2.5-VL은 68.16 → 72.26(정방향 72.62)로 하락폭이 4.5%p뿐**입니다. 저자 표현대로 *"coarse 회전에는 부분적 invariance가 있지만 fine-grained에서는 모든 파이프라인이 무너지므로"* **12-class 이상으로 가야 격차가 보입니다.**
5. ORB는 **디지털 회전**으로 만들어져 resampling artifact 통제가 안 돼 있습니다.

---

# T. tool 구조

## T-1. Adaptive Chain-of-Focus (arXiv 2505.15436v3) `✅` — ★ shortcut 문제의 해답

> **주의: v3에서 제목과 초점이 바뀌었습니다.** *"**Adaptive** Chain-of-Focus Reasoning via Dynamic Visual Search and Zooming for **Efficient** VLMs"*. 방법 이름도 **Adaptive-CoF**입니다.

- **기존 문제점:** 능동적 zoom은 성능을 올리지만 **연산 비용이 크고 전역 이해를 해칠 수 있다.**
- **핵심 아이디어:** **필요할 때만** 검색·확대하도록 가르친다. SFT(visual search agent가 만든 궤적) → RL(**AGAR**).

**★ AGAR가 우리 문제에 직접 쓰입니다.**

```
r_i = c_i·[ d_i·1 + z_i·(1 − δ·g) ] + (1 − c_i)·(γ·f_i)
      c: 정답 여부   d: 직답   z: zoom 경로   f: 형식 유효
      g: 그룹 G개 rollout 중 "직답으로 맞힌 게 하나라도 있으면" 1
      δ = 0.2 (페널티)   γ = 0.1 (오답이어도 형식 맞으면 주는 보너스)
```

읽는 법 — **직답으로 맞히면 항상 최대 보상 1.** tool을 쓴 경로는 **그룹에 직답 성공이 있을 때만(g=1) 0.8로 깎입니다.** 직답이 아무도 못 맞히면(g=0) tool 경로도 만점입니다.

**결과:** V\* Bench 71.2 → **90.1 (+18.9)**, MME-RealWorld-Lite 42.3 → 50.9 (+8.6). 그러면서 **zoom 호출 75% 감소, 토큰 약 50% 감소.** base는 Qwen2.5-VL-7B.

**→ 아이디어 씨앗:**
1. **★ AGAR가 R-1(ToolsRL)이 지적한 shortcut 문제의 해답입니다.** ToolsRL은 정방향 샘플을 *빼서* shortcut을 막았는데, AGAR는 **빼지 않고도 막습니다** — 회전된 이미지는 직답이 실패하니 g=0이 되어 tool 사용이 만점을 받고, 정방향 이미지는 직답이 통하니 g=1이 되어 tool 사용이 깎입니다. **abstain을 별도 장치 없이 자연스럽게 학습**시킬 수 있습니다.
2. **단, 방향이 반대일 수 있습니다.** 이 논문의 목표는 tool 호출을 *줄이는* 것(75% 감소)인데, rotated OCR은 대부분 회전이 필요합니다. **δ를 낮추거나 부호를 재검토**해야 합니다.
3. visual token을 SFT·RL loss에서 **mask out**하는 처리가 학습 안정성에 중요하다고 서술합니다.
4. base가 7B 하나뿐이라 **3B 재현은 미검증**입니다.

## T-2. OpenThinkIMG (arXiv 2505.08617v2) `✅` — ★ **2B 검증 사례**

- **기존 문제점:** 시각 tool RL을 하려면 **tool interface·궤적 생성·학습 환경을 밑바닥부터** 만들어야 하고, 정적 시범에 대한 SFT만으로는 **동적 tool 호출에 일반화가 안 된다.**
- **핵심 아이디어:** 표준 tool interface + 궤적 생성 + 학습 환경을 오픈소스로 제공하고, 그 위에 **V-ToolRL**(tool 상호작용 피드백으로 직접 최적화)을 얹는다.

**★ 우리 제약에 가장 직접적인 증거입니다 — base가 QWEN2-VL-2B입니다.**

| 단계 | 정확도 (chart reasoning) |
|---|---|
| Qwen2-VL-2B base | 29.56 |
| + SFT (cold start) | **45.67** |
| + text-based RL | 51.63 |
| **+ V-ToolRL (full)** | **59.39** |

**2B로 GPT-4.1을 +8.68 앞섭니다.** TACO·CogCoM 같은 supervised tool-learning baseline 대비 평균 +12.7.

**→ 아이디어 씨앗:**
1. **★ 2B에서 "SFT cold start → tool RL" 파이프라인 전체가 작동한다는 직접 증거입니다.** ReVPT의 "2B 스케일 향상"보다 훨씬 구체적이에요.
2. **cold start만으로는 부족합니다** — SFT 45.67에서 RL로 59.39까지 **+13.72**가 더 나옵니다.
3. **"V"가 핵심입니다** — tool 출력 이미지를 실제로 다시 넣는 V-ToolRL이 text-only RL보다 **+7.76**. 회전은 돌린 이미지를 봐야 하므로 이 성분이 필수입니다.
4. **tool 목록에 rotate가 없습니다** (POINT, DrawH/VLine, ZoomInSubplot, SegmentRegionAroundPoint — chart 특화). **인프라만 가져오고 tool은 새로 넣어야** 합니다.
5. **⚠️ 학습이 진행되면 tool 호출이 0.63 → 0.10~0.12로 급감**합니다. chart에서는 적절하지만 **rotated OCR에서는 해로울 수 있는 방향**이라 reward 설계에서 통제가 필요합니다.
6. 궤적 생성에 **Qwen2-VL-72B**를 씁니다 — 데이터 준비에 대형 모델이 필요합니다.

---

# R. 학습 방법

## R-1. ToolsRL / Visual Reasoning through Tool-supervised RL (arXiv 2604.19945, CVPR 2026) `✅` ★

- **기존 문제점:** tool 사용법과 답 정확도를 **한꺼번에 학습시키면 불안정**하고, 좋은 tool-use trajectory를 만드는 게 **비싸다.**
- **핵심 아이디어:** **2단계 curriculum으로 분리**한다 — Stage 1에서 정답 기반 **per-tool reward**로 tool 사용법만 익히고, Stage 2에서 GRPO로 답 정확도를 최적화한다.

tool을 쓰는 법과 문제를 푸는 법을 동시에 가르치면 둘 다 어설퍼집니다. 이 논문은 **먼저 도구 쓰는 법만 확실히 가르치고, 그다음에 문제 풀이를 붙입니다.** 그리고 1단계의 정답은 데이터를 만들 때 이미 알고 있으므로 **사람이 만든 시범 궤적이 필요 없습니다.**

tool 목록에 **rotate·flip이 이미 포함**돼 있고, base는 Qwen2.5-VL-7B입니다.

**회전 reward가 이진 지시함수 하나입니다:** `R_rotflip(s_t) = 1[o(I_t) = o*]`. 현재 이미지 방향이 목표와 같으면 1. 끝입니다.

**→ 아이디어 씨앗:**
1. **회전은 각도를 직접 걸어서 데이터를 만들기 때문에 per-tool reward가 공짜입니다.** 3B에서 SFT cold start가 부담이라면 이게 더 싼 대안입니다.
2. **★ 단계 분리가 필수입니다.** ablation에서 tool reward와 accuracy reward를 **동시에** 최적화하면 **58.1로, accuracy 단독(62.6)보다 나빴습니다.** 2단계 curriculum은 77.3입니다.
3. **★ 함정 — Stage 1에는 회전된 샘플만 넣어야 합니다.** 정방향 문서를 섞으면 모델이 *"그냥 정방향이라 가정하고 답하기"* shortcut을 배웁니다. **이건 abstain 학습과 충돌하므로**, Stage 1은 회전만 / Stage 2에서 정방향을 섞는 설계가 필요합니다.
4. base가 Qwen2.5-VL-7B 하나뿐이라 **3B 재현은 미검증**입니다.

## R-2. ReVPT / Reinforced Visual Perception with Tools (arXiv 2509.01656) `✅` ★ **3B 검증**

> **정정:** 이전에 *"2B 스케일에서 큰 향상"*이라고 적었는데 틀렸습니다. 실제로는 **ReVPT-3B / ReVPT-7B**입니다 — 오히려 우리 규모에 정확히 맞습니다.

- **기존 문제점:** SFT로 시각 tool을 붙이면 **데이터 생성이 비싸고, 세심한 필터링에 의존하며, 일반화가 나쁘다.**
- **핵심 아이디어:** GRPO 기반 RL로 **4개 고정 tool**(object detection · zoom-in · edge detection · depth estimation) 사용을 학습시킨다.

**성능:** CV-Bench에서 instruct 대비 **ReVPT-3B +9.03%**, ReVPT-7B +9.44%. base는 **Qwen2.5-VL-3B-Instruct / 7B-Instruct**. 3B에서 SFT·text-only GRPO baseline을 모두 앞섭니다.

**→ 아이디어 씨앗:**

1. **★★ cold start 없는 RL은 3B에서도 무너집니다.** 저자들은 처음에 R1-Zero 방식을 시도했다가 **"tool을 쓰려는 성향이 점진적으로 감소"**하는 걸 관찰하고 cold start를 도입했습니다. **CodeVision Figure 16(7B)의 붕괴를 3B에서 독립적으로 재현**한 셈이에요.
   - 저자 설명이 중요합니다 — *"시각 과제를 푸는 데 tool이 본질적으로 필요하지 않았고, **처리된 이미지로 추론하는 것이 모델의 초기 학습 데이터로부터의 distribution shift**였기 때문"*.
   - **이건 A-2(EquiAdapt)의 misalignment 문제와 같은 이야기입니다** — 앞단이 만든 이미지가 본 모델에게 낯설다는 것. 계열이 다른 두 논문이 같은 현상을 지적합니다.
   - cold-start 데이터는 **GPT-4.1**로 합성했습니다.

2. **★★ 저자들이 rotation tool을 실제로 시도했다가 뺐습니다.**
   > *"초기에는 보조선 그리기, 하이라이팅, **rotation** 등 더 넓은 tool을 포함했으나 **극히 낮은 활용률**을 발견했다. **소형 모델은 제한된 world knowledge 때문에 여러 tool을 동시에 학습하기 어렵다**는 점을 보여준다."*

   **맥락을 정확히 봐야 합니다.** ① **여러 tool을 동시에** 학습시켰고 ② 그들의 과제(SAT·CV-Bench·BLINK·MMStar)는 **회전을 요구하지 않습니다.** 모델이 rotation을 쓸 이유가 없었어요.
   → 우리는 **tool이 1~2개이고 과제가 회전을 반드시 요구**합니다. 조건이 정반대라 이 결과가 우리에게 그대로 적용되진 않지만, **"tool을 늘리지 말 것"**이라는 강한 교훈은 남습니다.

3. **★ 3B가 tool이 가장 유용한 구간입니다.** 저자들의 결론 — *"tool specialist의 유용성은 모델 능력과 **non-monotonic 관계**다. **3B 같은 자원 제약 모델에서는 tool이 지각 향상의 강력한 지름길**(9%+ 향상)이지만, 중간 규모로 가면 **더 큰 vision encoder가 외부 tool 의존도를 낮춰 marginal benefit이 감소**한다."*
   → **우리의 3B 선택을 정당화하는 근거**입니다.

4. **tool 출력이 틀릴 수 있다는 한계** — object detection이 매트리스를 베개로 오인하는 등, 외부 specialist의 오류가 본 모델을 오염시킵니다. **단 회전 tool은 결정적 연산이라 이 문제가 없습니다** — 우리 설계의 이점입니다.

## R-3. Beacon: Knowing When and How to Perform Agentic Visual Reasoning (arXiv 2607.28595) `✅`

- **기존 문제점:** tool을 "불렀나"로만 보면 **실제 효과와 무관**해진다.
- **핵심 아이디어:** tool 사용을 두 축으로 분해한다 — **Mode Adaptiveness**(tool이 정말 필요한지 인식하는가)와 **Tool Effect**(실제로 능력을 확장했는가).

**★ 핵심 발견이 우리에게 직접 경고입니다.**

> *"어려운 예제에서 tool 사용으로 얻은 이득이, **이미 풀 수 있던 쉬운 예제에서 생긴 손해로 대부분 상쇄된다**."*

**A-2가 CIFAR10에서 관측한 것(96.97 → 93.29)과 같은 현상**입니다. 계열이 완전히 다른 두 논문이 같은 결론에 도달했어요.

메커니즘은 **Necessity-Aware Adaptive Reward(NAAR)** + **Hint-Guided Capability Expansion**이고, forgetting 완화를 위해 **순수 텍스트 궤적을 일부 주입**합니다. base는 **Qwen3-VL-8B-Instruct**(3B 아님).

**→ 아이디어 씨앗:**
1. **"쉬운 케이스에서의 손해"를 반드시 측정해야 합니다.** 우리 경우 = **정방향 문서에서 앞단이 성능을 떨어뜨리지 않는가.** 논문에 이 수치가 없으면 심사에서 바로 지적받습니다.
2. tool-induced gain을 학습 신호로 쓰는 발상은 유효하지만, **rotated OCR에서는 teacher 없이 회전 전후 OCR 정확도 차이로 직접 잴 수 있습니다.**
3. **forgetting 완화용 텍스트 궤적 주입**은 값싼 실무 기법이라 그대로 차용할 만합니다.

---

# 평가셋

| | 규모 | 용도 |
|---|---|---|
| **ORB** (P-1 부속) `✅` | 1,863장 | **주 평가.** rotated OCR 전용 |
| **TIR-Bench** (2511.01833) `✅` | 13 task 중 1개 | **난이도 확인.** 최고 46%로 미포화. Rotated OCR task 보유 |
| 변형 OCRBench / ChartQAPro | 1,000 + 1,948문항 ×6조건 | **비교 축.** CodeVision 수치와 직접 대조 |
| OCR-Robust (2606.26041) `✔` | 812문항 | **지표 틀만** (RCR·WCR·CRI) |

상세는 `벤치마크 분석.md` §6·부록 2 참조.

**TIR-Bench의 fine-tuning 비교 실험** — Qwen2.5-VL-7B를 rotated OCR로 학습시킨 결과입니다.

| 데이터 | Direct SFT | Tool-use SFT |
|---|---|---|
| 1k | ~0.44 | ~0.735 |
| 15k | ~0.445 | ~0.848 |

**Direct SFT는 데이터를 15배로 늘려도 0.44에서 평평합니다.** 반면 회전 각도를 먼저 출력하고 복원된 이미지를 읽는 agentic 방식은 계속 오릅니다. **"본 모델을 직접 학습시키는 것"에 대한 가장 강한 반증**입니다.

> ⚠️ 저자들은 그 원인을 catastrophic forgetting으로 **추정**하지만(*"may cause forgetting"*), **측정한 실험은 없습니다.** 인용할 때는 **scaling 결과만** 쓰고 forgetting은 "저자들의 설명"으로만 언급하세요.

---

# 읽는 순서

1. **P-1 Seeing Straight** — 만들 것의 직접 선례이자 넘어야 할 baseline. **여기서 시작**
2. **T-2 OpenThinkIMG** — **2B에서 전 파이프라인이 작동한다는 증거.** 실현 가능성이 여기서 결정됨
3. **R-1 ToolsRL** — rotate reward 설계와 2단계 curriculum
4. **T-1 Adaptive-CoF** — AGAR로 shortcut·abstain을 한 번에 처리하는 법
5. **R-2 ReVPT / R-3 Beacon** — reward 정교화 단계에서

---

# 구버전 파일에서 이어받을 것

`[구버전] 논문 search (0단계).md` / `(1단계).md`가 폐기된 건 아닙니다. 스코프가 넓었을 뿐, 아래는 **여전히 유효**합니다.

| 항목 | 어디에 | 왜 여전히 필요한가 |
|---|---|---|
| **A그룹 (canonicalization)** | 구 0단계 A-1~A-5 | **Seeing Straight과 같은 계열**의 이론적 배경. 특히 **A-2의 identity prior**(*"이미 정방향이면 '회전 불필요'라고 답하라"*)와 **A-4의 실물 촬영 검증**은 그대로 쓸 수 있음 |
| **CodeVision 대비 포지셔닝** | 구 1단계 B-1 | 경쟁 논문. **Rot90 +2.1 / Rot270 +3.5**라는 분해 수치는 우리 motivation의 핵심 |
| **Thinking with Images taxonomy** | 구 1단계 B-0 | related work 골격. engine/machine 어휘 |
| **RotBench 진단** | 구 0단계 전제 | 보조 근거로만. *"아무도 90°/270°를 구분 못 한다"*는 **자연 이미지** 기준이라, 문서 도메인에서는 **P-1(Seeing Straight)의 직접 측정치**(Gemini-2.5 Pro 4-class 34.11%)를 쓰는 게 강합니다 |

**폐기된 판단:**
- ~~closed-source 적용 가능성이 최대 강점~~ → 본 모델 feature를 쓰기로 하면 성립 안 함
- ~~abstain이 핵심 빈 자리~~ → B-0 §8.5.2에 부분 선점됐고, 문서 스코프에서는 비중이 작음

---

*관련 파일: `방법론 분류 (rotated OCR × 소형모델).md` (적합성 판정 근거) · `벤치마크 분석.md` (평가셋) · `논문 비교 (A-1 vs A-2 vs A-4).md` (앞단 계열 이론)*
