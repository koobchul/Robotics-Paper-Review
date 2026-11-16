# IntervenGen: Interventional Data Generation for Robust and Data-Efficient Robot Imitation Learning

## 0. Paper Information

- Title: IntervenGen: Interventional Data Generation for Robust and Data-Efficient Robot Imitation Learning
- Authors: Ryan Hoque, Ajay Mandlekar, Caelan Garrett, Ken Goldberg, Dieter Fox
- Year: 2024
- Venue: IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS 2024)
- Paper: [arXiv](https://arxiv.org/abs/2405.01472) · [IEEE Xplore](https://doi.org/10.1109/IROS58592.2024.10801523)
- Code: 공식 구현 링크 미확인. 전체 의사코드는 프로젝트 페이지에 제공.
- Project: [IntervenGen Project Page](https://sites.google.com/view/intervengen2024)

---

## 1. One-line Summary

> IntervenGen은 새 장면에서 기존 정책을 실행해 실수 상태를 만들고, 소수의 인간 복구 시연을 물체 자세에 맞게 변환·재생해 대규모 교정 데이터를 생성하는 방법이다.

---

## 2. Why this Paper?

### Problem

- 성공 시연으로 학습한 정책은 물체 위치를 잘못 추정하거나 grasp를 놓치면 시연에 없던 상태를 방문한다.
- Interactive IL에서는 사람이 그 상태에서 복구 행동을 보여줄 수 있다. 물체 배치와 실수 유형을 충분히 다루려면 정책을 반복 실행하고 사람의 개입을 수집해야 한다.

### Motivation

- MimicGen은 물체 위치가 바뀌어도 물체 기준의 조작 궤적을 재사용할 수 있다는 점을 활용한다. 같은 아이디어를 **사람이 보여준 복구 궤적**에도 적용할 수 있다.
- 복구 시연을 재사용하더라도 **복구가 시작되는 상태**는 새로 얻을 수 있다. 물체 배치와 자세 추정 오차를 바꾼 뒤 정책을 실행하면, 해당 조건에서 실제로 어떤 실수가 생기는지 관찰할 수 있다.

### Main Contribution

- **Closed-loop policy execution**으로 다양한 실수 상태를 수집하고, **open-loop trajectory replay**로 해당 상태에서의 복구 데이터를 생성한다.
- 동일한 BC-RNN에서 학습 데이터만 바꿔 전체 성공 시연, 직접 수집한 복구 개입, 합성 복구 개입의 효과를 비교한다.
- 물체 자세 추정 오차와 물체 형상 불일치에서 복구를 평가하고, 시뮬레이션에서 학습한 블록 집기 정책을 실제 로봇에 zero-shot으로 배포한다.

---

## 3. Key Concepts

| 개념 | IntervenGen에서의 의미 |
|---|---|
| Distribution shift | 정상 시연에서 본 상태와, 관측 오차가 있는 환경에서 정책이 방문하는 상태가 달라지는 현상 |
| Interactive Imitation Learning | 현재 정책이 방문한 상태에 대한 전문가 피드백을 수집하는 방식. DAgger는 전문가 행동 라벨을 얻고, IntervenGen의 원본 개입 수집에서는 사람이 직접 제어권을 가져간다. |
| Human-gated intervention | 사람이 정책을 모니터링하다가 개입 시점을 판단하고 짧은 복구 궤적을 시연하는 방식 |
| Mistake / Recovery trajectory | 정책이 실수 상태에 이르기까지 실행한 구간과, 개입 후 그 상태에서 복구하는 구간 |
| Object pose estimation error | 로봇이 추정한 물체 자세와 실제 자세의 차이. 실제 물체 배치가 같아도 추정 오차가 달라지면 실수 행동이 달라질 수 있다. |
| BC-RNN | 관측 이력을 내부 상태에 담는 RNN 정책을 Behavior Cloning으로 학습하는 방식. 배포 시 새 관측을 받아 다음 행동을 결정한다. |

---

## 4. Method

### Overall Pipeline

```text
10 Human Task Demonstrations
             ↓
1,000 MimicGen Demonstrations → Train Base BC-RNN
             ↓
Roll Out Base Policy under Observation Error
             ↓
Collect 10 Human Interventions
             ↓
Sample New Scene and Observation Error
             ↓
Closed-Loop Policy Execution → Detect Mistake
             ↓
Transform and Replay Human Recovery Trajectory
             ↓
Keep Successful Rollouts and Extract Corrective Data
             ↓
Aggregate with MimicGen Demonstrations → Train Final BC-RNN
```

### Interventional Data Collection

- 초기 정책은 정확한 물체 자세 조건에서 얻은 전체 시연 10개를 MimicGen으로 증강한 데이터로 학습한다.
- 관측 오차를 넣고 초기 정책을 실행한다. 사람이 실수를 감지하면 텔레오퍼레이션으로 제어권을 받아 복구한다. 이후 합성에 재사용하는 원본은 **사람이 조작한 복구 구간**이다.

### Mistake Generation: Closed-Loop Policy Execution

- 새 물체 배치와 관측 오차를 샘플링하고, 정책이 현재 관측을 받아 행동하도록 실행한다.
- 예를 들어 peg가 실제보다 왼쪽에 있다고 관측하면 로봇이 빗나간 위치로 접근할 수 있다. 오차를 바꿔 정책을 다시 실행하면 접근 경로와 접촉 지점도 달라질 수 있다.
- 주요 실험은 **접촉 감지**를 정책 실행의 종료 및 복구 시작 조건으로 사용한다.

### Recovery Generation: Open-Loop Trajectory Replay

1. 원본 개입을 하나 선택해 사람의 복구 구간을 추출한다.
2. 물체와 end-effector 사이의 상대 자세가 보존되도록 복구 궤적을 현재 물체 자세에 맞춰 변환한다.
3. 현재 end-effector 자세에서 변환된 복구 궤적의 시작까지 보간한다.
4. 보간 구간과 복구 궤적을 순서대로 재생한다. 저수준 컨트롤러가 목표 자세를 추종하더라도, 작업 수준의 목표 자세 시퀀스는 재생 전에 결정된다.

### Policy Training with Generated Interventions

- 전체 과제를 성공한 합성 실행을 선별한다. 필터링의 목적은 정책이 저지른 실수 행동을 BC의 정답으로 학습시키는 것을 방지하는 데 있다.
- 합성 개입 1,000개를 기본 MimicGen 시연 1,000개와 합쳐 최종 BC-RNN을 학습한다.
- **재현 메모** — §IV-D의 제거 구간을 가리키는 문구와 뒤이어 설명하는 필터링 목적이 모호하다. 실제 학습 데이터에 저장하는 구간은 구현 전에 프로젝트 의사코드와 대조할 항목이다.

### Inter-Subtask Recovery & Offline Mode

- **Inter-Subtask Recovery** — 잘못 내려놓은 물체를 다시 잡아야 하는 경우처럼, 이전 subtask로 돌아가는 복구를 허용한다.
- **Offline Mode** — 사람이 실수와 복구를 의도적으로 시연하고 복구 구간을 표시한다. 합성 시에는 원본 실수 구간도 재생할 수 있다.

---

## 5. Important Models / Datasets / Methods

| 비교군 | 논문에서의 데이터 구성 |
|---|---|
| Base | 정상 조건의 기본 MimicGen 데이터로 학습한 초기 정책 |
| Source Int | 기본 데이터 + 사람 개입 10개 |
| Weighted Src Int | Source Int와 동일한 데이터. 사람 개입을 기본 데이터와 같은 빈도로 샘플링 |
| Source Demo | 테스트 조건에서 수집한 전체 인간 시연 10개 |
| MG Demo | 테스트 조건의 전체 인간 시연 10개를 MimicGen으로 증강한 1,000개 |
| I-Gen − Policy | 원본 개입 10개에서 합성 개입 1,000개 생성. 실수 생성에 open-loop replay 사용 |
| I-Gen | 원본 개입 10개에서 합성 개입 1,000개 생성. 실수 생성에 closed-loop 정책 실행 사용 |

- **Policy** — 모든 비교군은 robomimic BC-RNN을 사용한다.
- **Tasks** — MuJoCo/robosuite의 Nut Insertion, 2-Piece Assembly, Coffee, Nut-and-Peg Assembly와 실제 로봇의 Block Grasp를 평가한다.

---

## 6. Strengths

- **실수 상태의 coverage** — 물체 배치와 추정 오차를 각각 바꿔, 같은 원본 개입을 여러 정책 실패 상황에서 활용할 수 있다.
- **데이터 생성 단계의 ablation** — `I-Gen − Policy`와 비교해 실수 구간을 정책 실행으로 생성하는 효과를 확인한다.
- **사람 데이터 비용 측정** — 개입 수와 수집 시간을 함께 보고해, 직접 수집과 합성 증강을 사람 예산 기준으로 비교할 수 있다.

---

## 7. Limitations

### 적용 가정

- 물체 중심 subtask를 정의할 수 있고, 데이터 생성 시 subtask에 해당하는 물체 자세를 알아야 한다.
- 잘못된 관측에서도 **복구를 판단할 단서**가 필요하다. 예를 들어 완전히 닫힌 그리퍼는 grasp에 실패했음을 알려줄 수 있다.
- 물체 기준으로 옮긴 복구 궤적이 새 시작 상태에서 실행 가능해야 한다. 접촉 상태나 실패 유형이 크게 달라지면 이 가정이 약해진다.

### 실험 범위

- 주요 결과 중 일부는 접촉 후 실제 물체 자세가 주어지는 이상화된 조건이다. 부분 관측 실험도 수행했지만, 실제 힘·토크 신호를 사용하는 일반적인 설정은 후속 과제로 남는다.
- 실물 정량 평가는 Block Grasp 한 과제에서 정책별 10회 시행으로 보고했다. 시뮬레이션 주 결과는 학습 중 성능이 가장 높은 체크포인트를 기준으로 한다.

---

## 8. Trade-offs

| 설계 | 얻는 것 | 필요한 조건 |
|---|---|---|
| 정책으로 실수 생성 | 새 관측 오차에서 정책이 실제로 방문하는 상태 | 추가 rollout과 복구 시작 조건 |
| 변환된 복구 궤적 재생 | 적은 사람 개입으로 여러 장면에 복구 데이터를 생성 | 물체 자세 정보와 재사용 가능한 복구 동작 |
| 합성 데이터 규모 확대 | 같은 사람 예산에서 더 많은 교정 예시 | 합성·학습 시간과 실행 성공 여부를 확인하는 절차 |

---

## 9. Key Findings

### Main Results

접촉 후 실제 물체 자세를 제공한 시뮬레이션 조건의 과제 성공률. 출처는 **Table I**.

| 학습 조건 | Nut Insertion | 2-Piece Assembly | Coffee |
|---|---:|---:|---:|
| Base | 22% | 6% | 2% |
| Source Int | 40% | 6% | 10% |
| Weighted Src Int | 50% | 16% | 6% |
| Source Demo | 42% | 12% | 12% |
| MG Demo | 64% | 16% | 18% |
| I-Gen − Policy | 86% | 52% | 42% |
| **I-Gen** | **98%** | **70%** | **80%** |

- 초록은 최대 `39×` 개선을 보고한다. Coffee의 표 수치는 **2% → 80%**다. 두 수치의 비는 40배이고, 기준 대비 상대 증가량은 39배다.

### Human Intervention Budget

- **2-Piece Assembly:** 사람 개입 100개는 **46%**, 사람 개입 10개에서 만든 합성 개입 100개는 **24%**였다. 같은 개입 수에서는 직접 수집한 데이터가 더 높은 성능을 보였다.
- 사람 개입 10개로 합성 개입을 **1,000개**까지 늘리면 **70%**였다. 사람의 수집 시간은 **3.6분**으로, 100개 직접 수집의 **29.9분** 대비 약 12%였다.
- 같은 10개 원본 개입에서 합성 데이터를 **200 → 1,000 → 5,000개**로 늘리면 성공률은 **34% → 70% → 88%**였다.

### Robustness to Object Pose Estimation Error

- **부분 관측, Table III** — 접촉 후 제한된 단서만 제공했을 때도 Nut Insertion **90%**, 2-Piece Assembly **66%**를 기록했다. 실제 물체 자세를 제공한 조건보다 각각 8%p, 4%p 낮았다.
- **형상 불일치, Table II** — Nut-and-Peg Assembly 두 형상에서 I-Gen은 **92% / 88%**, Base는 **100% / 0%**였다. I-Gen은 대체 형상에서 첫 파지가 빗나가는 상황을 복구하도록 학습했다.

### Simulation vs. Real-Robot Results

- **Block Grasp, Table IV** — 시뮬레이션에서 학습한 정책을 실물 Franka에 zero-shot으로 배포했다.
- I-Gen은 시뮬레이션 **100% (50/50)**, 실제 로봇 **90% (9/10)**였다. Base는 각각 **6% (3/50)**, **0% (0/10)**였다.

### Ablation / Analysis

- `I-Gen − Policy` 대비 I-Gen은 Nut Insertion, 2-Piece Assembly, Coffee에서 각각 **12%p, 18%p, 38%p** 높았다.
- Nut Insertion의 3개 학습 시드에서 I-Gen은 **98%, 100%, 98%**를 기록했다.

---

## 10. Future Work / Open Challenges

- 실제 힘·토크나 촉각 신호로 접촉 위치와 복구 방향을 얼마나 잘 추론할 수 있는가?
- 접촉 전에 실수 가능성을 판단하면 어떤 실패를 예방할 수 있고, 어떤 상태의 복구 데이터가 줄어드는가?
- 양팔 또는 다지 손 조작에서 물체 기준 궤적을 변환할 때, 양팔의 상대 자세와 접촉 관계를 어떻게 보존할 수 있는가?

---

## 11. My Takeaway

### What I Learned

- **성공 시연 coverage**는 어디에서 작업을 수행할 수 있는지와 관련되고, **복구 시연 coverage**는 실행 중 어디로 벗어났을 때 돌아올 수 있는지와 관련된다.
- IntervenGen의 실험에서 중요한 비교는 `MG Demo vs. I-Gen`과 `I-Gen − Policy vs. I-Gen`이다. 앞의 비교는 **전체 시연과 개입 데이터**, 뒤의 비교는 **실수 상태를 얻는 방식**의 차이를 보여준다.
- 합성 개입의 가치는 생성 개수만으로 판단하기 어렵다. 같은 수의 개입에서는 직접 수집한 데이터가 강했고, 합성의 이점은 적은 사람 예산으로 더 넓은 실수 상황을 다룰 수 있다는 데서 나타났다.

### Connection to Our Research

> **연구 질문** — MimicGen 성공 시연으로 국소 동역학을 학습하고 CCIL 보정 라벨을 추가하면, 별도의 사람 복구 시연 없이도 실행 중 이탈한 정책의 복귀율을 높일 수 있는가?

| 비교 축 | IntervenGen | MimicGen + CCIL 연구 후보 |
|---|---|---|
| 교정 대상 상태 | 초기 정책이 새 장면과 관측 오차에서 방문한 실수 상태 | 성공 시연 주변에서 동역학 모델로 구성한 이탈 상태 |
| 교정 정답 | 사람의 복구 행동을 변환한 다단계 궤적 | 목표 시연 상태에 가까워지도록 계산한 한 스텝 상태–행동 쌍 |
| 사람 데이터 | 초기 성공 시연과 복구 개입 | 초기 성공 시연. 추가 복구 시연 없이 확보할 수 있는 성능을 검증 |
| 확인할 범위 | 원본 복구 궤적이 적용 가능한 물체 배치와 실수 유형 | 생성한 라벨이 유효한 시연 이탈 거리와 실제 정책 실패 상태의 coverage |

- 첫 비교는 CCIL README의 설계와 같이 **블록 집기의 접촉 전 접근 구간**에서 시작한다. 그리퍼 위치를 교란한 뒤 시연 근방으로 돌아오는지 확인한다. IntervenGen의 주 실험과 직접 비교할 경우에는 **물체 위치 관측 오차**를 별도 조건으로 맞춘다.
- 원본 시연, 정책 구조, 평가 초기 상태, 교란 조건을 공유한다. 데이터 양 대조군을 추가할 때는 **상태–행동 쌍 수와 BC 업데이트 횟수**를 맞춘다. 사람 개입 수와 수집 시간은 별도로 기록한다.
- **평가 지표** — 무교란 과제 성공률, 교란 후 일정 시간 안에 같은 작업 단계의 시연 근방으로 돌아온 비율, 교란 후 최종 과제 성공률.

### Connection to Previous Papers

| 논문 | 연결 지점 |
|---|---|
| MimicGen | 성공 궤적에 쓰던 물체 중심 변환을 IntervenGen은 복구 구간에 적용한다. |
| DexMimicGen | 양팔 관계와 접촉 순서를 고려하는 성공 시연 생성 방식이다. 양팔 복구에서는 어떤 관계까지 보존해야 하는지 연결해 볼 수 있다. |
| CCIL | 성공 시연의 국소 전이 모델에서 보정 라벨을 생성한다. IntervenGen과의 비교 질문은 사람 복구 개입 없이 **어떤 이탈 범위까지** 유효한 라벨을 얻을 수 있는가이다. |
| DemoGen | ADR과 비교할 때는 외란을 주는 위치, 복구 궤적을 만드는 방식, 합성 데이터의 검증 절차를 맞춰 본다. |

### Next Action

- [ ] robosuite 블록 집기에서 접촉 전 구간을 정하고, 기준 시연 상태와 교란 후 복귀 판정 조건을 정의한다.
- [ ] `BC (MimicGen)` rollout에 교란을 가해 실제 이탈 상태를 기록하고, CCIL이 생성한 상태와 얼마나 겹치는지 확인한다.
- [ ] `BC (MimicGen)`과 `BC (MimicGen + CCIL)`의 복귀율과 최종 성공률을 동일한 평가 조건에서 비교한다.
- [ ] 사람 복구 개입을 확보하면 `BC (MimicGen + Human Interventions)`을 추가한다. 같은 원본 개입을 증강한 비교군까지 구현하면 직접 수집과 합성의 효과를 각각 비교한다.

---

## 12. Section Notes

| Section | 읽으면서 남긴 메모 |
|---|---|
| Abstract / §I Introduction | 정책이 실수할 때마다 사람이 직접 새 복구를 수집해야 하는지 묻는다. 주 실험은 인식 오류 대응에 초점을 둔다. |
| §II Related Work | Interactive IL은 정책이 방문한 상태에 전문가 피드백을 제공한다. MimicGen은 물체 기준 변환을 통해 적은 시연을 재사용한다. |
| §III Preliminaries | POMDP에서 관측이 손상되는 문제를 설정한다. 생성 시 물체 자세에 접근할 수 있으며, 복구에 필요한 단서가 정책 관측에 남아 있다고 가정한다. |
| §IV IntervenGen | Fig. 2는 새 장면·오차 샘플링 → 정책 실행 → 실수 감지 → 궤적 변환 → 복구 재생의 순서를 보여준다. |
| §V Experiment Setup | MuJoCo/robosuite 네 과제와 실물 블록 집기 한 과제. 모든 baseline에 같은 BC-RNN을 사용한다. |
| §VI Experiments | Table I은 자세 오차와 baseline, Table II는 형상 불일치, Table III은 부분 관측, Table IV는 sim-to-real 결과다. |
| §VII Conclusion | 사람의 복구를 증강하면 적은 개입으로 복구 데이터를 넓힐 수 있다. 실제 접촉 센서 활용은 남은 문제다. |
| Supplementary | 전체 의사코드, Offline Mode, Inter-Subtask Recovery, 추가 시드 평가와 실물 사례가 있다. |

---

## 13. Questions / Discussion

1. CCIL이 생성한 상태는 실제 정책이 방문하는 실수 상태를 얼마나 포함하는가? 접촉 전과 접촉 후의 coverage가 다른가?
2. 새 상태에서 복구가 실패했다면, 해당 상태를 학습 데이터가 다루지 못한 것인지 정책 관측에 복구 단서가 부족한 것인지 어떻게 확인할 수 있는가?
3. 단일 스텝 복귀 오차가 줄어들 때, 정책 수준의 복귀율과 최종 과제 성공률도 함께 올라가는가?

---

## 14. References / Related Links

- [IntervenGen](https://arxiv.org/abs/2405.01472)
- [IntervenGen Project Page and Pseudocode](https://sites.google.com/view/intervengen2024)
- [MimicGen](../MimicGen/README.md)
- [DexMimicGen](../DexMimicGen/README.md)
- [DemoGen](../DemoGen/README.md)
- [CCIL](../CCIL/README.md)
- [DAgger](https://proceedings.mlr.press/v15/ross11a.html)
- [robomimic](https://github.com/ARISE-Initiative/robomimic)