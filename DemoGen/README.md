# DemoGen: Synthetic Demonstration Generation for Data-Efficient Visuomotor Policy Learning

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

-

---

## 4. Method

### Overall Pipeline

-

### Source Demonstration & Task Segmentation

-

### Action Trajectory Generation

-

### Point Cloud Observation Generation

-

### Visuomotor Policy Learning

-

### Augmentation for Disturbance Resistance

-

### Obstacle Avoidance

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

### Spatial Generalization

-

### Data Generation Efficiency

-

### Ablation Studies

-

### Out-of-Distribution Evaluation

-

---

## 10. Future Work / Open Challenges

-

---

## 11. My Takeaway

### What I Learned

-

### Connection to Our Research

- VR로 수집한 소수의 demonstration에 적용할 수 있는 부분:
- 현재 로봇 구성에 적합한 task:
- 합성 observation과 action의 일관성을 확보하는 방법:
- 생성 데이터의 품질과 정책 일반화를 평가하는 방법:
- 실제 환경에 적용하기 위해 필요한 추가 구성:

### Connection to Previous Papers

- MimicGen / DexMimicGen과의 공통점:
- MimicGen / DexMimicGen과의 차이점:
- CCIL과 비교하거나 결합할 때 검토할 부분:

### Next Action

- [ ] 원본 demonstration에 필요한 정보와 전처리 정리
- [ ] action과 observation의 생성 과정을 각각 정리
- [ ] 생성 과정에서 필요한 가정과 적용 한계 확인
- [ ] MimicGen과 데이터 생성 비용 및 검증 방식 비교
- [ ] 현재 연구에 적용할 task 후보 정리

---

## 12. Section Notes

### Abstract

- low-cost, fully synthetic approach for automatic demonstration generation 
- using only one human-collected demonstration per task 
- generate spatially augmented demonstrations by adapting the demonstrated action trajectory to novel object configuration
- visual observations are synthesized by leveraging 3D point clouds as the modality and rearranging the subjects in the secne via #D editing
- can be extended to enable additional out-of-distribution capabilities, including disturbance resistance and obstacle avoidance

### Section 1.

- DemoGen adopts a fully synthetic pipeline that efficiently concretizes the generated plans into
spatially augmented demonstrations ready for policy training.

- For action generation, DemoGen develops the MimicGen strategy by incorporating techniques from Task and Motion Planning (TAMP), similar to the practice in the recently released SkillMimicGen 

- it selects point clouds as the observation modality and synthesizes the augmented visual observations through 3D editing.

- we manifest the effectiveness of DemoGen by
evaluating the performance of visuomotor policies trained on DemoGen-generated datasets from only one human collected demonstration per task.

-  With such minimal cost, DemoGen significantly enhances policy performance, generalizing to un-demonstrated configurations and achieving an average of 74.6% across 8 real-world tasks.

### Section 2.

#### Visumotor Policy Learning

- Represented by Diffusion Policy [7] and its extensions, visuomotor policy learning refers to the imitation learning methods that learn to predict actions directly from visual observations in an end-to-end fashion. 

#### Data-Efficient Imitation Learning

- Attempting to develop manipulation policies from only a handful of demonstrations, data-efficient imitation learning methods often build on the principles of Task and Motion
Planning (TAMP), while incorporating imitation learning to replace some components in the TAMP pipeline.

-  DemoGen leverages the TAMP principles for
synthetic data generation. Subsequently, the synthetic demonstrations are used to train closed-loop visuomotor policies for
task resolution.

#### Data Generation for Robotic Manipulation

- Building upon MimicGen and its extensions, DemoGen incorporates their strategies for generating execution plans, but replaces the expensive on-robot rollouts with an efficient,
fully synthetic generation process. This enables DemoGen to generate real-world demonstrations ready for policy training in a cost-effective manner.

### Section 3.

- Spatial generalization: ability of a policy to
perform tasks involving objects placed in configurations that were not seen during training. 

#### Visualization of Spatial Effective Range

- We evaluate a Button-Large task adapted from the
MetaWorld benchmark, where the robot approaches a button and presses it down.

- We adopt 3D Diffusion Policy (DP3) [59] as the studied policy, as our benchmarking results indicate that 3D observations provide superior spatial generalization compared to 2D approaches.

- Thus, to train a policy that generalizes well across the entire object randomization range, demonstrations must cover the full workspace,
resulting in substantial data collection costs. 

- as task precision requirements increase, the effective range shrinks to more localized areas, necessitating a greater number of demonstrations to adequately cover the workspace.

#### Benchmarking Spatial Generalization Capabilitty

- 

### Section 4.

-

### Section 5.

-

### Section 6.

-

### Appendix / Supplementary Material

-

---

## 13. Questions / Discussion

-

---

## 14. References / Related Links

-