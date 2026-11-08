# DemoGen

## 0. Paper Information

- Title: DemoGen: Synthetic Demonstration Generation for Data-Efficient Visuomotor Policy Learning
- Authors: Zhengrong Xue, Shuying Deng, Zhenyang Chen, Yixuan Wang, Zhecheng Yuan, Huazhe Xu
- Year: 2025
- Venue: Robotics: Science and Systems (RSS 2025)
- Paper: https://arxiv.org/abs/2502.16932
- Code: https://github.com/TEA-Lab/DemoGen
- Project: https://demo-generation.github.io/
- Proceedings: https://roboticsproceedings.org/rss21/p157.html

---

## 1. One-line Summary

> 하나의 원본 시연에서 조작 궤적과 3D 점군을 새로운 물체 배치에 맞게 변환하여, 추가 로봇 실행 없이 공간적으로 다양한 관측–행동 데이터를 생성하는 시스템.

---

## 2. Why this Paper?

### Problem

- Visuomotor policy는 학습하지 않은 물체 위치에서 작업을 수행하는 spatial generalization에 한계가 있음.
- 작업 공간 전체에서 안정적으로 동작하려면 다양한 물체 배치를 포함하는 시연이 필요함.
- 정밀한 조작일수록 하나의 시연으로 대응할 수 있는 공간 범위가 좁아져 데이터 수집 부담이 커짐.
- MimicGen 계열은 변환한 궤적을 환경에서 실행하여 새로운 관측을 수집하므로, 실제 로봇에서는 생성 비용이 여전히 큼.

### Motivation

- 물체 위치만 달라졌다면 기존 시연의 조작 방식을 재사용할 수 있음.
- 행동뿐 아니라 해당 행동에 대응하는 시각 관측까지 합성하면 추가 로봇 실행을 줄일 수 있음.
- 3D 점군은 공간 좌표를 직접 변환할 수 있어 물체 위치 변화에 따른 관측 생성에 적합함.

### Main Contribution

- 행동과 관측을 함께 생성하는 fully synthetic demonstration generation pipeline 제안.
- 시연을 motion / skill 구간으로 분해하고, 구간별로 다른 행동 변환 적용.
- 3D 점군의 segment-and-transform을 통한 관측 합성.
- 작업당 하나의 원본 시연으로 시뮬레이션·실제 로봇에서 공간 일반화 성능 개선.
- Disturbance resistance와 obstacle avoidance를 위한 데이터 생성 확장 제시.

---

## 3. Key Concepts

### Visuomotor Policy

- 시각 관측과 로봇 상태를 입력받아 행동을 결정하는 정책.
- DemoGen 실험에서는 DP3를 사용.

```text
Point Cloud + Robot State
            ↓
       DP3 Policy
            ↓
End-Effector Pose + Hand Command
```

- DemoGen은 학습 데이터를 생성하는 방법.
- DP3는 생성된 데이터로 학습하여 실제 행동을 결정하는 모델.

### Spatial Generalization

- 학습 시 보지 못한 물체 위치·자세에서도 작업을 수행하는 능력.
- 물체 종류나 작업 자체가 바뀌는 일반화와는 구분해야 함.

### Object-Centric Segmentation

- 전체 시연을 특정 물체를 기준으로 하는 구간으로 분해.
- 물체에 대한 상대적인 조작 방식을 새로운 배치에서도 보존하기 위한 구조.

### Motion Segment

- 조작 구간으로 접근하거나 조작 구간 사이를 연결하는 이동.
- 물체를 잡은 채 이동하는 과정도 포함할 수 있음.
- 새로운 물체 배치에 맞게 경로를 다시 계획.

### Skill Segment

- 물체 근처에서 수행하는 접촉 기반 조작.
- 예: 꽃 잡기, 꽃병에 꽃 꽂기.
- 물체 배치 변화에 맞춰 궤적 전체를 함께 이동·회전.

