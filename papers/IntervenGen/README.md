# IntervenGen: Interventional Data Generation for Robust and Data-Efficient Robot Imitation Learning

## 0. Paper Information

- Title: IntervenGen: Interventional Data Generation for Robust and Data-Efficient Robot Imitation Learning
- Authors: Ryan Hoque, Ajay Mandlekar, Caelan Garrett, Ken Goldberg, Dieter Fox
- Year: 2024
- Venue: IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS 2024)
- Paper: [arXiv](https://arxiv.org/abs/2405.01472) · [IEEE Xplore](https://doi.org/10.1109/IROS58592.2024.10801523)
- Code: 공개된 공식 구현 링크는 확인되지 않음
- Project: [IntervenGen Project Page](https://sites.google.com/view/intervengen2024)

---

## 1. One-line Summary

> 이 논문을 한 문장으로 요약하면?

-

---

## 2. Why this Paper?

### Problem

-

### Motivation

-

### Main Contribution

-

---

## 3. Key Concepts

- Distribution shift:
- Interactive imitation learning / DAgger:
- Human intervention:
- Mistake trajectory vs. recovery trajectory:
- Object pose estimation error:

---

## 4. Method

### Overall Pipeline

-

### Interventional Data Collection

- 원본 시연과 사람의 개입 데이터는 각각 어떻게 수집하는가?
- 개입 데이터에서 어떤 구간을 복구 궤적으로 사용하는가?

### Mistake Generation: Closed-Loop Policy Execution

- 어떤 정책을 실행하고, 어떤 방식으로 실수 상태를 만드는가?
- 어디에서 정책 실행을 멈추고 복구를 시작하는가?

### Recovery Generation: Open-Loop Trajectory Replay

- 수집한 복구 궤적을 새로운 실수 상태에 어떻게 맞추는가?
- 생성된 복구 궤적의 성공 여부는 어떻게 확인하는가?

### Policy Training with Generated Interventions

-

### Inter-Subtask Recovery & Offline Mode

-

---

## 5. Important Models / Datasets / Methods

-

---

## 6. Strengths

-

---

## 7. Limitations

-

---

## 8. Trade-offs

-

---

## 9. Key Findings

### Main Results

-

### Human Intervention Budget

-

### Robustness to Object Pose Estimation Error

-

### Simulation vs. Real-Robot Results

-

### Ablation / Analysis

-

---

## 10. Future Work / Open Challenges

-

---

## 11. My Takeaway

### What I Learned

-

### Connection to Our Research

- **연구 질문:** MimicGen으로 생성한 성공 시연에 CCIL 교정 라벨을 추가하면, 별도의 사람 개입 시연 없이도 실행 중 이탈 상태에서 복구할 수 있을까?
- **IntervenGen과 비교할 핵심:** 실수 상태를 어디에서 가져오는지, 교정 행동을 어떤 정보로 만드는지, 복구에 필요한 사람 개입량은 얼마인지.
- **차별화 가설:** 사람의 복구 궤적을 변환·재생하는 방식과, 시연 주변에서 학습한 동역학으로 교정 행동을 계산하는 방식이 각각 어떤 이탈을 복구할 수 있는가?
- **비교 조건:** 같은 원본 시연, 정책 구조, 평가 환경에서 학습 데이터 규모와 사람 개입량을 각각 기록.
- **평가 지표:** 외란 없는 과제 성공률, 외란 후 복구율, 외란 후 최종 과제 성공률.

### Connection to Previous Papers

- **MimicGen / DexMimicGen:** 새로운 물체 배치에서 성공 시연을 생성하는 것과, 정책이 실제로 저지른 실수에서 복구 데이터를 생성하는 것은 어떻게 다른가?
- **CCIL:** 시연 근처에서 생성한 교정 라벨이 IntervenGen의 복구 궤적을 대체할 수 있는 범위는 어디까지인가?
- **DemoGen:** ADR에서 합성하는 외란 상황과 IntervenGen이 정책 실행으로 얻는 실수 상황은 어떻게 다른가?

### Next Action

- [ ] 실수 생성 → 사람 개입 수집 → 복구 궤적 생성의 흐름을 예시 하나로 정리
- [ ] IntervenGen과 CCIL의 입력 데이터, 교정 방법, 적용 범위를 표로 비교
- [ ] robosuite 과제 하나에서 같은 외란을 주고 `MimicGen`과 `MimicGen + CCIL`을 비교할 평가 조건 설계
- [ ] 사람 개입을 확보할 수 있다면, 개입량을 명시한 IntervenGen 비교군의 구현 범위 정리

---

## 12. Section Notes

### Abstract

- We propose IntervenGen (I-Gen), a novel data
generation system that can autonomously produce a large set of corrective interventions with rich coverage of the state space from a small number of human interventions. 

- We apply I-Gen to 4 simulated environments and 1 physical environment with object pose estimation error and show that it can increase policy robustness by up to 39× with only 10 human interventions.

### Section I. Introduction

- RQ: do we actually need to have a human operator collect corrections every single time a policy makes a mistake?

- Inspired by MimicGen, we propose a data generation system for interventional data

- Our system can be applied to a broad range of applications such as improving policy success rates on a task of interest, making policies robust to errors in perception, and more broadly, acting as a domain randomization [14] procedure to aid in
sim-to-real transfer of IL policies without requiring additional data collection from a human supervisor

- focus: improving policy robustness to errors in perception

### Section II. Related Work

- Data Collection Approaches for Robot Learning
- IL from Human Demonstrations
- Ineractive IL
- Policy Adaptation under Domain Shift
- MimicGen 
    - 1) generates interventional data rather than full demonstrations
    - 2) relaxes the assumption of precise object pose knowledge, which is critical to MimicGen’s success
    - 3) integrates closed-loop policy execution that allows the robot to visit novel states during the data generation process, 
    - 4) allows variation in not just object poses but also robot belief states about these object poses.


