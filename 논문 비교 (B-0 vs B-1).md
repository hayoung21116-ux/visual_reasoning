## 비교 분석 — Thinking with Images (B-0) vs CodeVision (B-1)

> 두 논문의 **관계와 구도**만 정리합니다. 세부 수치·실험 세팅은 `[구버전] 논문 search (1단계).md` 참고.

---

## 1. 한 눈에

| | **B-0. Thinking with Images** | **B-1. CodeVision** |
|---|---|---|
| 정식 제목 | *Thinking with Images for Multimodal Reasoning: Foundations, Methods, and Future Frontiers* | *Thinking with Programming Vision: Towards a Unified View for Thinking with Images* |
| 유형 | **Survey** (실험 없음, 57쪽) | **Method** (CVPR 2026, 20쪽) |
| 시점 | 2025-07 (v3) | 2025-12 (**5개월 뒤**) |
| 소속 | HKUST, UNC, Microsoft, CUHK, UIUC | ZJU + ByteDance |
| 다루는 범위 | 계열 **전체** | **한 칸** (Stage 2 × RL) |
| 산출물 | 3단계 taxonomy | code-as-tool 프레임워크 + 학습 레시피 + benchmark |
| 회전 문제 | **사실상 없음** (`rotate` 1회, `orientation` 0회) | **핵심 소재** |

**핵심 관계: B-0은 지도이고, B-1은 그 지도 위의 한 점입니다.** 경쟁 관계가 아니라 포함 관계예요.

> *두 논문 모두 원문 PDF 대조 완료 (B-0: v3 57쪽 / B-1: v1 20쪽).*

---

## 2. "unified"라는 같은 단어, 다른 뜻

제목이 헷갈리는 이유가 여기 있습니다. **둘 다 "통합"을 표방하는데 대상이 다릅니다.**

| | 무엇을 통합하나 |
|---|---|
| B-0 | **분류상의 통합** — 흩어진 연구 계열을 하나의 지도로 정리 |
| B-1 | **인터페이스상의 통합** — 흩어진 tool들을 코드 하나로 호출 |

B-0은 "이 분야가 어떻게 생겼는지" 통합하고, B-1은 "도구를 어떻게 부를지" 통합합니다. 서로 다른 층위라 충돌하지 않습니다.

---

## 3. 문제 진단의 층위가 다르다

두 논문이 "왜 thinking with images가 필요한가"에 답하는 방식이 다릅니다.

**B-0 — 인식론적 진단**
> text-centric CoT는 vision을 정적 context로 취급한다 → perceptual data와 symbolic thought 사이에 **semantic gap**이 생긴다.

원리적인 이야기입니다. "이미지를 말로 바꾸는 순간 잃는 게 있다."

**B-1 — 실증적 진단**
> 기존 연구는 crop만 쓰는데, **tool을 써도 2~5%밖에 안 오른다.** tool 없는 RL로도 비슷하다. 즉 **tool이 정말 필요한 문제를 다룬 적이 없다.**

훨씬 공격적이고 구체적입니다. 선행 연구가 도구의 가치를 증명하지 못했다는 지적이에요.

**→ 이 대비가 유용한 이유:** 우리 논문 intro를 쓸 때 **B-0에서 큰 그림을, B-1에서 날카로운 문제 제기를** 가져올 수 있습니다. B-0만으로는 "그래서 왜 회전이냐"가 안 나오고, B-1만으로는 "왜 이미지를 조작해야 하냐"가 안 나옵니다.

---

## 4. CodeVision의 좌표

B-0의 taxonomy에 B-1을 얹으면 이렇게 됩니다.

| Stage | 이름 | CodeVision |
|---|---|---|
| 1 | Tool-Driven Visual Exploration | |
| **2** | **Programmatic Visual Manipulation** | ← **여기 (RL 칸)** |
| 3 | Intrinsic Visual Imagination | |

그런데 **Stage 2 × RL 칸에는 이미 Thyme과 Visual Agentic RL Fine-Tuning이 있습니다.** CodeVision이 새 칸을 연 게 아니에요.

**그럼 CodeVision의 실제 기여는 무엇인가:**