### Task and Motion Planning (TAMP)

- 행동의 순서·조건을 다루는 task planning과 실제 이동 경로를 다루는 motion planning을 결합하는 접근.
- DemoGen에서는 시연의 작업 순서를 활용하면서 조작 구간과 연결 이동을 분리하는 원리를 사용.
- 작업 순서를 처음부터 탐색하는 전체 TAMP 시스템으로 해석할 필요는 없음.

### Invariance / Equivariance

- **Hand command — invariant:** 물체 위치가 바뀌어도 원본의 그리퍼·손가락 명령을 유지.
- **Arm command — spatially equivariant:** 물체 배치가 바뀌면 손끝 궤적도 그에 맞게 변경.

---

## 4. Method

### Overall Pipeline

```text
One Source Demonstration
            ↓
Point Cloud Pre-processing
            ↓
Object Segmentation + Motion / Skill Parsing
            ↓
Sample New Object Configuration
            ↓
Action Generation + Point Cloud Generation
            ↓
Synthetic Observation–Action Dataset
            ↓
Train DP3
            ↓
Closed-Loop Robot Execution
```

### Source Demonstration

- 각 시점의 observation–action pair로 구성.
- Observation: 점군 + 로봇 팔·손 상태.
- Action: 목표 end-effector pose + 그리퍼·손가락 명령.
- 목표는 원본과 다른 초기 물체 배치를 조건으로 새로운 시연을 생성하는 것.

### Point Cloud Pre-processing

- 단일 RGBD 카메라로 점군 획득.
- 배경·테이블의 불필요한 점 제거.
- Clustering으로 노이즈와 outlier 제거.
- Farthest point sampling으로 점 개수를 일정하게 조정.
- 첫 프레임에서 Grounded SAM으로 물체를 분할하고, depth와 연결해 3D 물체 점군 식별.

### Source Trajectory Segmentation

꽃을 꽃병에 꽂는 예시:

```text
Move to Flower → Pick Flower → Move to Vase → Insert Flower
     Motion          Skill         Motion          Skill
```

- 물체 점군 중심과 end-effector 사이의 거리를 기준으로 skill 구간 식별.
- 인접한 skill 구간 사이를 motion 구간으로 분류.
- 실제 접촉을 직접 측정하는 방식이 아니라 거리 임계값을 사용하는 근사적 구분.

### TAMP-Based Action Generation

| 구성 | 생성 방법 |
|---|---|
| Hand command | 원본 시점별 명령 유지 |
| Skill trajectory | 기준 물체의 공간 변환을 구간 전체에 적용 |
| Motion trajectory | 변환된 skill 구간들을 연결하도록 재계획 |

예시:

- 꽃이 오른쪽으로 이동하면 꽃을 잡는 궤적도 함께 이동.
- 꽃병이 다른 위치로 이동하면 꽂는 궤적도 꽃병에 맞게 이동.
- 두 조작 구간 사이의 운반 경로는 새롭게 생성.

> 물체에 대한 조작 방식은 보존하고, 조작 사이의 이동은 새로운 배치에 맞게 만든다.

### Point Cloud Observation Generation

- 행동만 바꾸면 새로운 장면에 대응하는 관측이 부족함.
- 물체·end-effector 점군을 구분하고, 행동 생성에 대응하는 공간 변환을 적용.
- 접촉 중인 부분은 함께 변환하여 상대적인 접촉 관계를 보존.
- 추가 로봇 rollout 없이 학습용 시각 관측 생성.

```text
New Object Configuration
        ↓
Consistent Action Transformation
        +
Corresponding Point Cloud Transformation
        ↓
Synthetic Observation–Action Pair
```

### Visuomotor Policy Learning

- 생성한 데이터로 DP3 학습.
- 입력은 점군과 proprioception.
- 학습된 정책은 실행 중 관측을 반복해서 받아 closed-loop으로 행동.
- 데이터 생성 단계의 궤적 변환과 실제 정책 실행을 구분해야 함.

