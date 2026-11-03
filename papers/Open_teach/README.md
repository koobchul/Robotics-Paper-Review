# Open Teach

## 0. Paper Information

- Title: OPEN TEACH: A Versatile Teleoperation System for Robotic Manipulation
- Authors: Aadhithya Iyer, Zhuoran Peng, Yinlong Dai, Irmak Güzey, Siddhant Haldar, Soumith Chintala, Lerrel Pinto
- Year / Venue: 2024 / CoRL 2024 (논문집 출판 연도와 구분)
- Paper: https://arxiv.org/abs/2403.07870
- Code: https://github.com/aadhithya14/Open-Teach
- Project: https://open-teach.github.io/

## 1. One-line Summary

> Quest 3의 손 추적과 시각 피드백으로 다양한 로봇을 원격조작하고, 정책 학습에 사용할 시범 데이터를 수집하는 오픈소스 시스템.

## 2. Why this Paper?

### Problem

- 정교한 조작에는 전문가 시범이 필요하지만, 수집 장비가 비싸거나 특정 로봇에 종속되는 경우가 많음.
- 손과 팔을 함께 직관적으로 제어하면서 여러 로봇에 적용하기 어려움.

### Motivation

- 별도의 장갑이나 로봇과 동일한 조작 장치 없이 손동작으로 제어하고 싶음.
- 사람이 로봇의 움직임을 즉시 보고 수정할 수 있도록 빠른 추적과 시각 피드백이 필요함.

### Main Contribution

- 다지 손, 양팔, 이동형 로봇 및 시뮬레이션을 지원하는 원격조작 프레임워크.
- 38개 작업에서 활용 가능성을 보이고, 10개 작업의 정책 학습으로 데이터 유용성을 평가.
- VR 앱, 인터페이스, 제어기를 포함한 공개 시스템을 지향.

## 3. Key Concepts

- **Teleoperation:** 사람이 실시간으로 로봇을 조작.
- **Retargeting:** 사람 손·손목 동작을 로봇 구조에 맞는 목표값으로 변환.
- **BC:** 관측–행동 쌍으로 전문가 행동을 예측하는 지도학습. 성공 데이터 선별 자체를 뜻하지 않음.
- **Residual RL:** 기본 행동에 더할 보정 행동을 환경과의 상호작용으로 학습.
- **FISH:** 시범 기반 기본 정책에 보정 정책을 더하고, 시각적 시범 유사도로 보상을 구성.
- **TAVI:** 시각과 촉각으로 보정 행동을 결정하고, 시각 기반 보상으로 학습.
- **GMM head:** 여러 가우시안 성분으로 행동분포를 표현하는 출력부. 단일 가우시안보다 여러 행동 후보를 표현하기 좋음.
- **Action chunking:** 한 번에 여러 시점의 행동을 예측하는 방식.

## 4. System / Method

### Overall Pipeline

Quest 손 추적 → 키포인트 전송 → 좌표 변환 및 retargeting → 로봇별 제어 → 시각 피드백 및 데이터 기록.

### Hardware Configuration

| 환경 | 구성 |
|---|---|
| Franka-Allegro | Franka + Allegro + Xela 촉각 센서 |
| Kinova-Allegro | Kinova Jaco + Allegro + Xela 촉각 센서 |
| Bimanual | xArm7 × 2 + 2지 그리퍼 |
| Stretch | 이동형 로봇 + 2지 그리퍼 |
| Allegro Sim | 공간에서 자유롭게 움직이는 Allegro 손 |
| LIBERO Sim | Franka + 2지 그리퍼 |

### Hand Tracking & Pose Estimation

- Quest 3의 내장 손 추정기를 사용하고, 최대 90Hz로 손 자세를 전달
- 외부 다중 카메라 시스템에 필요한 별도 보정 절차를 줄임
- Quest 2 + WiVRn/OpenXR에서 같은 품질·주기가 나오는지는 별도 확인 필요

### Coordinate Mapping & Retargeting

- 팔: 손목과 손가락 뿌리 키포인트로 손 좌표계를 만들고, 손목 위치와 자세 변화를 말단 동작에 매핑
- Allegro 손: 손가락 관절각을 대응시키되, 엄지는 끝점 위치와 IK를 활용
- 2지 그리퍼: 손가락 pinch 제스처로 개폐 상태 전환
- 우리 Inspire 손은 Allegro와 구조가 다르므로 별도 매핑 필요

### Robot Control

- xArm: 공식 xArm API
- Franka: Deoxys controller
- Kinova: 선행연구의 공개 제어기
- Allegro: ROS 기반 비동기 제어
- 손 추적 주기, 명령 전송 주기, 하위 제어 주기를 구분해서 읽어야 함
- pause 및 동작 해상도 조절 기능으로 자세 재정렬과 미세 조작 지원

