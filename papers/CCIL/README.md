# CCIL: Continuity-Based Data Augmentation for Corrective Imitation Learning

## 0. Paper Information

- Title: CCIL: Continuity-Based Data Augmentation for Corrective Imitation Learning
- Authors: Liyiming Ke, Yunchu Zhang, Abhay Deshpande, Siddhartha Srinivasa, Abhishek Gupta
- Year: 2024 (arXiv 최초 공개: 2023)
- Venue: International Conference on Learning Representations (ICLR 2024)
- Paper: [arXiv](https://arxiv.org/abs/2310.12972) · [ICLR Proceedings](https://proceedings.iclr.cc/paper_files/paper/2024/hash/99831104028c3b7e6079fd8bdcc42c8f-Abstract-Conference.html)
- Code: [personalrobotics/CCIL](https://github.com/personalrobotics/CCIL)
- Project: [CCIL Project Page](https://personalrobotics.github.io/CCIL/)

---

## 1. One-line Summary

> CCIL은 전문가 시연으로 학습한 국소 연속 동역학 모델을 이용해, 시연에서 살짝 벗어난 상태에서 복귀하는 행동을 합성하고 그 보정 데이터를 Behavior Cloning에 추가하는 방법이다.

---

## 2. Why this Paper?

### Problem

- BC는 전문가가 방문한 상태에서 어떤 행동을 해야 하는지 학습한다. 정책을 실제로 실행하면 작은 행동 오차나 외란으로 인해 시연에 없던 상태에 들어갈 수 있다.
- 낯선 상태에서 정책이 또 잘못된 행동을 하면 시연 경로에서 더 멀어진다. 이때 실행 중 방문하는 상태 분포가 학습 데이터의 상태 분포와 달라지고, 오차가 연속해서 누적된다.

### Motivation

- 시연 주변의 이탈 상태에 대해 복귀 행동을 학습 데이터로 제공하면, 정책이 실행 중 작은 오차에서 회복할 수 있다.
- 실제 동역학과 학습한 동역학 모델이 시연 주변에서 모두 완만하게 변하고, 시연 지점의 예측 오차가 작다면, 시연에 없던 인접 상태에서도 예측 오차의 상한을 줄 수 있다.
- CCIL은 이 성질을 이용해 기존 시연 전이로부터 복귀 행동을 생성한다. 보정 라벨 생성에 추가 전문가 개입이나 새 환경 롤아웃을 요구하지 않는다.

### Main Contribution

- 시연 상태로 한 스텝 내에 근접하게 만드는 전이를 **corrective label**로 정의한다.
- 국소 연속성을 유도한 동역학 모델과 두 가지 라벨 생성 방법인 **BackTrack**과 **DisturbedAction**을 제안한다.
- 생성 상태와 행동이 시연에서 벗어난 정도에 따라 예측 오차를 상계하고, 이를 진자 실험에서 실증적으로 점검한다.
- 시뮬레이션 14개 과제에서 보정 데이터를 추가한 BC 정책을 평가한다.

---

## 3. Key Concepts

| 개념 | CCIL에서의 의미 |
|---|---|
| Covariate shift | BC 학습에 사용한 전문가 상태 분포와 학습된 정책이 실행 중 방문하는 상태 분포가 달라지는 현상 |
| Compounding error | 이전 행동의 오차가 이후 상태를 바꾸고, 그 상태에서 발생한 추가 오차가 순차적으로 누적되는 현상 |
| Expert data support | 전문가 상태 분포가 커버하는 영역. CCIL은 관측된 시연의 주변에서 새로운 상태–행동 쌍을 생성한다. |
| Local Lipschitz continuity | 특정 근방에서 상태나 행동의 작은 변화가 동역학 출력에 제한된 크기의 변화만 일으키는 성질 |
| Residual dynamics model | 현재 상태와 행동을 받아 한 스텝의 상태 변화량 $s_{t+1}-s_t$를 예측하는 모델 |
| Corrective label | 생성 상태 $s_G$에서 행동 $a_G$를 실행하면 목표 시연 상태 $s^*$에 가까워지는 전이 $(s_G,a_G,s^*)$. BC가 학습하는 입력과 정답은 $(s_G,a_G)$이다. |

---

## 4. Method

### Overall Pipeline

1. 전문가 시연에서 전이 $(s_t^*,a_t^*,s_{t+1}^*)$를 추출한다.
2. 전이를 잘 예측하면서 시연 주변에서 국소적으로 완만하게 변하는 동역학 모델 $\hat f$를 학습한다.
3. $\hat f$로 시연 상태에 도달할 수 있는 시작 상태와 행동의 조합을 구한다.
4. 시연에서 너무 멀리 떨어진 후보를 거리 기준으로 제외한다.
5. 통과한 보정 쌍을 전문가 시연과 합쳐 표준 BC로 정책을 학습한다.

### Expert Demonstrations & State–Action Representation

- 논문의 기본 입력은 $\mathcal D^*=\{(s_t^*,a_t^*,s_{t+1}^*)\}$이다.
- 실제 잔차 동역학을 $f$로 두면 상태 전이는 $s_{t+1}=s_t+f(s_t,a_t)$이다.
- 학습 모델 $\hat f$의 회귀 타깃은 관측된 다음 상태와 현재 상태의 차이인 $s_{t+1}^*-s_t^*$이다.
- 목표 상태 $s^*$에 대한 보정 전이의 조건은 $\|s_G+f(s_G,a_G)-s^*\|\le\epsilon_c$이다. 실제 동역학 $f$를 모르는 설정에서는 $\hat f$로 후보를 생성한다.

### Learning Locally Continuous Dynamics Models

- **식 (2)**는 시연 전이의 회귀 오차를 줄인다. 예를 들어 시연에서 그리퍼가 10 mm에서 12 mm로 이동했다면 해당 상태와 행동에서 예측할 변화량은 $+2$ mm이다.
- **식 (4)**는 같은 행동을 고정한 채 상태에 작은 가우시안 섭동 $\Delta_s$를 넣어 국소 민감도를 추정한다. 섭동 전후의 출력 차이를 입력 변화로 나눈 유한차분을 사용한다. 이 계산에는 섭동 상태의 실제 다음 상태 라벨이 필요하지 않다.
- **식 (3)**은 시연 전이 회귀 손실에 $\lambda\,\mathbf 1(\|\hat f'\|>L)$을 더하는 예시를 제시한다. $L$은 민감도 기준이고, $\lambda$는 위반에 부과하는 가중치다. 식에 적힌 지시함수는 초과량과 관계없이 위반 시 $\lambda$를 부과한다.
- **직관**은 동일한 행동에서 시작 위치를 0.1 mm 바꿨을 때 예측 이동량이 0.02 mm 변하는 경우와 3 mm 변하는 경우를 구분하는 것이다. 두 모델은 시연 지점에서 같은 예측을 내더라도 주변 상태에 대한 민감도가 다르다.
- 샘플 기반 패널티는 관측한 근방에서 모델의 국소 연속성을 유도한다. 모든 방향이나 실제 시스템의 립시츠 조건까지 확인하려면 별도의 근거가 필요하다.

### Generating Corrective State–Action Pairs

| 방법 | 먼저 정하는 행동 | 복귀 목표 | 역으로 찾는 값 |
|---|---|---|---|
| **BackTrack** | 시연 행동 $a_G=a_t^*$ | 시연의 현재 상태 $s_t^*$ | $s_G+\hat f(s_G,a_t^*)\approx s_t^*$를 만족하는 시작 상태 $s_G$ |
| **DisturbedAction** | 시연 행동에 작은 노이즈를 더한 $a_G=a_t^*+\Delta_a$ | 시연의 다음 상태 $s_{t+1}^*$ | $s_G+\hat f(s_G,a_G)\approx s_{t+1}^*$를 만족하는 시작 상태 $s_G$ |

- 두 방법 모두 **행동과 목표 상태를 먼저 정하고 시작 상태를 역으로 구한다.** 논문은 이를 근 찾기 문제로 쓰고, 계산이 간단한 Backward Euler 기반 방법을 사용한다.
- DisturbedAction에서 흔드는 것은 **행동 후보**다. §4.2 식 (4)에서 **동역학의 상태 민감도를 재기 위해 상태 입력을 흔드는 것**과 사용 목적이 다르다.

### Corrective Label Quality & Discontinuous Regions

- **BackTrack (Theorem 4.2)**에서 시연 지점의 모델 오차가 $\epsilon$ 이하이고 실제 동역학과 학습 모델이 해당 근방에서 각각 국소 립시츠 조건을 만족하면 다음 상한을 얻는다.

$$
\|f(s_G,a^*)-\hat f(s_G,a^*)\|
\le \epsilon+(K_1+K_2)\|s_G-s^*\|.
$$

- **부록 B.1 증명**은 생성 상태의 예측 오차를 실제 동역학의 변화, 시연 지점에서의 모델 오차, 학습 동역학의 변화로 분해한 뒤 삼각부등식을 적용한다.
- 모델이 복귀 목표를 정확히 맞추면, 생성 상태에서의 한 스텝 모델 예측 오차가 실제 복귀 오차와 같다. 근 찾기 결과가 목표에 정확히 도달하지 않으면 모델상에서 남은 목표 오차도 상한에 더한다.
- **DisturbedAction (Theorem 4.3)**의 상한에는 상태 이탈 거리, 행동 섭동 크기, 모델상 목표 도달 잔차가 들어간다.
- **Rejection sampling**은 생성 상태가 기준 시연 상태에서 거리 임계값을 넘으면 후보를 버린다. 거리는 상한을 관리하는 실용적인 기준이다. 접촉이 생기거나 끊기는 경계에서는 가까운 두 상태에서도 결과가 급변할 수 있다.

### Behavior Cloning with Augmented Data

- BC 입력 데이터는 원래 시연의 $(s_t^*,a_t^*)$와 채택한 보정 쌍 $(s_G,a_G)$의 합집합이다.
- 국소 립시츠 패널티는 **동역학 모델 학습**에 적용된다. 최종 정책은 증강 데이터에서 관측에 대응하는 행동을 표준 BC로 학습한다.
- 배포 시에는 학습된 BC 정책이 행동을 출력한다. 동역학 모델과 역산은 오프라인 데이터 생성 단계에서 사용된다.

---

## 5. Important Models / Datasets / Methods

| 항목 | 역할 |
|---|---|
| BC | 전문가 시연으로 정책을 학습하는 기본 비교군 |
| NoisyBC | 노이즈 기반 증강을 사용하는 BC 비교군 |
| MILO | 오프라인 데이터에서 동역학을 학습하는 모방학습 비교군 |
| MOREL | 실제 보상 함수에 대한 접근이 필요한 모델 기반 오프라인 RL 비교군 |
| BackTrack / DisturbedAction | 학습 동역학으로 보정 상태–행동 쌍을 만드는 두 방법 |
| Pendulum / Discontinuous Pendulum | 실제 동역학을 이용한 라벨 오차 측정과 불연속성 영향 분석 |
| F1TENTH | 고차원 LiDAR 관측을 쓰는 주행 환경 |
| gym-pybullet-drone | Hover, Circle, FlyThrough를 평가한 드론 시뮬레이터 |
| MuJoCo / MetaWorld | 이동 과제 4개와 조작 과제 4개에서 접촉과 잡음의 영향을 평가 |

---

## 6. Strengths

- 기존 시연 전이로 복귀 학습 데이터를 만들 수 있어, 사람이 정책의 실수마다 보정 행동을 다시 시연해야 하는 부담을 줄인다.
- 보정 라벨 생성과 정책 학습이 분리되어 있어 기존 BC 파이프라인에 적용하기 쉽다.
- 국소 가정 아래 라벨의 신뢰도를 분석할 수 있고, 이탈 거리와 행동 섭동 크기가 오차 상한에 어떻게 들어가는지 설명한다.
- 라벨 자체의 한 스텝 복귀 오차와 증강 데이터로 학습한 정책의 실행 성능을 각각 평가한다.

---

## 7. Limitations

- 시연 주변에서 실제 동역학이 국소적으로 연속적이고 학습 모델이 시연 지점에서 잘 맞는다는 가정에 의존한다.
- 접촉 시작·해제, 충돌, 미끄러짐처럼 작은 상태 차이가 다른 결과를 만드는 구간에서는 좋은 라벨을 확보하기 어렵다.
- 생성 상태가 시연에 가깝다는 사실만으로 모든 라벨이 물리적으로 타당하거나 실제로 복귀한다는 보장이 생기지는 않는다.
- 한 스텝 뒤 시연 상태에 가깝게 돌아오는 행동도 장기 과제 성공이나 최적성을 보장하지 않는다.
- 원 논문의 실증은 시뮬레이션이다. 픽셀 관측을 사용하는 정책이나 실제 양팔 로봇에 적용할 때는 상태 표현과 전이 예측 오차를 별도로 검증해야 한다.

---

## 8. Trade-offs

| 조절 변수 | 보수적으로 설정하면 | 범위를 넓히면 |
|---|---|---|
| 생성 상태의 허용 이탈 거리 | 시연 가까운 상태를 선택해 거리 기반 오차 상한을 줄이기 쉽다. | 더 큰 이탈을 커버하지만 외삽 오차가 커질 수 있다. |
| DisturbedAction의 행동 노이즈 | 시연 행동에 가까운 후보를 만든다. | 다양한 행동을 다루지만 목표에 도달하기 어려운 후보가 늘 수 있다. |
| 민감도 기준 $L$과 패널티 $\lambda$ | 예측의 급격한 변화를 줄이는 압력이 커진다. | 실제 급변을 표현할 여지가 커지지만 생성 후보의 예측 신뢰도가 낮아질 수 있다. |

---

## 9. Key Findings

### Main Results

- 논문의 §5.5는 CCIL이 **14개 시뮬레이션 과제 중 10개에서 비교 학습 방법 중 가장 높은 성능**을 냈다고 요약한다. 나머지에서는 기본 BC와 비교 가능한 성능을 보고한다.
- F1TENTH 성공률은 **BC 31.9%, NoisyBC 39.3%, CCIL 56.4%**였다. CCIL은 기본 BC보다 **24.5%p** 높았다. 평균 점수는 각각 0.58, 0.62, 0.75였다. (§5.2, Table 1)

### Robustness to Perturbations

- 드론 평가에서는 관측과 행동에 잡음을 넣었다. CCIL은 Hover, Circle, FlyThrough 세 과제에서 모두 비교 학습 방법보다 높은 점수를 기록했다. (§5.3, Table 2)
- MuJoCo와 MetaWorld 평가에도 관측 및 행동에 가우시안 잡음을 넣었다. 이 설정은 교란을 포함한 정책 성능을 평가하며, 교란 직후 시연 경로로 복귀한 비율을 직접 측정한 결과와는 구분한다. (§5.4, Table 3)

### Effect of Dynamics Discontinuity

- Pendulum에서 생성 라벨의 평균 L2 복귀 오차는 **0.02367**, 논문이 제시한 이론 상한은 **0.065**였다. (§5.1)
- CCIL은 연속 진자와 불연속 진자 모두에서 기본 BC를 개선했다. 불연속 진자에서는 개선 폭이 더 작았다. (§5.1, Fig. 3)

### Ablation Studies

- 진자 실험에서 명시적인 립시츠 제약 없이 학습한 동역학으로도 라벨을 생성했다. 그 조건은 기본 BC를 소폭 개선했지만 CCIL보다 성능이 낮았다. (§5.1)
- 우리 실험에서는 이 비교를 가져와 **같은 개수의 보정 라벨을 생성하되 동역학 학습의 국소 연속성 패널티 유무를 바꾸는 ablation**으로 검증할 수 있다.

### Manipulation Results

- MuJoCo와 MetaWorld의 8개 과제 중 CCIL은 Hopper, Walker, HalfCheetah, CoffeePull의 4개에서 비교 학습 방법 중 가장 높은 평균 점수를 기록했다.
- MetaWorld 4개 과제 모두에서 CCIL의 평균 점수는 기본 BC보다 높았다. CoffeePush와 DrawerClose에서는 다른 증강 또는 모방학습 비교군의 평균 점수가 더 높았다.
- Table 3의 값은 **과제 점수**다. 예를 들어 CoffeePull은 **BC 3552.59 → CCIL 4168.46**이었다. 이를 성공률이나 복귀율로 해석하지 않는다. (§5.4, Table 3)

---

## 10. Future Work / Open Challenges

- **국소 연속성 측정** — 상태 공간에서 어떤 방향과 어느 거리까지 한 스텝 예측이 신뢰할 만한지 데이터로 추정한다.
- **접촉 구간 처리** — 자유공간 접근, 접촉 시작, 파지 후 이동을 구분해 동역학 오차와 보정 라벨 채택률을 확인한다.
- **고차원 관측** — 상태 기반 동역학에서 생성한 $s_G$를 이미지나 관측 이력과 정합시키는 방법을 연구한다.
- **실물 적용** — 시뮬레이션에서 확인한 보정 효과가 실제 로봇의 제어 지연, 관측 잡음, 접촉 오차에서도 유지되는지 검증한다.

---

## 11. My Takeaway

### What I Learned

- 성공 시연을 많이 확보하는 것과, **성공 시연에서 살짝 벗어났을 때 어떤 행동으로 복귀하는지**를 가르치는 것은 서로 다른 데이터 요구다.
- CCIL에서 립시츠 조건은 보정 행동을 직접 결정하는 공식이 아니다. 시연 주변에서 **행동의 결과를 예측하는 모델**을 얼마나 신뢰할 수 있는지 뒷받침한다.
- 라벨 생성 단계에서는 **한 스텝 복귀 오차**, 정책 평가 단계에서는 **교란 후 복귀율과 최종 과제 성공률**을 확인할 수 있다.

### Connection to Our Research

> **연구 질문** — 별도의 인간 복귀 시연 없이, MimicGen이 생성한 성공 시연의 전이에서 학습한 국소 동역학으로 보정 라벨을 만들면 실행 중 교란에 대한 BC 정책의 복귀율과 과제 성공률이 향상되는가?

| 단계 | 적용 계획 |
|---|---|
| VR 시연 | xArm7 조작 과제의 원본 성공 시연을 수집한다. |
| MimicGen | 같은 과제에서 물체 배치가 다른 성공 시연을 생성한다. |
| CCIL | MimicGen 성공 시연의 전이로 $\hat f$를 학습하고, 시연 주변에서 복귀를 유도하는 $(s_G,a_G)$를 합성한다. |
| BC | MimicGen 시연과 보정 라벨을 이용해 정책을 학습한다. 두 조건에서 BC 구조와 학습 설정을 맞춘다. |

- **첫 적용 구간**은 블록에 닿기 전 그리퍼 접근으로 잡는다. 파지 전후와 파지 후 이동은 별도 구간으로 분석한다.
- **전이 로그**에는 관절 위치·속도, 엔드이펙터 자세, 그리퍼 상태, 물체 자세, 실제 제어기에 전달된 행동, 동일한 제어 주기에서의 다음 상태를 기록한다. 첫 실험은 상태 기반 BC로 구성해 생성 상태와 정책 입력의 대응을 단순하게 유지한다.
- **라벨 검증**에서는 보류한 시뮬레이션 전이와 물리적으로 유효한 인접 상태를 사용해, 거리 및 접촉 상태별로 한 스텝 예측 오차와 목표 상태 도달 오차를 확인한다.
- **정책 평가**에서는 같은 초기 배치, 교란 시점, 교란 크기, 평가 시드를 사용한다. 복귀율은 교란 후 정해진 스텝 안에 같은 과제 단계의 기준 시연 근방에 다시 진입한 비율로 정의하고, 최종 과제 성공률을 별도로 기록한다.

### Experiment Design

| 실험군 | BC 학습 데이터 | 비교 목적 |
|---|---|---|
| **BC (Human)** | VR 원본 시연 | 원본 시연 기준 |
| **BC (MimicGen)** | MimicGen 성공 시연 | 핵심 기준 |
| **BC (MimicGen + CCIL)** | 동일한 MimicGen 성공 시연 + CCIL 보정 라벨 | CCIL 추가 효과 |
| **BC (MimicGen + extra MimicGen)** | 동일한 MimicGen 성공 시연 + 추가 MimicGen 전이 | 추가 샘플 수를 맞춘 데이터 양 대조군 |

- 핵심 비교는 **BC (MimicGen) vs. BC (MimicGen + CCIL)**이다. 데이터 양 대조군에서는 추가된 **상태–행동 쌍 수**와 BC 업데이트 횟수를 맞춘다.
- 립시츠 패널티의 기여를 검증할 때는 `MimicGen + CCIL (w/o Lipschitz penalty)` 조건을 추가하고 생성 방식, 라벨 수, 평가 조건을 동일하게 둔다.

### Connection to Previous Papers

- **MimicGen / DexMimicGen** — 원본 시연을 새 장면 배치로 옮겨 성공 궤적을 확보하는 접근이다. 우리 실험에서는 그 궤적을 따라가는 중 생긴 국소 이탈에 대한 학습 데이터를 추가한다.
- **IntervenGen** — 소수의 인간 복귀 개입을 바탕으로 합성 복귀 데이터를 만들고, 일반 MimicGen 시연을 비교군으로 사용했다. 우리 설계의 차이는 **성공 시연 전이에서 동역학을 학습해, 별도의 인간 복귀 개입 없이 보정 라벨을 생성한다**는 점이다. 이 차이는 데이터 수집 조건과 라벨 생성 방식의 차이이며, 성능 우위는 실험으로 확인해야 한다.
- **DemoGen** — ADR과 비교할 때는 교란을 어디에 주는지, 복구 행동을 어떻게 정하는지, 어떤 관측과 행동을 BC에 제공하는지를 기준으로 읽는다.

### Next Action

- [ ] 블록 집기 과제에서 접촉 전 접근 구간을 선택하고 $(s_t,a_t,s_{t+1})$ 로그를 확인한다.
- [ ] 동일한 MimicGen 데이터로 BC 기준 정책과 잔차 동역학 모델을 학습한다.
- [ ] 접근 구간에서 BackTrack 라벨을 생성하고, 일부 후보를 시뮬레이터에서 검증해 실제 한 스텝 복귀 오차를 기록한다.
- [ ] `BC (MimicGen)`과 `BC (MimicGen + CCIL)`을 같은 무교란·교란 조건에서 비교한다.
- [ ] 보정 데이터가 도움이 되는지 확인한 뒤, 샘플 수를 맞춘 대조군과 립시츠 패널티 ablation을 추가한다.

---

## 12. Section Notes

| Section | 읽으면서 남긴 메모 |
|---|---|
| **Abstract** | 시연의 국소 동역학을 이용해 보정 데이터를 만들고, 모방학습 정책의 교란 대응을 평가한다. |
| **§1 Introduction** | BC의 데이터 요구는 성공 경로의 다양성과 실행 중 이탈 상태의 커버리지에서 생긴다. 논문은 후자를 복귀 라벨 생성으로 다룬다. |
| **§2 Related Work** | DAgger 계열의 전문가 활용, GAIL 계열의 환경 전이 수집, MOREL의 보상 접근, MILO의 오프라인 데이터처럼 분포 이동을 다루는 방법마다 필요한 정보가 다르다. MOREL과 MILO는 실험에도 포함된다. |
| **§3 Preliminaries** | $s_{t+1}=s_t+f(s_t,a_t)$로 표현한다. 학습 시 실제 동역학 함수와 보상 함수는 주어지지 않으며 전문가 시연 전이를 사용한다. |
| **§4.1 Corrective Labels** | 생성 상태에서 보정 행동을 적용했을 때 목표 시연 상태까지 남는 오차와, 그 전이를 예측한 모델의 오차를 각각 고려한다. |
| **§4.2 Dynamics Learning** | 시연 전이에 대한 회귀 손실과 시연 근방에서의 민감도 패널티를 함께 사용한다. |
| **§4.3 Label Generation** | BackTrack과 DisturbedAction은 목표와 행동을 정한 뒤 시작 상태를 역산한다. 생성 상태가 시연에서 너무 멀면 제외한다. |
| **§4.4 CCIL Algorithm** | 동역학 학습 → 후보 생성 → 거리 기준 선별 → 시연과 보정 데이터를 합쳐 BC 학습. |
| **§5 Experiments** | 진자에서 라벨 품질과 불연속성 영향을 분석한다. LiDAR 주행, 드론, 이동, 조작에서 증강 후 정책 성능을 평가한다. |
| **§6 Conclusion** | 국소적으로 신뢰할 수 있는 동역학 모델로 복귀 예시를 만들 수 있다. 실제 로봇과 복잡한 관측으로의 확장은 열린 문제다. |
| **Appendix** | A는 실제 동역학을 아는 경우의 라벨 생성, B는 오차 상한 증명, C.1은 동역학 학습 손실 후보, D는 실험 재현 세부 사항을 다룬다. |

---

## 13. Questions / Discussion

1. 원본 VR 시연과 MimicGen 시연 중 어느 쪽 전이가 시연 주변의 실제 동역학을 더 정확하게 예측하는가?
2. 접촉 상태와 이탈 방향을 고려했을 때, 어느 범위까지 생성 라벨의 한 스텝 복귀 오차가 허용 가능한가?
3. 복귀율이 높아졌을 때 최종 과제 성공률도 올라가는가? 복귀 후 어느 구간에서 다시 실패하는가?
4. 라벨 수를 맞춰도 CCIL의 국소 연속성 패널티가 라벨 품질과 최종 정책의 복귀율을 개선하는가?
5. 양팔 과제에서는 한쪽 팔의 이탈만 보정하는 예시와 두 팔 및 물체 상태를 함께 보정하는 예시가 각각 어떤 상황에서 필요한가?

---

## 14. References / Related Links

- Ke et al. (2024). *CCIL: Continuity-Based Data Augmentation for Corrective Imitation Learning*. [Paper](https://arxiv.org/abs/2310.12972) · [Code](https://github.com/personalrobotics/CCIL)
- Mandlekar et al. (2023). *MimicGen: A Data Generation System for Scalable Robot Learning using Human Demonstrations*. [CoRL Proceedings](https://proceedings.mlr.press/v229/mandlekar23a.html)
- Jiang et al. *DexMimicGen: Automated Data Generation for Bimanual Dexterous Manipulation via Imitation Learning*. [Paper](https://arxiv.org/abs/2410.24185)
- Hoque et al. (2024). *IntervenGen: Interventional Data Generation for Robust and Data-Efficient Robot Imitation Learning*. [Paper](https://arxiv.org/abs/2405.01472)