### Augmentation for Disturbance Resistance (ADR)

- 작업 도중 물체가 이동하는 상황을 합성.
- 물체와 end-effector에 비동기적인 변환을 적용.
- 물체가 먼저 이동하고 손끝이 다시 접근한 뒤 작업을 이어가는 데이터를 구성.
- 정적인 초기 배치 증강을 넘어 외란 이후의 복구 행동을 학습하도록 유도.

### Obstacle Avoidance

- 원본 점군에 상자·원뿔 등 합성 장애물 점군 추가.
- Motion planner로 장애물을 우회하는 행동 궤적 생성.
- 장애물 관측과 우회 행동을 함께 학습 데이터에 포함.

---

## 5. Important Models / Datasets / Methods

| 구성요소 | 역할 |
|---|---|
| DP3 | 점군과 로봇 상태로 행동을 예측하는 visuomotor policy |
| Grounded SAM | 원본 RGB 이미지의 물체 분할 |
| Point Cloud Processing | 배경 제거, 노이즈 제거, downsampling |
| TAMP Principles | 조작 구간 재사용과 연결 이동 계획 |
| MetaWorld | 시뮬레이션 작업 구성의 기반 |
| MimicGen | 물체 중심 시연 변환·재생 기반 데이터 생성 |
| SkillMimicGen | 조작 구간을 변환하고 motion planning으로 연결하는 관련 방법 |

### Spatial Generalization Benchmark

- DP, DP3 및 사전학습 시각 표현을 사용하는 정책 비교.
- DP의 시각 인코더를 R3M, DINOv2, CLIP으로 교체하는 설정 포함.
- 3D 관측과 사전학습 표현이 도움이 되지만, 다양한 공간 배치를 포함하는 데이터의 필요성은 남음.

---

## 6. Strengths

- 작업당 하나의 원본 시연으로 다양한 물체 배치의 데이터 생성.
- 추가 로봇 실행 없이 관측과 행동을 함께 합성하여 생성 비용 절감.
- 디지털 트윈에서 전체 물리 과정을 재현하지 않고 실제 수집 데이터 활용.
- 단일 팔·양팔, 평행 그리퍼·다지 손 등 다양한 구성에서 평가.
- 강체뿐 아니라 변형체·유체를 포함하는 작업에서도 활용 가능성을 제시.
- 공간 일반화와 데이터 coverage의 관계를 실험적으로 분석.
- 외란 대응·장애물 회피처럼 필요한 행동을 데이터 생성 전략으로 추가.

---

## 7. Limitations

### Segmented Point Cloud Dependency

- 물체와 end-effector 점군을 구분할 수 있어야 함.
- 심한 가림, 복잡한 배경, 밀집된 물체 배치는 분할을 어렵게 만들 수 있음.

### Visual Mismatch

- 단일 카메라 점군은 원본 시점에서 보이는 표면만 포함.
- 물체를 새로운 위치로 옮기면 실제 카메라에서는 다른 표면이 보일 수 있음.
- 기존 점군을 변환하는 것만으로는 새롭게 보이는 표면을 생성할 수 없음.

```text
Synthetic Observation:
원본에서 보이던 표면을 새로운 위치로 이동

Real Observation:
새 위치의 시점·가림 관계에 따라 보이는 표면 자체가 달라짐
```

- 원본 배치와 멀어질수록 관측 불일치가 커질 수 있음.

### Limited Behavioral Diversity

- 기존 조작 방식과 작업 구조를 재사용하는 접근.
- 새로운 조작 전략이나 작업 순서를 자동으로 발견하는 방법은 아님.
- 공간 변환으로 유효한 시연을 만들 수 있는지가 적용 가능성을 좌우함.

### No Rollout-Based Validation