1. **회전이라는 소재 발굴** — tool이 "있으면 좋은" 게 아니라 "없으면 못 푸는" 문제를 찾은 것
2. **dense process reward 설계** — must-use tool 보상 + reward hacking 억제 penalty
3. **평가 인프라** — 변형된 OCRBench/ChartQAPro + MVToolBench

패러다임 자체는 기존 것입니다. **이렇게 위치시키면 CodeVision을 정확하게 평가할 수 있습니다** — 과대평가도 과소평가도 아니게.

---

## 5. 흥미로운 지점: CodeVision은 서베이를 인용하지 않는다

6개월 먼저 나왔고, CodeVision이 정확히 그 서베이의 Stage 2에 속하는데도 **참고문헌에 없습니다.** (CodeVision이 "thinking with images"를 인용할 때 다는 출처는 **OpenAI o3 블로그**입니다.)

해석은 두 가지가 가능합니다.
- 이 분야가 너무 빨리 움직여서 서베이가 레퍼런스로 자리잡기 전이다
- 방법 논문 입장에서 서베이는 인용 우선순위가 낮다

어느 쪽이든 **실무적 함의는 같습니다** — 이 분야에서 서베이는 아직 표준 좌표계가 아닙니다. 우리 논문에서 taxonomy를 쓸 거라면 **서베이를 인용하되, 개별 방법 논문들도 직접 인용**해야 안전합니다.

---

## 6. 기여를 주장하는 축은 하나가 아니다 ★

*(원문 확인 후 정정한 절입니다. 처음엔 "B-0은 위로 가라 하는데 B-1은 옆으로 갔다"고 적었는데, 서베이가 그렇게 말하지 않습니다.)*

B-0은 3단계를 **"increasing cognitive autonomy"**로 제시하지만, §2.2 *A Note on Non-Linearity*에서 **선형 진보가 아니라고 명시적으로 못 박습니다.**

> *"These three stages do not represent a strictly linear progression. They are different implementation strategies."*

근거로 든 예가 인상적입니다 — Stage 3가 가능한 모델이라도 "소파가 문을 통과하나?"를 알려면 full simulation보다 **Stage 1의 `measure` tool 한 번**이 낫다는 것. 그래서 **지능은 peak capability가 아니라 "과제에 맞는 인지 도구를 고르는 능력"**이라고 결론냅니다.

*(단, 초록은 "spectrum of increasing cognitive autonomy"라 하고 Figure 1은 "Intelligence Increasing", Stage 3는 "the most advanced stage"라 부릅니다. **서베이 내부에 위계와 비선형성이 공존**하는 긴장이 있어요. 인용할 때 어느 쪽을 쓰는지에 따라 논지가 달라지므로 주의.)*

**→ 그래서 진짜 함의는 이겁니다.** B-1이 Stage 2에 머문 건 서베이에 역행한 게 아니라 **서베이 자신의 원칙과 일치**합니다. 그리고 이건 우리에게 좋은 소식이에요 — **기여를 주장하는 방법이 "다음 단계로 올라가기"만이 아니라는 걸 서베이가 스스로 인정**한 셈이니까요.

| 축 | 사례 |
|---|---|
| 단계를 올린다 | Stage 3 계열 연구 |
| **문제를 바꾼다** | **B-1** (회전이라는 소재 발굴) |
| **구조를 바꾼다** | **우리** (본 모델 freeze + 앞단 분리) |

---

## 7. 두 논문의 공통 사각지대

겹쳐 놓고 보면 **둘 다 안 다루는 것**이 드러납니다. 여기가 우리 자리입니다.

| 사각지대 | 내용 |
|---|---|
| **canonicalization 계열 전체** | B-0 57쪽 전문에 `canonical`·`equivariance` **각 0회**. B-1 참고문헌에도 RotNet·EquiAdapt·RECON 없음. 회전이라는 같은 문제를 다른 학문 계열이 다뤄왔는데 **서로 모름** (정량 확인 완료) |
| **본 모델을 freeze하는 설계** | 둘 다 "본 모델이 주체"를 전제. 별도 앞단 모듈이라는 선택지가 taxonomy 축에 없음 |
| **closed-source 본 모델** | B-1의 방법은 weight가 필요해 GPT-5·Gemini에 적용 불가. B-0의 Stage 1~3도 대부분 학습 전제 |
| **방향이 정의되지 않는 이미지** | 두 논문의 평가가 전부 텍스트 중심(OCR·chart)이라 방향이 항상 잘 정의됨 |

