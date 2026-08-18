## 비교 분석 — 앞단 모델 계열 세 논문 (A-1 / A-2 / A-4)

> "본 모델 앞에 뭔가를 붙여 회전을 되돌린다"는 같은 발상의 세 논문을 비교합니다.
> 세 편 모두 **원문 PDF 대조 완료**. 세부가 더 필요하면 요청 주세요.

---

## 0. 논문 확인

| | 정식 제목 | 저자 | 시점 |
|---|---|---|---|
| **A-1** | *Equivariance with Learned Canonicalization Functions* | Kaba, **Mondal**, Zhang, Bengio, **Ravanbakhsh** (Mila/McGill) | arXiv 2211.06489v3 (2023-07) |
| **A-2** | *Equivariant Adaptation of Large Pretrained Models* | **Mondal**, Panigrahi, **Kaba**, Rajeswar, **Ravanbakhsh** | arXiv 2310.01647v2, NeurIPS 2023 |
| **A-4** | *Efficient Rotation Invariance in Deep Neural Networks through **Artificial Mental Rotation*** | Tuggener, Stadelmann, **Schmidhuber** | arXiv 2311.08525v1 |

**A-4 정체 확인:** 첨부해주신 파일이 맞습니다. 0단계 노트에 부제만 "Artificial Mental Rotation"으로 적어둬서 헷갈리셨을 텐데, 그게 이 논문의 방법 이름(AMR)입니다.

**A-1과 A-2는 같은 연구팀입니다.** 저자가 셋이나 겹쳐요(Kaba, Mondal, Ravanbakhsh). **A-2는 A-1의 직접 후속작**이고, A-1의 약점을 자기들이 고친 논문입니다. A-4는 완전히 독립된 그룹(Schmidhuber 랩)입니다.

---

## 1. 계보

```
A-1 (2022)  canonicalization을 학습으로 하자
   │        └ 문제 발견: 정렬된 자세가 본 모델에게 낯설면 성능이 떨어진다
   ↓
A-2 (2023)  그 자세를 본 모델이 익숙한 쪽으로 규제하자 (prior)
            └ 대상을 large pretrained model로 확장 (SAM 등)

A-4 (2023)  (독립) 사람의 mental rotation을 그대로 구현하자
```

**흥미로운 사실 — A-1도 mental rotation을 근거로 듭니다.** A-4만 그런 게 아니에요. A-1 서론이 Shepard & Metzler(1971)를 인용하며 "회전 각도에 비례해 반응 시간이 늘어나는 건 사람이 mental rotation을 한다는 가설과 부합한다"고 씁니다.

그리고 Tarr & Pinker(1989)의 3분류를 그대로 가져옵니다:

| 유형 | 딥러닝 대응 |
|---|---|
| viewpoint-independent | 구조적 equivariant 네트워크, invariant feature만 쓰기 |
| multiple-view | 모든 변환에 대해 평균내기, **data augmentation** |
| **single-view-plus-transformation** | **canonicalization ← A-1, A-2, A-4 전부 여기** |

A-1의 표현으로 *"the transformation approach has seen less interest"* — **인지과학이 지지하는데도 딥러닝에서 가장 덜 탐구된 갈래**라는 겁니다. 우리 논문 intro에 그대로 쓸 수 있는 문장이에요.

---

## 2. 핵심 차이 — 무엇을 보장하려 하나

| | A-1 | A-2 | A-4 |
|---|---|---|---|
| 근거 | group theory | group theory + 데이터 분포 | 인지과학 |
| **보장** | **일관성** (equivariance, 구조적) | 일관성 **+ 본 모델 친화성** | 없음 |
| 목표 자세 | (아무거나, 일관되기만) | **본 모델이 익숙한 자세** | **정방향(upright)** |

**A-1의 함정이 A-2에서 숫자로 증명됩니다.** CIFAR10 / ResNet50 기준:

| 방법 | 원본 정확도 | 회전 평균(C8) |
|---|---|---|
| Vanilla (아무 처리 없음) | **96.97** | 57.77 |
| Rotation Augmentation | 94.91 | 90.11 |
| **A-1의 Learned Canonicalization** | **93.29** ⚠ | 92.96 |
| **A-2의 Prior-Regularized LC** | **96.19** | **95.31** |

**A-1을 붙이면 회전 성능은 오르지만 원본 성능이 96.97 → 93.29로 떨어집니다.** 앞단이 만든 "canonical 자세"가 본 모델에겐 낯선 그림이라서요. A-2의 prior가 이걸 96.19까지 회복시킵니다.

**A-2의 prior는 놀랍도록 단순합니다.** 이산 그룹에서 prior를 identity에 두면 손실이 이렇게 줄어듭니다:

```
L_prior = −E_x~D [ log p_c(x)(I) ]
```