### Demonstration Collection

- 원격조작 중 로봇과 센서 데이터를 기록하고, 이후 정책 학습에 활용.
- Franka-Allegro에서는 시각·촉각, Allegro Sim과 LIBERO에서는 시각 데이터를 사용.
- 양팔 세팅은 5대 카메라로 데이터를 수집.
- 우리 시스템에서는 관측·실행 명령·실제 로봇 상태의 기록 항목과 시간 정렬을 명시해야 함.

## 5. Important Models / Datasets / Methods

| 학습 실험 | 사용 방법 | 의미 |
|---|---|---|
| Franka-Allegro | TAVI | 시각·촉각을 활용한 정책 학습 |
| Allegro Sim | FISH | 시각 기반 정책 학습 |
| LIBERO Sim | Transformer BC + GMM head + action chunking | 시범 행동을 이용한 정책 학습 |

TAVI/FISH는 OPEN TEACH의 필수 구성요소가 아니라, 수집한 데이터의 유용성을 평가하는 학습 방법.

## 6. Strengths

- 여러 로봇 형태에 적용하는 공통 수집 구조.
- 손 추적과 실시간 시각 피드백을 결합해 조작자가 오류를 수정하기 쉬움.
- 단순 조작 시연을 넘어 정책 학습 결과도 제시.
- 연구실 시스템과 비교할 수 있는 공개 구현 제공.

## 7. Limitations

- 손가락 가림과 자세 추정 오류가 제어 품질에 영향을 줌.
- pinch 인식 오류는 그리퍼 제어에도 영향을 줄 수 있음.
- 로봇별 retargeting과 제어기 연결은 필요함. 모든 로봇에 즉시 적용된다는 뜻은 아님.
- 양팔 원격조작 시연과 양팔 자율 정책의 학습 검증은 구분해야 함.

## 8. Trade-offs

| 선택 | 장점 | 남는 과제 |
|---|---|---|
| 자유로운 손동작 입력 | 별도 외골격 없이 직관적 입력 | 로봇의 도달 범위와 관절 제약을 직접 체감하기 어려움 |
| 비전 기반 손 추적 | 손에 장비를 착용할 필요가 적음 | 가림·인식 오류 |
| 다양한 로봇 지원 | 공통 프레임워크 재사용 | 형태별 매핑과 제어기 구현 |
| 시범 기반 residual RL | 적은 시범에서 보정 학습 가능 | 추가 환경 상호작용 필요 |

## 9. Key Findings

### Teleoperation Performance

- 38개 작업에서 다지 손·양팔·이동형 조작 등을 시연.
- AnyTeleop과의 사용자 비교는 팔 retargeting 코드 공개 범위 문제로 팔 이동이 필요 없는 두 작업에 제한됨.

### Demonstration Quality

- 수집한 데이터로 정책을 학습해 유용성을 평가.
- 여러 학습 방법을 사용했으므로 결과를 데이터 수집 시스템만의 효과로 분리해서 해석하기는 어려움.

### Robot Learning Results

- TAVI/FISH를 사용한 작업: 평균 성공률 82%.
- LIBERO BC 작업: 평균 성공률 93%.
- 전체 10개 학습 작업: 평균 성공률 86%.
- 이 수치를 양팔 세팅의 자율 작업 성공률로 해석하면 안 됨.

## 10. Future Work / Open Challenges

아래는 논문 한계와 우리 장비를 바탕으로 정리한 후속 검증 과제.

- 가림·추적 유실 이후 안정적인 복구.
- Inspire 손에 맞는 retargeting과 촉각 활용.
- 두 팔의 좌표계 정렬, 상대 위치, 명령 시간 동기화.
- 원격조작 데이터가 자율 정책 성능으로 이어지는지 검증.

## 11. My Takeaway

### What I Learned

- 원격조작 시스템과 정책 학습 알고리즘은 구분해서 봐야 함.
- 높은 주기 자체보다 입력부터 실제 동작·시각 피드백까지의 지연과 안정성이 중요함.
- BC와 residual RL은 시범 데이터를 활용하는 방식이 다름.

### Connection to Our Research

- 현재 시스템: Quest 2 + WiVRn/OpenXR 기반 양팔 원격조작 및 손 추적.
- 로봇 구성: xArm7 × 2 + UFACTORY Gripper + Inspire RH56E2 Hand.
- 논문의 양팔은 양쪽 모두 그리퍼. 우리는 손과 그리퍼를 함께 사용하는 구성.
- **비교할 부분:** 좌표계, 위치·회전 매핑, 주기·지연, pause, 해상도 조절, 기록 방식.
- **적용할 부분:** 양팔 수집 구조와 사용자 피드백 설계. 손 매핑은 Inspire에 맞게 검토.
- **추가 검증:** 촉각 데이터의 종류·접근 방법·주기, 손 관절 명령과 실제 관절 움직임의 관계.
- Quest 하드웨어만 비교하지 말고, 추정 → WiVRn/OpenXR 전달 → 로봇 명령으로 이어지는 경로를 확인해야 함.