### ⚠ 반대로, 선점된 것

사각지대만 보면 안 됩니다. **B-0 §8이 이미 제안해둔 것**도 있어요.

| 우리가 빈 자리로 봤던 것 | B-0의 선점 |
|---|---|
| **회전 abstain 정책** | §8.5.2의 **Metacognitive Controller** — "Thinking with Images가 필요한가?"를 판정하는 부품을 청사진으로 제시. §8.1은 **"불필요한 visual step에 페널티를 주는 reward"**를 명시 |
| 오차 전파 문제 | §2.4 Challenge 2 (**Information Density**) — 시각적 오류가 **false ground truth**를 세워 이후 추론 전체를 오염시킨다고 이미 언어화 |

**abstain을 그대로 novelty로 주장하면 안 됩니다.** 다만 결정적 차이가 하나 있어요 — 서베이의 Controller는 **통합 모델 내부의 부품**이고, 우리 것은 **본 모델 외부의 독립 모듈**입니다. 그리고 §2.4 Challenge 2는 **우리한테 불리한 게 아니라 유리합니다** — 앞단 방식의 최대 리스크를 서베이가 직접 언어화해줬으니, 우리가 그걸 정면으로 측정하면 그 자체가 기여가 됩니다.

---

## 8. 우리 과제의 위치

B-0의 축은 **"본 모델이 얼마나 스스로 하는가"**입니다. 우리 구상은 이 축 위에 없습니다 — 앞단 모델이 자율적으로 tool을 불러도 그건 본 모델의 autonomy가 아니니까요.

```
B-0의 축:  본 모델이 얼마나 스스로 하는가   (Stage 1 → 2 → 3)
우리의 축:  그 능력이 어디에 위치하는가      (본 모델 내부 ↔ 외부 모듈)
```

**"우리는 Stage 4다"가 아니라 "우리는 직교하는 설계 차원을 제안한다"**가 방어하기 좋은 프레임입니다.

**그리고 B-0이 이 프레임의 어휘를 직접 줍니다.** §8.5.1이 두 개념을 구분해요:

> Thinking with Images = **cognitive engine** (어떻게 사고하는가)
> Agent framework = 그 engine이 들어가는 **machine** (어떻게 행동하는가)

이 구분을 빌리면 우리 기여는 **"engine을 교체 가능하게 만드는 machine 층위의 설계"**가 됩니다. 서베이가 engine 내부(Stage 1→3)를 정리했다면, 우리는 engine과 그 앞단의 인터페이스를 다루는 것.

그 차원에서 나오는 실용적 이득:
- 본 모델을 건드리지 않으므로 **다른 능력이 퇴화하지 않음**
- 학습 비용이 소형 앞단으로 국한
- (앞단을 독립 네트워크로 두면) closed-source 본 모델에도 적용 가능 — **단 이건 설계 선택에 달림. `논문 비교 (A-1 vs A-2 vs A-4).md` §5 참고**

---

## 9. 정리

- B-0과 B-1은 **경쟁 관계가 아니라 포함 관계**입니다. 지도와 그 위의 한 점.
- B-1의 novelty는 **패러다임이 아니라 문제 선택과 학습 레시피**에 있습니다. 그리고 그 소재 발굴은 실제로 새로웠어요 — **B-0에 `orientation`은 0회 등장**합니다.
- 우리가 넘어야 할 건 B-0이 아니라 **B-1**입니다. B-0은 오히려 우리 논문의 **positioning 도구**로 쓸 자산이에요 (§8.5.1의 engine/machine 어휘).
- 남는 가장 큰 빈 자리는 **canonicalization 계열과의 단절**과 **본 모델을 freeze하는 설계**입니다.
- 다만 **abstain 아이디어는 B-0 §8.5.2에 부분 선점**돼 있으니, 그대로 novelty로 내세우면 안 됩니다.

---

*상세 내용: `[구버전] 논문 search (0단계).md` (요약) / `[구버전] 논문 search (1단계).md` (실험 세팅·결과·taxonomy 전문)*
*두 논문 모두 원문 PDF 대조 완료.*