- 생성 과정에서 모든 궤적을 물리적으로 실행하여 성공을 검증하지 않음.
- 공간적으로 일관된 변환이 모든 배치의 물리적 성공을 보장하지는 않음.

### Task Suitability

- 주된 목표는 물체 배치 변화에 대한 일반화.
- In-hand reorientation 등 공간 배치 증강의 이점이 작은 작업에는 적합성이 낮음.

---

## 8. Trade-offs

| 선택 | 장점 | 남는 과제 |
|---|---|---|
| Fully synthetic generation | 빠르고 저렴한 데이터 생성 | 실행 기반 성공 검증 부재 |
| 단일 시점 점군 | 간단한 센서 구성과 직접적인 3D 편집 | 가림·새로운 표면에 따른 visual mismatch |
| 기존 skill 재사용 | 정교한 조작 패턴 보존 | 새로운 행동 전략의 다양성 제한 |
| 넓은 공간으로 증강 | 학습 coverage 확대 | 원본과 멀어질수록 관측 불일치 증가 |
| 생성 데이터 수 증가 | 초기 성능 개선 | 일정 규모 이후 성능 포화 |

---

## 9. Key Findings

### Simulation Results

- MetaWorld에서 구성한 8개 작업으로 평가.
- 작업당 원본 시연 1개 사용.
- 한 물체의 공간 배치를 바꾸는 작업은 100개, 두 물체를 다루는 작업은 200개의 합성 시연 생성.
- 비교 설정 간 학습에 사용하는 observation–action pair 처리량을 고정.

Table I의 3개 seed 평균 성공률 기준:

| Task | 1 Source | DemoGen |
|---|---:|---:|
| Pick-Cube | 0% | 73% |
| Button-Small | 4% | 84% |
| Drawer-Close | 50% | 100% |
| Faucet-Open | 23% | 92% |
| Handle-Press | 16% | 100% |
| Box-Lid | 11% | 95% |
| Stack-Cube | 0% | 77% |
| Assembly | 0% | 83% |
| Average | 13% | 88% |

- 논문 표는 최대값 / 평균값을 함께 보고하며, 위 표는 평균값만 사용.
- 10개 source 시연으로 학습한 설정의 평균 68%보다 높음.
- 25개 source 시연 설정의 평균 91%에 근접.
- 시뮬레이션 source 시연은 scripted policy로 준비하므로 인간 수집 시연과 구분해야 함.

### Real-World Results

- 3개 플랫폼의 8개 실제 작업에서 평가.
- 작업당 하나의 인간 시연으로 생성한 데이터를 활용.
- 평균 성공률 74.6% 보고.
- 단일 팔·양팔, 평행 그리퍼·다지 손 등 다양한 구성에서 효과 확인.

### Spatial Generalization

- 하나의 시연이 충분히 대응하는 공간 범위는 제한적.
- 정밀도가 높아질수록 유효 범위가 좁아짐.
- 작업 공간을 충분히 덮는 demonstration coverage가 중요.
- 3D 표현이나 사전학습 인코더만으로 공간 일반화 문제가 해결되지는 않음.

### Data Generation Efficiency

- 논문은 시연 하나를 생성하는 데 약 0.01초의 계산 비용을 보고.
- 원본 시연 수집·전처리·정책 학습까지 포함한 전체 비용과는 구분해야 함.

### Performance Saturation

- 생성 데이터의 공간 범위를 넓히면 초기에는 성공률이 개선.
- 일정 범위 이후에는 visual mismatch로 성능 향상이 포화.
- 공간 범위를 고정하고 데이터 밀도만 높여도 무한히 개선되지는 않음.

> 생성 데이터의 개수뿐 아니라 공간 coverage와 실제 관측과의 일치도가 중요하다.

### Out-of-Distribution Extensions

- ADR을 적용한 외란 대응에서 일반적인 공간 증강보다 높은 성능.
- 장애물 회피 실험에서 25회 중 22회 성공.
- 해당 능력은 외란·장애물을 반영한 데이터 생성 전략으로 학습시킨 결과.