말로 옮기면 — **"이미 정방향인 학습 데이터에 대해서는 '회전 필요 없음'이라고 답하라."** 그게 전부입니다.

> **이게 우리 과제의 abstain 아이디어와 정확히 같은 것입니다.** B-1의 Inappropriate Tool Use Penalty도, B-0의 Metacognitive Controller도 결국 이 얘기예요. **2023년에 A-2가 이미 손실 함수 한 줄로 풀어놨습니다.**

---

## 3. 돌리는 메커니즘 비교

| | A-1 | A-2 | A-4 |
|---|---|---|---|
| 앞단의 형태 | G-CNN | G-CNN / G-WRN | BM feature를 tap하는 **add-on 헤드** |
| 출력 | 이산 회전군 원소 (`argmax` over fiber) | 동일 + softmax 분포 | **360-way 각도 분류** |
| 미분 가능성 | `argmax` 비미분 → **straight-through estimator** | 동일 | 분류 손실이라 문제 없음 |
| 본 모델 | **함께 학습** (end-to-end) | **freeze 또는 fine-tune** 둘 다 실험 | **freeze** |
| 본 모델 내부 접근 | 불필요 (독립 네트워크) | **불필요** (독립 네트워크) | **필요** ⚠ (5개 stage에서 feature 복사) |
| 학습 신호 | downstream task 손실 | task 손실 + prior 규제 | **self-supervised** (랜덤 회전 → 각도 복원) |

**셋 다 "픽셀 → 각도"를 뱉는 미분 가능(또는 분류) 모듈입니다.** reasoning도, tool call도, semantic 판단도 없습니다. **여기가 우리 novelty의 자리입니다.**

---

## 4. 각 논문의 결정적 실험

### A-4 — 가장 강한 결과, 그리고 우리 설정에 가장 가까움

ImageNet, 9개 아키텍처 평균 (ceiling = 정방향 학습·정방향 테스트):

| 조건 | 회전 정확도 | ceiling 대비 |
|---|---|---|
| 정방향 학습 모델에 회전 입력 | 0.501 | 69% |
| **Rotation augmentation** | 0.640 | **87%** |
| **AMR (33 epochs)** | 0.716 | **98%** |
| AMR (5 epochs) | 0.708 | 97% |

**augmentation 87% vs AMR 98%.** 그리고 **5 epoch만 학습해도 97%**입니다. 앞단 방식이 augmentation을 확실히 이긴 사례예요.

두 가지 추가 실험이 특히 중요합니다.

**① 실물 촬영 검증** — 이미지를 **인쇄해서 실제로 촬영**한 84장에서:
- ResNet50 단독: **0.57**
- ResNet50 + AMR: **0.96**

저자 결론: *"AMR이 self-supervision 과정이 남긴 artifact에 의존한 게 아니라 **이미지 내용을 이해해서** 각도를 분류하는 법을 배웠다."* → **0단계에서 제가 걱정했던 리샘플링 artifact 문제를, A-4는 직접 검증해서 넘었습니다.**

**② 얼린 downstream 모델로 transfer (재학습 0)** — ImageNet에서 학습한 AMR 모듈을, COCO semantic segmentation 사전학습 모델에 **아무 수정 없이** 갖다 붙임:

| | mean IoU |
|---|---|
| 정방향 | 57.6 |
| 회전 | **32.7** |
| **AMR 보정 후** | **55.2** |

**이게 선생님 구상의 가장 가까운 선례입니다.** 다른 데이터셋·다른 과제·얼린 모델에 앞단만 갈아끼워 손실 대부분을 회복.

### A-2 — 대형 얼린 모델에 붙인 사례

SAM(641M)에 canonicalizer(0.2M / 1.9M)를 붙인 zero-shot 실험:

| | mAP | C4-평균 mAP |
|---|---|---|
| SAM zero-shot | 62.34 | 58.78 |
| + canonicalizer (0.2M, G-CNN) | 59.28 | 59.28 |
| + canonicalizer (1.9M, G-WRN) | **62.13** | **62.13** |

**앞단 용량이 결정적입니다.** 작은 canonicalizer(0.2M)를 쓰면 equivariance는 얻지만 **원본 mAP가 62.34 → 59.28로 떨어집니다.** 저자 설명: *"표현력이 부족한 canonicalization function은 **복잡한 이미지를 identity로 매핑하지 못해서**"*.

> **번역하면 — 앞단이 약하면 "돌릴 필요 없는 이미지"를 괜히 돌립니다.** §2에서 본 abstain 문제가 용량 문제로 다시 나타난 겁니다. 우리가 소형 모델을 쓸 거라면 **반드시 마주칠 트레이드오프**예요.

---

## 5. 우리 과제 관점

### 우리 설정은 A-2 계열입니다

지난 대화에서 확인했듯, 본 모델이 하나로 고정돼 있고 범용성은 필요없습니다. 그러면:

