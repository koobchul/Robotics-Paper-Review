# CP-Gen: Constraint-Preserving Data Generation for Visuomotor Policy Learning

## 0. Paper Information

- Title: Constraint-Preserving Data Generation for Visuomotor Policy Learning
- Authors: Kevin Lin, Varun Ragunath, Andrew McAlinden, Aaditya Prasad, Jimmy Wu, Yuke Zhu, Jeannette Bohg
- Year: 2025
- Venue: Conference on Robot Learning (CoRL 2025)
- Paper: [arXiv](https://arxiv.org/abs/2508.03944) · [Proceedings](https://proceedings.mlr.press/v305/lin25b.html)
- Code: [kevin-thankyou-lin/cpgen](https://github.com/kevin-thankyou-lin/cpgen)
- Project: [CP-Gen Project Page](https://cp-gen.github.io/)
- Note: 제목은 첨부한 arXiv v1 기준. PMLR 논문집 제목은 “Constraint-Preserving Data Generation for One-Shot Visuomotor Policy Generalization”.

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

- Geometry generalization vs. pose generalization:
- Actor keypoint:
- Target keypoint trajectory:
- Keypoint-trajectory constraint:
- Constraint-preserving transformation:
- Joint configuration optimization:

---

## 4. Method

### Overall Pipeline

-

### Source Demonstration & Skill Segmentation

- 자유 공간 이동과 skill 구간을 어떻게 구분하는가?
- 원본 시연 외에 사람이 제공해야 하는 정보는 무엇인가?

### Keypoint-Trajectory Constraint Formulation

- actor keypoint와 target trajectory는 각각 어디에 정의되는가?
- 로봇이 물체를 잡고 있을 때 제약은 어떻게 표현되는가?

### Object Pose & Geometry Transformation

- 물체의 위치·자세·크기·형상을 어떻게 변형하는가?
- 물체 변형에 맞춰 keypoint와 참조 궤적을 어떻게 바꾸는가?

### Robot Joint Configuration Optimization

- 최적화 변수:
- 목적함수:
- 제약조건:
- 해를 구하지 못하는 경우의 처리:

### Motion Planning & Demonstration Generation

- skill 시작점까지의 충돌 없는 경로를 어떻게 생성하는가?
- 생성한 시연의 유효성과 성공 여부를 어떻게 검증하는가?

### Visuomotor Policy Learning & Deployment

- 정책 입력:
- 행동 표현:
- 학습 알고리즘:
- 배포 시 필요한 정보:

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

### Demonstration Generation Results

-

### Pose Generalization

-

### Geometry Generalization

-

### Comparison with Baselines

-

### Real-Robot & Zero-Shot Sim-to-Real Results

-

### Ablation Studies

-

---

## 10. Future Work / Open Challenges

-

---

## 11. My Takeaway

### What I Learned

-

### Connection to Our Research

- 소수의 VR 시연에 추가로 필요한 keypoint·skill annotation:
- 현재 robosuite 과제에서 적용 가능한 물체 형상 변화:
- 제약을 만족하는 시연 생성과 실행 중 오류 복구의 차이:
- CCIL을 적용할 때 형상 변화에 따른 동역학 차이를 처리할 방법:
- 생성 성공률, 정책 일반화, 외란 후 복구율을 구분할 평가 기준:

### Connection to Previous Papers

- **MimicGen:** 물체 중심의 궤적 변환으로 다루기 어려운 형상 변화를 CP-Gen은 어떻게 처리하는가?
- **SkillMimicGen:** skill 변환·연결 방식과 배포 정책 구조는 어떻게 다른가?
- **CCIL:** 기하학적 제약을 보존한 시연 주변에서도 교정 라벨이 추가로 필요한 상황은 무엇인가?
- **IntervenGen:** 형상·배치 일반화와 정책 실수에서의 복구를 어떻게 구분해 비교할 것인가?
- **DemoGen:** 행동과 관측의 합성 방식 및 환경 실행 요구사항은 어떻게 다른가?

### Next Action

- [ ] 과제 하나를 골라 actor keypoint와 target trajectory를 직접 표시
- [ ] MimicGen / SkillMimicGen / CP-Gen의 시연 변환·연결·정책 구조 비교
- [ ] 물체 배치 변화와 형상 변화를 분리한 평가 조건 설계
- [ ] CCIL을 결합할 후보 구간과 국소 동역학 가정의 유효 범위 정리

---

## 12. Section Notes

### Abstract

-

### Introduction

-

### Related Work

-

### Problem Formulation / Preliminaries

-

### CP-Gen Method

-

### Experiments

-

### Conclusion / Limitations

-

### Appendix / Supplementary Material

-

---

## 13. Questions / Discussion

-

---

## 14. References / Related Links

-