---

## 10. Future Work / Open Challenges

### Directions Discussed in the Paper

- Contrastive learning이나 3D generative model을 통한 visual mismatch 완화.
- 추가 원본 시연을 활용하여 관측·행동 다양성 보완.
- 인간 데이터 수집 비용과 정책 성능 사이의 균형 탐색.

### Follow-up Questions

아래는 논문 결과를 바탕으로 정리한 후속 검증 과제.

- 어떤 새로운 배치에서 합성 시연의 품질이 크게 떨어지는가?
- 물리적 실행 없이 생성 데이터의 유효성을 어떻게 평가할 것인가?
- 다중 시점 관측이 visual mismatch를 얼마나 줄일 수 있는가?
- 제한된 예산에서 합성 데이터를 늘릴지, 새 원본 시연을 수집할지 어떻게 결정할 것인가?
- 양팔 협응과 복잡한 접촉 관계를 보존하려면 어떤 추가 제약이 필요한가?

---

## 11. My Takeaway

### What I Learned

- DemoGen의 핵심은 새로운 policy architecture가 아니라 저비용 데이터 생성.
- 행동 궤적과 시각 관측을 함께 바꿔야 학습 가능한 시연이 됨.
- 조작 구간은 물체 기준으로 보존하고, 연결 이동은 새롭게 계획할 수 있음.
- 데이터가 많아도 실제 관측과 차이가 크면 성능 개선에 한계가 있음.
- 정책의 공간 일반화는 모델뿐 아니라 데이터의 coverage와 품질에 크게 의존.

### Connection to Our Research

- VR teleoperation으로 수집한 소수 시연을 증강용 seed data로 활용하는 방향과 연결.
- 적용하려면 RGBD 관측, end-effector pose, 손 명령, 실제 로봇 상태를 시간 정렬하여 기록해야 함.
- RGB 이미지나 손 추적 기록만으로 DemoGen pipeline을 바로 적용할 수 있는 것은 아님.
- 초기에는 물체 위치만 바뀌는 단순 grasp / place 작업에서 효과를 확인하는 것이 적절.
- 양팔·Inspire 손 적용에서는 좌표계, 명령 정의, 접촉 구간과 양팔 상대 관계를 추가로 검토해야 함.
- 평가는 동일한 policy에서 원본 시연만 사용한 경우와 증강 데이터를 사용한 경우를 비교.

### Connection to Previous Papers

| 논문 | 핵심 역할 | DemoGen과의 연결 |
|---|---|---|
| OPEN TEACH | 인간 시연 수집 | 증강에 사용할 원본 데이터 확보 |
| MimicGen | 물체 중심 궤적 변환·환경 실행 | 실행을 통해 관측을 수집하는 방식과 비교 |
| SkillMimicGen | Skill 변환과 motion planning 연결 | 행동 생성의 구조적 아이디어 공유 |
| DexMimicGen | 양팔·다지 손 시연 생성 | 양팔 협응과 순서 제약 관점에서 비교 |

- CCIL과의 구체적인 비교·결합 가능성은 해당 논문의 생성 방식과 가정을 확인한 뒤 정리.

### Next Action

- [ ] RGBD·팔·손 데이터의 기록 항목과 시간 정렬 방식 정리.
- [ ] 물체 중심 motion / skill 분할이 가능한 작업 선정.
- [ ] 원본 점군에서 물체·end-effector 분할 가능 여부 확인.
- [ ] Action과 observation에 대응하는 변환 적용.
- [ ] 동일한 DP3에서 원본 데이터와 증강 데이터 비교.
- [ ] 작업 공간 전체의 위치별 성공률 평가.
- [ ] 원본에서의 거리와 visual mismatch·성공률의 관계 확인.
- [ ] 양팔 협응·손 접촉을 보존하는 추가 조건 검토.

---

## 12. Section Notes

