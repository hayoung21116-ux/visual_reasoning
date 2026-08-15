## 1단계 정리 — 핵심 실험 세팅·결과

> 0단계에서 추린 논문 중 **더 깊이 봐야 할 것만** 골라 1단계(실험 세팅·결과)로 내려 정리합니다.
> 0단계 파일: `논문 search (0단계).md`

**표기 규칙**
- `✅` 2차 출처(논문 소개 페이지·리뷰·저자 공개 요약)에서 확인한 내용
- `❓` 아직 원문 대조가 안 된 것 — 인용 전에 반드시 PDF에서 확인
- 기술 용어는 영어 원어 그대로 씁니다.

**수록 현황**
- [x] B-1. CodeVision
- [ ] A-2. EquiAdapt
- [ ] B-3. DeepEyes
- [ ] B-4. AdaTooler-V
- [ ] (필요할 때 추가)

---

# B-1. CodeVision

## 서지 정보

| 항목 | 내용 |
|---|---|
| 정식 제목 | *Thinking with Programming Vision: Towards a Unified View for Thinking with Images* |
| 방법 이름 | **CodeVision** (논문 제목엔 안 들어가고 method name으로 등장) |
| 학회 | CVPR 2026 |
| arXiv | 2512.03746 (2025-12-03) |
| 소속 | ByteDance (BandAI) + ZJU |
| 제1저자 | Guo (CVPR proceedings 파일명 기준) `❓` 전체 저자 목록 미확인 |
| 코드 | github.com/ByteDance-BandAI/CodeVision |

**0단계 한 줄 복습:** 고정된 tool registry를 버리고, 모델이 **코드를 생성해 임의의 image operation을 호출**하게 한다(code-as-tool).

---

## 1. 이 논문이 실제로 측정하는 것

이 논문의 실험은 두 축으로 갈립니다. 우리 과제에 중요한 건 **①번 축**입니다.

1. **Orientation robustness** — 입력 이미지가 회전·반전됐을 때 성능이 얼마나 유지되는가
2. **Multi-tool reasoning** — 여러 tool을 순서대로 조합해야 풀리는 문제를 푸는가

`✅` 저자들의 출발 관찰: SOTA MLLM이 **단순한 orientation 변화나 natural corruption에 놀랄 만큼 취약**하다(surprisingly brittle). 이게 그동안 간과돼 왔다는 게 논문의 motivation입니다.

> **우리 과제와의 연결:** RotBench(0단계 전제)가 "회전을 *맞히지* 못한다"를 보였다면, CodeVision은 "회전되면 *다운스트림 태스크가* 무너진다"를 보입니다. 진단의 층위가 다릅니다 — 우리 논문 intro에서 둘을 나란히 쓰면 "인식 실패 → 성능 실패"로 논리가 이어집니다.

---

## 2. 방법: code-as-tool

`✅` 핵심 설계는 **tool registry를 고정하지 않는 것**입니다.

- 기존 thinking-with-images: `crop`, `zoom` 등 미리 정의된 함수 목록을 주고 그중에서 고르게 함
- CodeVision: **코드를 universal interface로 삼아**, 모델이 필요한 image operation을 그때그때 코드로 작성해 실행

즉 tool 집합의 크기가 "저자가 등록한 개수"가 아니라 "코드로 표현 가능한 모든 연산"이 됩니다. 회전 관점에서 보면 `rotate(90)` / `rotate(180)` 같은 이산적 선택지가 아니라 임의 각도·임의 조합이 가능해집니다.

`❓` 실행 환경(sandbox), 허용 라이브러리(PIL/OpenCV 등), turn 수 상한 — 원문 확인 필요.

---

## 3. 학습 파이프라인 (2-stage)

### Stage 1 — SFT (cold start)

`✅` 약 **5,000개** 규모의 고품질 데이터셋. 세 종류의 시나리오를 커버합니다.

| 시나리오 | 내용 |
|---|---|
| single-tool | tool 하나로 끝나는 경우 |
| multi-tool chain | 복잡한 tool 연쇄 |
| error handling | 일부러 잘못된 tool 사용·runtime error를 넣음 |

`✅` **데이터 생성 방향이 역방향**인 게 포인트입니다. metadata를 먼저 sampling하고, 그에 맞춰 원본(정방향) 이미지를 **변형해서** 모델의 초기 관찰을 만듭니다. 예: 정답 tool이 `rotate-180`이면 원본을 180° 돌린 걸 입력으로 준다.

`✅` error handling 데이터는 모델이 **error log를 읽고 → 코드를 고쳐 → 재시도 → 결국 올바른 tool로 갈아타는** 궤적을 담습니다.

