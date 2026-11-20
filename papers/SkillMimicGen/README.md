# SkillMimicGen: Automated Demonstration Generation for Efficient Skill Learning and Deployment

## 0. Paper Information

- Title: SkillMimicGen: Automated Demonstration Generation for Efficient Skill Learning and Deployment
- Authors: Caelan Reed Garrett, Ajay Mandlekar, Bowen Wen, Dieter Fox
- Year: 2024 (학회 발표), 2025 (PMLR 출판)
- Venue: Conference on Robot Learning (CoRL 2024)
- Paper: [arXiv](https://arxiv.org/abs/2410.18907) · [PMLR Proceedings](https://proceedings.mlr.press/v270/garrett25a.html)
- Code: 공식 프로젝트 페이지에서 별도 코드 저장소 링크를 확인하지 못함
- Project: [SkillGen Project Page](https://skillgen.github.io/)

## 1. One-line Summary

> SkillMimicGen은 소수의 인간 시연에서 물체 조작 구간을 추출해 새 장면에 맞추고, 그 사이의 이동을 motion planning으로 연결해 시연을 생성한 뒤, 배포 시 planner와 closed-loop 조작 정책을 번갈아 실행한다.

## 2. Why this Paper?

### Problem

- Long-horizon manipulation의 전체 궤적을 사람이 반복 시연하는 데 많은 시간이 든다.
- MimicGen은 물체 기준으로 시연 구간을 변환해 재사용한다. 구간 사이를 직선 보간할 때 물체 배치가 크게 달라지거나 장애물이 추가되면 충돌이 발생할 수 있다.

### Motivation

- 물체까지 접근하거나 물체를 든 채 운반하는 구간은 motion planner가 경로를 계산할 수 있다.
- 잡기와 삽입처럼 접촉이 중요한 구간에는 사람의 시연을 재사용하고, 배포 시 관측에 반응하는 정책을 학습시킬 수 있다.

### Main Contribution

- **SkillGen**은 대상 물체 기준으로 변환한 짧은 조작 구간과 새로 계획한 이동 구간을 연결해 시연 데이터를 생성한다.
- **Hybrid Skill Policy (HSP)**는 조작을 시작할 자세, 조작 중 행동, 조작을 종료할 시점을 생성 데이터에서 학습한다.
- 시뮬레이션 18개 변형에서 60개 인간 시연으로 총 24,000개 이상의 시연을 생성했다. 실제 로봇에서 데이터를 생성한 3개 과제와 별도의 zero-shot sim-to-real 조립 과제도 평가했다.

## 3. Key Concepts

| 개념 | 의미 |
| --- | --- |
| Manipulation skill | 대상 물체 근처에서 수행하는 국소 조작. 커피 캡슐 잡기와 삽입이 각각 하나의 스킬이 될 수 있다. |
| Transit | 물체를 들지 않은 로봇이 다음 스킬의 시작 자세까지 이동하는 구간. |
| Transfer | 물체를 거의 고정된 상태로 잡은 채 다음 스킬의 시작 자세까지 운반하는 구간. |
| Initiation pose | 조작 정책을 시작할 end-effector의 위치와 방향. Motion planner의 목표 자세로 쓰인다. |
| Closed-loop skill | 실행 중 최신 관측으로 다음 행동을 결정하는 조작 정책. |
| HSP | 시작 자세 예측과 종료 판정을 학습하고, motion planner로 조작 정책을 이어서 실행하는 구조. |

## 4. Method

### 4.1 Skills Framework

스킬을 $\psi=\langle O,\mathcal I,\pi,\mathcal T\rangle$로 정의한다. $O$는 대상 물체, $\mathcal I$는 시작 가능한 상태 집합, $\pi$는 조작 정책, $\mathcal T$는 종료 조건이다. 이 구조를 원본 시연 분할, 시연 생성, 배포에 공통으로 사용한다.

### 4.2 Transit and Transfer Motion

커피 캡슐 과제는 **캡슐에 접근 (transit) → 캡슐 잡기 (skill) → 캡슐 운반 (transfer) → 홀더에 삽입 (skill)**으로 나뉜다. 직전 스킬이 끝난 관절 상태에서 다음 스킬의 시작 자세로 이동한다. 목표 end-effector pose를 역기구학으로 관절 구성으로 바꾼 뒤 joint-space 경로를 계획한다.

### 4.3 Source Demonstrations

- 각 원본 시연에서 스킬의 시작과 끝을 표시해 이동 구간과 조작 구간으로 나눈다.
- 일반 teleoperation에서는 사람이 구간 경계를 표시할 수 있다. 주 실험의 HITL-TAMP 수집에서는 사람이 국소 조작을 시연하고 TAMP가 이동을 담당하므로, 제어 주체가 바뀐 기록에서 경계를 추출한다.
- 접촉 전 대상 물체의 자세를 기록한다. 조작 시작 시점의 **실제 end-effector pose**와 조작 중 각 스텝의 **end-effector 목표 명령**을 물체 좌표계로 표현한다. 첫 자세는 planner가 도달할 목표이고, 마지막 관측은 종료 분류 학습에 쓰인다.

### 4.4 Demonstration Generation

1. 새 초기 상태를 샘플링하고, 각 스킬에 쓸 원본 조작 구간을 선택한다.
2. 새 장면의 물체 pose를 이용해 원본의 시작 자세와 action pose를 새 세계 좌표계로 옮긴다.
3. 시작 자세까지 경로를 계획해 이동한 뒤, 변환한 조작 action들을 순서대로 재생한다.
4. 모든 스킬을 실행한 뒤 전체 과제의 성공 여부를 확인하고, 성공한 시연을 저장한다.

$T^X_Y$를 좌표계 $X$에서 $Y$로의 변환으로 쓰면, 새 장면의 조작 시작 자세는 $T^{E'_0}_W=T^{O'}_W T^{E_0}_O$이다. 각 시점의 목표 명령에도 같은 방식으로 변환을 적용해 **물체를 기준으로 한 손의 상대 위치와 방향**을 유지한다.

### 4.5 Initiation Augmentation

- 배포 중 시작 자세 예측 오차나 경로 추종 오차가 생기면, 스킬이 학습에서 거의 보지 못한 자세에서 시작할 수 있다.
- 생성 단계에서 시작 자세를 교란하고, **교란된 시작 자세에서 원래 시작 자세로 돌아오는 recovery segment**를 조작 구간 앞에 추가한다. 이렇게 만든 시연으로 조금 어긋난 자세에서 출발하는 행동을 함께 학습한다.
- 주요 실험에서는 이 증강을 HSP-Reg의 학습 데이터에 적용했다.

### 4.6 Policy Learning and Deployment

| 구성 | 입력과 출력 | 실행 시 역할 |
| --- | --- | --- |
| $\mathcal I_\theta$ | 직전 스킬 종료 시점의 관측 → 다음 스킬의 시작 자세 | 예측한 자세를 planner의 목표로 사용한다. |
| $\pi_\theta$ | 최신 관측 → 조작 행동 | 생성된 관측·행동 쌍으로 BC-RNN을 학습하고, 시작 자세에 도달하면 실행한다. |
| $\mathcal T_\theta$ | 최신 관측 → 계속(0) 또는 종료(1) | 종료를 예측하면 주어진 순서의 다음 스킬로 넘어간다. |

| 변형 | 시작 자세 | 종료 |
| --- | --- | --- |
| HSP-Reg | 최근 관측에서 연속적인 end-effector pose를 직접 회귀한다. 시작 자세 예측에 별도의 물체 pose 입력을 요구하지 않는다. | 학습한 분류기 |
| HSP-Class | 원본 시연의 시작 자세 후보를 분류하고, 선택한 물체 기준 자세를 현재 장면에 맞춘다. 이 변환에 현재 물체 pose가 필요하다. | 학습한 분류기 |
| HSP-TAMP | 학습한 조작 정책을 HITL-TAMP 안에서 실행한다. 시작과 종료 조건은 TAMP 모델에 정의되어 있다. | TAMP 모델에 정의한 조건 |

**생성**에서는 변환한 조작 명령을 재생해 데이터를 수집한다. **배포**에서는 planner가 조작 시작 자세까지 이동한 다음, 학습 정책이 현재 관측에 반응하며 조작을 실행한다.

## 5. Important Models / Datasets / Methods

- **Simulation** · robosuite / MuJoCo. Square, Threading, Coffee, Three Piece Assembly, Nut Assembly, Coffee Prep의 6개 과제를 각각 D0·D1·D2에서 평가했다. D0는 원본 시연의 기본 초기화 분포이며, D1·D2는 더 넓고 어려운 초기화 분포다.
- **Training data** · 주요 프로토콜은 과제당 D0 원본 시연 10개, 변형당 성공 시연 1,000개다. 논문 초록의 24K+는 전체 보고 규모이며, 변형당 1,000개는 주요 실험의 기본 설정이다.
- **Policy** · 기본 실험에서는 정면·손목 RGB와 로봇 proprioception을 사용한다. 정책 성능은 학습 중 평가에서 기록한 최대 성공률을 보고한다.
- **Planning stack** · TRAC-IK, RRT-Connect, OSC를 각각 역기구학, 관절공간 경로 계획, 작업공간 제어에 사용한다. 충돌 검사는 시뮬레이션에서 장애물의 실제 형상을, 실물에서는 분할한 점군을 이용한다.
- **Real-robot setup** · Franka Panda, 정면 RealSense D415, 손목 RealSense D435를 사용했다. 실물 데이터 생성 과제에서는 FoundationPose로 물체 자세를 추정했다.

## 6. Strengths

- 장애물 배치가 달라져도 이동 경로를 다시 계획할 수 있어, 원본 시연에 없던 장면에서 생성 시도 성공률을 높였다.
- BC의 학습 범위를 국소 조작으로 좁히고, 시험 시 관측에 따라 조작을 갱신할 수 있다.
- 동일한 HSP-TAMP 구조에서 원본 데이터만 학습한 조건과 생성 데이터를 학습한 조건을 비교해, 생성 데이터의 효과를 일부 확인했다.

## 7. Limitations

- 과제를 수행하는 스킬 순서와 스킬별 대상 물체를 미리 제공해야 한다. 스킬 경계도 원본 데이터에 표시되어야 한다.
- 데이터 생성 시 접촉 전에 물체 자세를 관측하거나 추정해야 한다. HSP-Class는 배포 시 시작 자세를 변환하기 위해서도 물체 pose가 필요하다.
- 검증 범위는 주로 준정적 강체 조작이다. 일반 teleoperation 시연보다 HITL-TAMP로 수집한 원본 시연에서 더 좋은 결과를 얻었다.
- Zero-shot 조립 실험은 에피소드 시작 시 물체 pose를 받고, 실행 중에는 proprioception 변화만 관측하는 제한된 설정이다. 조작 행동도 위치 제어로 제한됐다.

## 8. Trade-offs

| 설계 선택 | 장점 | 필요한 정보와 비용 |
| --- | --- | --- |
| Planner로 구간 연결 | 장면마다 장애물을 고려해 경로를 다시 계산 | 충돌 모델, 역기구학, 계획 시간 |
| HSP-Reg | 관측에서 시작 pose를 직접 얻음 | 연속 pose 회귀 오차가 조작 시작 위치에 전달될 수 있음 |
| HSP-Class | 원본 시연 후보를 고르는 이산 예측 문제 | 배포 시 물체 pose 추정 및 자세 변환 |
| HSP-TAMP | 사람이 정의한 조건으로 스킬 전환을 관리 | 과제별 시작·종료 조건 설계 |

## 9. Key Findings

### Demonstration Generation and Policy Learning

| 지표 | MimicGen | SkillGen |
| --- | ---: | ---: |
| 평균 데이터 생성 성공률 (§6.1, Appendix F) | 40.7% | 75.4% |
| 평균 정책 성공률 (Fig. 4) | 59.1% (BC-RNN) | 85.7% (HSP-TAMP), 82.9% (HSP-Class), 72.6% (HSP-Reg) |

- 생성 성공률은 **생성 시도 중 과제를 성공한 비율**이고, 정책 성공률은 **학습한 에이전트가 평가에서 과제를 성공한 비율**이다.
- 평균 정책 성공률은 데이터와 배포 구조가 함께 바뀐 시스템 비교다. HSP-Reg 평균은 §6.2 본문에 72.2%, Fig. 4 및 §6.1에 72.6%로 표기되어 있어 표에는 Fig. 4 값을 사용했다.

### Scene Variations, Clutter, and Robot Transfer

- **Square D2 생성 성공률** · SkillGen 87.7%, MimicGen 31.8%.
- **Coffee Prep D2** · SkillGen 생성 성공률 59.9%. MimicGen은 성공 시연을 생성하지 못했다. SkillGen으로 학습한 HSP 세 변형의 정책 성공률은 74–84%였다.
- **Square·Coffee Clutter 변형** · 생성 성공률은 SkillGen 49.0–72.0%, MimicGen 4.0–16.5%였다. 이 결과는 해당 clutter 변형에서의 데이터 생성 결과다.
- **Robot transfer** · Panda 원본 시연을 Sawyer에 적응시켜 데이터를 생성하고 정책을 학습하는 실험도 수행했다 (Appendix N).

### Effect of Dataset Size and Source Data

- **생성 데이터 효과** · D0에서 HSP-TAMP를 원본 시연만으로 학습했을 때와 비교하면, 생성 데이터 학습 시 Three Piece Assembly는 28% → 96%, Nut Assembly는 22% → 100%였다.
- **동일한 학습 데이터 수** · 4개 과제에서 사람 시연 200개와 사람 시연 10개에서 생성한 SkillGen 시연 200개를 비교했다. HSP-TAMP 정책 성공률 차이는 최대 10%p였다.
- **데이터 규모** · 200개에서 1,000개로 늘렸을 때 비교한 4개 과제와 HSP 변형에서 성능이 개선됐다. 1,000개에서 5,000개로 늘렸을 때 HSP-Reg는 Square D2에서 52% → 72%, Threading D1에서 60% → 76%였다.

### Real-Robot and Zero-Shot Sim-to-Real

| 평가 | 학습 데이터 | 결과 |
| --- | --- | --- |
| Pick-Place-Milk / Cleanup-Butter-Trash | 과제당 실제 시연 3개 → 실제 생성 시연 100개 | HSP-Class 각 95% |
| Coffee | 실제 시연 3개 → 실제 생성 시연 100개 | HSP-Class 65%, 논문의 MimicGen 비교값 14% |
| Nut Assembly | 시뮬레이션 시연 1개 → 시뮬레이션 생성 시연 1,000개 | 전체 과제 기준 HSP-Class는 시뮬레이션 92%, 실물 35%. MimicGen은 각각 72%, 0%. |

### Reading the Comparisons

- Fig. 4의 HSP-TAMP, HSP-Class, HSP-Reg 비교에는 시작·종료 방식의 차이가 있다. 주요 실험의 HSP-Reg에는 시작 자세 증강도 적용됐다.
- D1·D2는 해당 분포에서 데이터를 **생성한 뒤 학습**한 실험이다. Zero-shot 전이는 별도의 Nut Assembly sim-to-real 실험에서 평가했다.

## 10. Future Work / Open Challenges

- 일반 VR teleoperation에서도 작업자 간에 일관된 스킬 시작·끝 기준을 정의할 수 있는가?
- 새 장애물이나 물체 pose 추정 오차 때문에 스킬 진입에 실패하면, 재계획과 국소 교정 중 어느 쪽이 유효한가?
- 고정된 스킬 순서와 준정적 강체 조작을 넘어갈 때 어떤 상태 정보와 전환 조건이 필요한가?

## 11. My Takeaway

### What I Learned

> 소수 시연을 확장할 때 **어느 구간의 행동을 재사용할지**와 **배포에서 그 구간에 어떻게 진입할지**를 함께 설계해야 한다. 동일한 조작 정책도 시작 자세가 달라지면 다른 상태를 만나므로, 조작 시작부의 데이터가 중요하다.

### Connection to Previous Papers

| 논문 | SkillMimicGen과 비교할 지점 |
| --- | --- |
| MimicGen | 두 방법 모두 물체 기준으로 시연을 변환한다. 변화점은 변환·재생 범위를 국소 조작에 집중하고, 그 사이를 planning으로 연결하며, 배포 시 스킬별 정책을 실행한다는 점이다. |
| CCIL | SkillGen의 initiation augmentation은 시작 자세 교란에서 원래 시작 자세로 회복하는 구간을 만든다. CCIL과 연결할 질문은 조작 시작 이후에 벗어난 상태에도 유효한 교정 행동을 만들 수 있는가다. |
| IntervenGen | 사람의 개입으로 수집한 복구 시연을 증강하는 접근과 비교할 수 있다. SkillGen의 시작 자세 증강은 스킬 진입 오차에 초점을 둔다. |
| DemoGen | 행동 변환, 관측 생성, 환경 실행, 배포 정책 구조를 각각 비교하면 합성 시연이 어떤 단계에서 만들어지는지 정리할 수 있다. |

### Connection to Our Research

- **VR 시연 분할** · xArm7 / robosuite의 삽입 과제 하나에서 접근, 잡기, 운반, 삽입 경계를 표시한다. 스킬 시작 시 대상 물체 pose와 실제 end-effector pose를 기록하고, 조작 중에는 목표 pose 명령을 별도로 저장한다.
- **실패 위치 기록** · planner가 시작 자세에 도달하지 못한 경우, 시작 자세는 도달했지만 조작에 실패한 경우, 종료가 잘못 판정된 경우를 롤아웃에 기록한다. 교정 데이터가 필요한 구간은 실패 위치를 확인한 뒤 고른다.
- **교정 데이터 비교** · HSP 구조와 원본 시연 수, 총 학습 시연 수, 평가 초기 상태를 고정하고 기본 SkillGen, 시작 자세 증강, 스킬 내부 교정 조건을 비교한다. 조작 성공률과 전체 성공률을 함께 기록하고 생성 시도 수 및 계획 시간도 남긴다.

### Next Action

- [ ] 삽입 과제 하나의 VR 시연에 스킬 경계를 표시하고, 원본 시연 수와 평가 초기 상태를 고정한다.
- [ ] 기본 HSP를 실행해 스킬 진입과 조작 내부의 실패를 기록하고, 그 결과로 교정 데이터의 적용 위치를 정한다.

## 12. Section Notes

| 원문 | 다시 볼 내용 |
| --- | --- |
| Abstract / §1 Introduction | 60개 → 24K+ 규모와 Fig. 1의 데이터 생성·배포 흐름. |
| §2 Related Work | SayCan과의 차이는 스킬 사이 이동, keyframe 방식과의 차이는 closed-loop 조작, HITL-TAMP와의 차이는 스킬 전환 조건에 있다. |
| §3 Prerequisites | POMDP 관측에서 행동을 고르는 BC. A1은 연속 EEF pose 명령과 이산 그리퍼 명령, A2는 조작 대상 물체, A3는 데이터 수집 시 접촉 전 물체 pose 접근이다. |
| §4 Method | §4.1–4.3은 스킬 구조와 원본 데이터, §4.4–4.5는 생성과 시작 자세 증강, §4.6과 Fig. 2는 HSP 배포. |
| §5 Experiment Setup | D0–D2 초기화 분포, BC-RNN, TRAC-IK / RRT-Connect / OSC, 최대 정책 성공률 보고 방식. |
| §6 Experiments | Fig. 4에서 정책 성공률과 데이터 규모 효과, Appendix F에서 생성 성공률을 확인. |
| §7 Limitations / §8 Conclusion | 스킬 순서, 물체 pose, 시연 수집법, 실물 전이 관측의 제약. |
| Appendices | D는 clutter, E는 데이터 규모, F는 생성 성공률, G는 시작 자세 증강, K는 sim-to-real, L은 일반 teleoperation, N은 로봇 간 적응, O는 알고리즘 의사코드. |

## 13. Questions / Discussion

1. 생성 데이터와 정책 구조의 효과를 분리하려면, 동일한 정책 구조에서 MimicGen과 SkillGen 데이터를 어떤 조건으로 비교해야 할까?
2. 시작 자세에 도달한 뒤 실패하는 사례에서, 시작 자세 증강과 스킬 내부 교정은 각각 어떤 오류를 줄일 수 있을까?
3. HSP-Class에서 물체 pose에 오차가 있을 때, 그 오차가 경로 계획과 국소 조작 중 어느 쪽에 더 크게 영향을 줄까?

## 14. References / Related Links

- [SkillMimicGen Proceedings](https://proceedings.mlr.press/v270/garrett25a.html)
- [SkillGen Project Page](https://skillgen.github.io/)
- [MimicGen](https://mimicgen.github.io/)
- [HITL-TAMP](https://hitltamp.github.io/)
- [CCIL](https://arxiv.org/abs/2310.12972)
- [IntervenGen](https://arxiv.org/abs/2405.01472)