| | 우리와의 핏 |
|---|---|
| A-1 | ✗ 본 모델과 **함께 학습**해야 함. 우리는 못 함 |
| **A-2** | ✓ **얼린 대형 모델 + 독립 앞단**. SAM 실험이 정확히 그 세팅 |
| A-4 | △ 얼린 본 모델은 맞지만 **본 모델 내부 feature를 tap**해야 함 |
| A-3 (RECON) | ✗ 범용 canonicalizer가 목표. 우리는 범용성 불필요 |

### A-4의 결정적 제약 — 이게 우리 자리를 만듭니다

A-4의 AMR 모듈은 base model의 **5개 stage에서 feature를 복사**해 옵니다. 저자들이 그렇게 설계한 이유도 명시돼 있어요 — "분류를 위해 학습된 feature가 각도 탐지에도 유용할 것"이고, 그래서 앞단이 아주 작아도 되고 빨리 학습된다는 겁니다.

**그런데 이건 white-box 접근을 전제합니다.** GPT-5나 Gemini에는 불가능해요. **A-2 방식(독립 네트워크)만이 closed-source 본 모델에 적용 가능합니다.**

### 남는 novelty 축 3개

**① 미분 가능 모듈 → reasoning 모델**
세 논문 모두 "픽셀 → 각도"를 뱉는 회귀/분류기입니다. 정방향 판정에는 **"이 물체는 원래 이렇게 서 있는 게 자연스럽다"는 semantic 지식**이 필요한데(0단계 A-5), 픽셀 회귀로는 그게 안 들어갑니다. reasoning 모델은 갖고 있고요.

**② CNN/ViT 분류기 → MLLM**
세 논문의 downstream은 전부 분류기·segmentation입니다. **MLLM은 아무도 안 했습니다.** 그리고 실패 양상이 다릅니다 — CNN의 회전 실패는 기하학적이지만, MLLM의 90°/270° 혼동은 semantic입니다(RotBench).

**③ self-supervised 각도 복원 → 본 모델 피드백**
A-4는 "랜덤 회전 후 각도 복원"이라는 self-supervision을 씁니다. A-2는 데이터셋 prior를 씁니다. **우리는 본 모델을 직접 물어볼 수 있습니다** — 앞단이 보정한 뒤 본 모델의 정답률/confidence를 reward로. A-2가 "데이터셋 분포"로 우회한 걸, 본 모델이 고정돼 있다는 조건 덕에 **직접** 할 수 있는 겁니다.

### 반드시 대비해야 할 것

- **원본 성능 하락.** A-1은 96.97→93.29, A-2의 작은 canonicalizer는 62.34→59.28. **앞단을 붙이면 정방향 이미지에서 손해를 볼 수 있습니다.** 논문에 반드시 이 수치를 보고해야 하고, A-2의 identity prior가 해법입니다.
- **앞단 용량.** A-2가 0.2M → 1.9M으로 키워야 원본 성능이 유지됐습니다. "소형이라 싸다"와 "충분히 커야 안 망친다" 사이 트레이드오프가 실재합니다.
- **augmentation baseline.** A-4에서 augmentation은 87% ceiling으로 만만치 않습니다. **우리 실험에도 augmentation baseline이 반드시 들어가야 합니다.**

---

## 6. 정리

- **A-1 → A-2는 같은 팀의 연속작.** A-1이 만든 문제(원본 성능 하락)를 A-2가 identity prior로 고쳤습니다.
- **우리 설정은 A-2 계열**입니다. A-3(RECON)도 A-1도 아닙니다.
- **A-4가 가장 가까운 선례**입니다 — 얼린 본 모델 + 앞단 + 다른 과제로 transfer + 실물 촬영 검증까지 했습니다. 다만 **본 모델 내부 접근이 필요**해서 closed-source에는 못 씁니다.
- **A-2의 identity prior가 abstain 문제의 기존 해답**입니다. B그룹이 2025년에 다시 발견한 걸 2023년에 손실 한 줄로 풀어놨어요. 우리가 abstain을 novelty로 주장하면 안 되는 이유가 하나 더 늘었습니다.
- **셋 다 reasoning을 쓰지 않고, 셋 다 MLLM을 다루지 않습니다.** 여기가 남은 자리입니다.

---

### ⚠ 인용 시 주의 — A-4 초록에 오류가 있습니다

초록에 *"top-1 **error** of 0.743, AMR outperforms ... (rotational data augmentation, average top-1 **error** of 0.626) by 19%"*라고 적혀 있는데, **이 숫자들은 error가 아니라 accuracy입니다.** (0.743 / 0.626 = 1.187 → "19% 향상"). 본문 Table 1도 전부 accuracy예요. **초록 그대로 인용하면 안 됩니다.**

---

*관련 파일: `논문 search (0단계).md` (요약) / `논문 비교 (B-0 vs B-1).md` (B그룹 비교)*