`✅` SFT의 역할: tool call의 기본 syntax와 패턴을 가르치는 **warm start**. 이 덕분에 이어지는 RL이 훨씬 안정적이고 효과적이 된다고 서술합니다.

### Stage 2 — RL (dense process reward)

`✅` outcome reward만으로는 부족하다는 게 이 단계의 주장입니다. reward가 두 부분으로 구성됩니다.

| 항목 | 역할 |
|---|---|
| **R_strategy** | strategy-shaping reward. 전략적인 tool 사용을 유도하는 **dense process signal** |
| **P_cost** | constraint penalty. 불필요하게 비싼/긴 tool 사용에 페널티 |

`❓` **RL 알고리즘 이름 미확인.** GRPO 계열일 가능성이 높지만 검색으로는 확정 못 했습니다. — 인용 전 반드시 확인
`❓` R_strategy의 구체적 수식/판정 기준, P_cost의 정의도 원문 확인 필요.

---

## 4. 평가 세팅

### 4-1. 기존 benchmark를 변형해서 씀

`✅` **OCRBench**, **ChartQAPro**에 **5가지 transformation**을 적용해 augmented version을 만듭니다.

| # | transformation |
|---|---|
| 1 | rotate 90° |
| 2 | rotate 180° |
| 3 | rotate 270° |
| 4 | horizontal flip |
| 5 | vertical flip |

두 benchmark를 고른 이유는 성격이 달라서입니다 — OCRBench는 **perception-critical**, ChartQAPro는 **reasoning-heavy**.

> **주의:** flip이 들어가 있다는 게 중요합니다. rotation만 다루는 우리 과제와 scope가 다릅니다. flip은 rotation group에 속하지 않으므로(reflection), canonicalization 이론틀(A-1/A-2)에서 다루는 group이 O(2)까지 넓어집니다. **우리가 rotation만 볼 것인지 flip까지 볼 것인지가 scope 결정 지점.**

### 4-2. 새 benchmark: MVToolBench

`✅` multi-tool reasoning을 평가하려고 새로 만든 benchmark. **orientation correction과 fine-grained cropping을 둘 다** 요구하도록 설계됐습니다. 즉 "돌려놓고 → 잘라서 봐야" 풀리는 문제들.

`❓` 규모(문항 수), 출처 이미지, 정답 생성 방식 — 원문 확인 필요.

### 4-3. Base model

`✅` **Qwen2.5-VL 계열**과 **Qwen3-VL 계열** 두 라인에서 실험.
- **CodeVision-7B** — base는 Qwen2.5-VL-7B
- **CodeVision-8B** — base는 Qwen3-VL-8B `❓` (사이즈로 미루어 보면 그렇지만 미확인)

비교 대상에 **Gemini-2.5-Pro** 같은 closed-source 모델도 포함됩니다.

---

## 5. 주요 결과

`✅` 확인된 수치는 다음 두 개입니다.

### transformed OCRBench

| 모델 | 평균 점수 |
|---|---|
| Qwen2.5-VL-7B (base) | **56.0** `❓` 역산값 (73.4 − 17.4) |
| **CodeVision-7B** | **73.4** |

→ base 대비 **+17.4** 개선.

`✅` 저자 해석: 모델이 어떤 transformation이 걸렸는지 인식하고, 대응하는 tool을 적용해 **canonical view를 복원**한 뒤 추론에 성공한다.

### MVToolBench

| 모델 | 점수 |
|---|---|
| Gemini-2.5-Pro | 32.6 |
| **CodeVision-7B** | **60.1** |

→ 강력한 경쟁 모델 대비 **거의 2배**.

`❓` ChartQAPro 변형본 수치, CodeVision-8B 수치, transformation별(90/180/270/h-flip/v-flip) 분해 수치 — **전부 미확인. 우리한테 제일 중요한 게 이 분해 수치입니다** (아래 §8 참고).

---

## 6. Ablation

`✅` **R_strategy를 빼면 성능이 크게 떨어집니다.** 저자 결론: 단순 outcome-based reward로는 복잡한 tool-use strategy를 학습시키기에 불충분하고, dense process signal이 효과적인 reasoning path를 발견·강화하는 데 결정적이다.

`❓` 구체적 ablation 수치, SFT 없이 RL만 돌린 조건, P_cost 제거 조건 — 미확인.

---

## 7. Emergent 능력

`✅` 학습 결과 다음 행동들이 emerge했다고 보고합니다.