<details>
<summary>Section-by-Section Reading Notes</summary>

### I. Introduction

- Visuomotor policy의 데이터 요구량과 인간 시연 수집 비용이 핵심 문제.
- 기존 시연 변환 방식의 실제 로봇 rollout 비용에 주목.
- Action과 point cloud observation을 함께 합성하는 pipeline 제안.

### II. Related Works

- Visuomotor policy learning.
- 소수 시연 기반 imitation learning과 TAMP 활용.
- MimicGen 계열의 데이터 생성 방식.
- DemoGen은 합성 데이터를 생성한 뒤 closed-loop policy를 학습.

### III. Spatial Generalization of Visuomotor Policies

- 단일 시연의 공간적 유효 범위를 분석.
- 정밀한 작업일수록 더 많은 위치의 데이터 필요.
- DP, DP3, 사전학습 시각 인코더를 사용하는 설정 비교.
- 표현 개선과 함께 데이터 coverage가 필요함을 확인.

### IV. DemoGen Methods

- 원본 시연의 observation–action pair 정의.
- RGBD 점군 전처리와 물체 분할.
- 거리 기반 motion / skill 구간 식별.
- Skill 변환과 motion 재계획.
- 점군 편집을 통한 대응 관측 생성.

### V. Simulated Experiments

- 하나의 source 시연으로 공간적으로 다양한 데이터 생성.
- 8개 작업에서 DP3 학습 성능 비교.
- Visual mismatch와 데이터 증가에 따른 성능 포화 분석.

### VI. Real-World Experiments: Spatial Generalization

- 3개 플랫폼, 8개 실제 작업에서 평가.
- 단일 팔·양팔 및 다양한 end-effector 구성 활용.
- 평균 성공률 74.6% 보고.

### VII. Real-World Experiments: Extensions

- ADR로 작업 중 물체 이동과 재접근 과정 합성.
- 합성 장애물 점군과 우회 경로를 활용한 obstacle avoidance.
- 필요한 일반화 능력을 데이터 생성 전략으로 학습.

### VIII. Conclusion / Appendix

- 저비용 행동·관측 합성으로 실제 로봇 데이터 수집 부담 완화.
- 점군 분할 의존성과 visual mismatch가 주요 한계.
- 추가 source 시연과 관측 불일치 완화가 후속 과제.

</details>

---

## 13. Questions / Discussion

- 우리 작업을 물체 중심 motion / skill 구간으로 나눌 수 있는가?
- 물체 중심과 손끝 거리만으로 조작 구간을 안정적으로 구분할 수 있는가?
- 손 명령을 그대로 유지해도 유효한 물체 배치의 범위는 어디까지인가?
- 양팔이 동시에 접촉하는 구간에서는 어떤 점군과 궤적을 함께 변환해야 하는가?
- 생성 궤적이 로봇의 도달 범위·관절 제약을 만족하는지 어떻게 확인할 것인가?
- Visual mismatch가 커지는 위치를 어떻게 탐지할 것인가?
- 생성 데이터의 양보다 source 시연의 위치 다양성을 늘리는 것이 유리한 시점은 언제인가?
- 실제 로봇 실행 없이 합성 데이터의 품질을 판단할 수 있는가?

---

## 14. References / Related Links

- [DemoGen Paper](https://arxiv.org/abs/2502.16932)
- [DemoGen Code](https://github.com/TEA-Lab/DemoGen)
- [DemoGen Project](https://demo-generation.github.io/)
- [RSS Proceedings](https://roboticsproceedings.org/rss21/p157.html)
- [MimicGen](https://arxiv.org/abs/2310.17596)
- [SkillMimicGen](https://arxiv.org/abs/2410.18907)
- [DexMimicGen](https://arxiv.org/abs/2410.24185)
- [DP3](https://3d-diffusion-policy.github.io/)
- [OPEN TEACH](https://arxiv.org/abs/2403.07870)