### Section III. Preliminaries

-

### Section IV. IntervenGen Method

- 1) trains a policy πθ on the current dataset; 
- 2) rolls out πθ for interventional data collection with human teleoperation (Sec. IV-A); 
- 3) synthesizes new intervention with closed-
loop policy execution and open-loop trajectory replay (Sec. IV-B and Sec. IV-C); 
- 4) returns the new synthetic dataset.

- Interventional Data Collection
    - consider human-gated interventions [10], in which the human monitors the robot policy execution and intermittently takes control to correct policy mistake

- Mistake Generation
    - aim to use the collected human interventions
    to automatically synthesize interventions for new scene configurations.
    - We use this robot policy during the data generation process to broaden the distribution of visited mistake states.

- Recovery Generation
    - 1) transforming the source trajectory to the current object pose
    - 2) linearly interpolating in end-effector space to the beginning of the transformed trajectory 
    - 3) executing the transformed trajectory open-loop

- Output Filtering and Dataset Aggregation
    - Each filtered episode of synthetic data is aggregated into the base dataset D (used to train the base policy πθ ), and the policy is retrained on the new dataset after data generation

- Inter-Subtask Recovery & Offline Mode:
    - The I-Gen framework accommodates additional modules not considered in the main set of experiments that greatly increases its range of applications, including 
        - 1) policy recovery from more severe failure modes that revert to earlier subtasks
        - 2) “offline” I-Gen, which allows humans to demonstrate mistakes intentionally


### Section V. Experiment Setup

- Tasks & environments: 
    - consider 4 tasks in the MuJoCo [42] robosuite simulation environment and 1 physical experiment.
- Baselines:
- Evaluation protocol:

### Section VI. Experiments

- Main results:
- Analysis:
- Real-robot evaluation:

### Section VII. Conclusion

-

### Appendix / Supplementary Material

-

---

## 13. Questions / Discussion

-

---

## 14. References / Related Links

-