### Connection to Previous Papers

- **AnyTeleop:** 여러 팔·손에 적용하는 범용 원격조작이라는 방향을 공유. 추적과 피드백, 배포 범위를 비교.
- **Lucid-XR:** 현재 시스템과 좌표계·전달 경로·지연·매핑을 비교할 항목으로 남김. 세부 비교는 원문/구현 확인 후 작성.
- **MimicGen / DexMimicGen:** 시범 수집 이후 데이터 생성·확장 단계와 연결. OPEN TEACH 자체가 자동 시범 증강 시스템인 것은 아님.
- **FISH / TAVI:** 수집한 시범을 기본 행동으로 활용하고, 온라인에서 보정하는 학습 방법.

### Next Action

- [ ] OPEN TEACH의 팔 좌표 변환과 현재 구현을 나란히 비교.
- [ ] Quest 2 → WiVRn/OpenXR → 로봇의 데이터 경로, 주기, 지연을 측정.
- [ ] Inspire 관절 매핑과 촉각 데이터 접근 방법 확인.
- [ ] 같은 작업에서 반복 가능한 시범과 동기화된 기록 확보.
- [ ] 자율 학습 목표를 정한 뒤 BC 또는 residual RL 실험 설계.

## 12. Section Notes

### Abstract / Introduction

- 핵심 동기: 학습에 필요한 시범을 저렴하고 직관적으로 수집.
- 외골격 방식은 로봇과의 기구학적 대응이 좋지만, 장비 비용과 형태 종속성이 존재.
- OPEN TEACH는 VR 손 추적과 피드백으로 다양한 형태를 지원하려는 접근.

### Related Work

- 로봇 전용 시스템과 범용 시스템의 차이를 중심으로 읽기.
- AnyTeleop의 방향을 계승하면서 실제 사용성과 적용 범위를 확장.

### Background on Imitation Learning

- BC: 전문가 행동의 likelihood 최대화.
- 고정 분산의 등방성 가우시안을 가정하면 제곱오차 최소화로 연결.
- residual policy: 기본 정책의 행동에 보정량을 더함.
- OT 기반 시범 매칭으로 학습 보상을 구성.

### Open Teach

- Quest 3 제품 사양보다 손 키포인트, 좌표계, 전송 주기, retargeting 구조에 집중.
- Quest 2에 적용할 때 동일하게 성립하는 부분은 직접 확인.

### Experimental Evaluation

- RQ1: 여러 로봇 세팅에 적용 가능한가?
- RQ2: 데이터로 학습한 정책이 성공하는가?
- RQ3: 복잡하고 긴 작업을 원격조작할 수 있는가?
- RQ4: 초보자가 사용하기 쉬운가?
- 각 실험이 어느 질문에 답하는지 구분해서 해석.

### Limitations / Appendix

- 손 추적 오류와 가림 문제 확인.
- Appendix A: 엄지 매핑, pause, 해상도 조절 세부 내용 재확인.
- Appendix B: 우리 양팔 작업과 비교할 작업 목록 확인.
- 사용자 실험은 과제·대상·비교 범위를 함께 기록.

## 13. Questions / Discussion

- 우리 입력은 어떤 손 좌표계와 관절 정의를 사용하는가?
- 위치와 회전을 절대값으로 매핑하는가, 시작 자세 대비 변화량으로 매핑하는가?
- pause 이후 기준 자세는 어떻게 다시 설정하는가?
- 손 추적 유실 시 마지막 명령 유지·정지·재초기화 중 어떤 동작을 하는가?
- Inspire 촉각은 접촉 여부, 힘, 분포 중 무엇을 제공하는가?
- 두 팔과 카메라·촉각 기록의 시간은 어떻게 정렬하는가?
- 현재 목표는 원격조작 성능 개선인가, 자율 정책 학습인가?

## 14. References / Related Links

- OPEN TEACH: https://arxiv.org/abs/2403.07870
- AnyTeleop: https://arxiv.org/abs/2307.04577
- FISH: https://arxiv.org/abs/2303.01497
- TAVI: https://arxiv.org/abs/2309.12300
- MimicGen: https://arxiv.org/abs/2310.17596
- DexMimicGen: https://arxiv.org/abs/2410.24185

※ 연구실 시스템 관련 항목은 사용자 제공 장비·초안을 기준으로 정리. 구현 및 성능이 검증되었다는 의미는 아님.