1. **flexible tool composition** — tool을 유연하게 조합
2. **efficient chained execution** — 효율적인 연쇄 실행
3. **error recovery from runtime feedback** — runtime 피드백을 읽고 스스로 복구

3번은 SFT에 error handling 데이터를 일부러 넣은 결과이므로, 순수 emergence라기보단 **설계된 emergence**로 읽는 게 맞습니다.

---

## 8. 우리 과제 관점 — 차별점과 공략 지점

### 이 논문이 이미 가져간 것

- "MLLM이 orientation 변화에 취약하다"는 **진단**
- 회전 보정을 tool call로 푸는 **접근 자체**
- 변형된 OCRBench/ChartQAPro라는 **평가 프로토콜**
- **MVToolBench**라는 multi-tool benchmark

즉 0단계에서 "경쟁 논문"이라고 적어둔 판단이 맞습니다. 문제 정의 수준에서는 상당 부분 선점당했습니다.

### 남아 있는 차별점

| 축 | CodeVision | 우리 구상 |
|---|---|---|
| 학습 대상 | 본 모델을 **통째로** SFT+RL | 본 모델은 **freeze**, 앞단만 학습 |
| 본 모델 교체 | 불가 (모델마다 재학습) | 가능 (앞단 재사용) |
| 학습 비용 | 7B/8B 전체 학습 | 소형 모델만 |
| 필요 조건 | 본 모델이 코드 생성 가능해야 함 | 본 모델은 아무거나 |

**가장 강한 실용적 논거:** CodeVision은 base model이 바뀔 때마다 전체 파이프라인을 다시 돌려야 합니다. GPT-5나 Gemini 같은 **closed-source 모델에는 아예 적용 불가**합니다(weight 접근이 없으니). 앞단 모델 방식은 closed-source 본 모델에도 그대로 붙습니다. — 이게 우리 쪽 novelty의 핵심 축이 될 수 있습니다.

### 반드시 확인해야 할 것 (우선순위 순)

1. **transformation별 분해 수치.** 90°와 270°에서 특히 약한지가 RotBench의 관찰과 이어집니다. 만약 CodeVision이 그것까지 해결했다면 우리 문제 정의가 흔들리고, 못 했다면 **정확히 거기가 우리 자리**입니다.
2. **회전이 필요 없는 정방향 이미지에서 성능이 떨어지지 않는가.** B-4(AdaTooler-V)가 지적한 "쓸데없이 tool 부르기" 문제. CodeVision에 P_cost가 있는 걸 보면 저자들도 인지한 문제로 보입니다.
3. **resampling artifact 통제 여부.** A-4(Artificial Mental Rotation)의 경고 — 디지털 회전은 알고리즘 특유의 artifact를 남기고, 모델이 그걸 shortcut으로 배울 수 있습니다. CodeVision은 디지털 변형으로 데이터를 만들었으므로 이 위험에 그대로 노출돼 있습니다. **저자들이 이걸 통제했는지 안 했는지가 우리의 공략 포인트** — 안 했다면 "실제 촬영 회전에서는 안 통한다"는 반증 실험이 곧 우리 기여가 됩니다.
4. 90°/180°/270°만 다루는지, 임의 각도(예: 37°)도 다루는지. 후자가 없다면 **continuous rotation이 빈 자리**입니다.

---

## 9. 원문에서 채워야 할 빈칸 (체크리스트)

- [ ] 전체 저자·소속 확정
- [ ] RL 알고리즘 이름
- [ ] R_strategy / P_cost 정확한 정의
- [ ] ChartQAPro 변형본 결과
- [ ] CodeVision-8B 결과 및 base model 확정
- [ ] transformation별 분해 수치 ★최우선
- [ ] 정방향(untransformed) 이미지에서의 성능 (regression 여부)
- [ ] MVToolBench 규모·구축 방식
- [ ] ablation 상세 수치
- [ ] Limitations 절 내용
- [ ] resampling artifact 언급 여부

---

## 출처

이 정리는 아래 2차 출처의 검색 요약을 기반으로 작성했습니다. **원문 PDF 대조 전입니다** (작업 환경에서 arxiv.org·openaccess.thecvf.com·alphaxiv.org 접근이 차단됨).

- arXiv 초록/HTML: https://arxiv.org/abs/2512.03746
- CVPR 2026 open access PDF: `openaccess.thecvf.com/content/CVPR2026/papers/Guo_Thinking_with_Programming_Vision_Towards_a_Unified_View_for_Thinking_CVPR_2026_paper.pdf`
- 코드: https://github.com/ByteDance-BandAI/CodeVision
- Hugging Face paper page: https://huggingface.co/papers/2512.03746
