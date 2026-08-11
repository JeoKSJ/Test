# K.O — Intelligent Robotic Boxing Training System

> **AI Vision · ROS 2 · Doosan M0609 · Force Control · Voice Interface · Web UI**  
> 사용자의 움직임을 인식하고, 협동로봇 미트가 능동적으로 반응하며, 훈련 결과를 데이터로 피드백하는 **지능형 복싱 트레이닝 시스템**입니다.

---

## 1. Project Overview

기존 샌드백은 정해진 위치를 반복 타격하는 수동적인 훈련에 가깝고, 실제 미트 훈련은 코치나 훈련 파트너가 필요합니다. 또한 혼자 훈련할 경우 자신의 자세와 타격 정확도를 객관적으로 확인하기 어렵고, 훈련 결과를 체계적으로 누적하기 어렵다는 한계가 있습니다.

**K.O**는 이러한 문제를 해결하기 위해 **Vision · Voice · Robot · Force · Data**를 하나의 ROS 2 기반 시스템으로 통합했습니다. 사용자의 신체 정보와 펀치 동작을 인식하고, 협동로봇이 사용자에게 맞는 위치와 방향으로 미트를 제시하며, 실제 타격이 발생하면 외력 데이터를 기반으로 반응합니다. 이후 훈련 결과를 저장하고 분석하여 개인별 코칭 리포트까지 제공하는 것을 목표로 합니다.

### 핵심 목표

- 3-Camera 기반 사용자 및 주먹 추적
- Robot BASE 기준 3D 주먹 위치·속도 산출
- 사용자 키·리치·주손 기반 미트 위치 보정
- 잽·스트레이트·훅·어퍼별 미트 위치 및 방향 제어
- Force 기반 실제 타격 감지 및 Rebound / Return
- Wake Word + Whisper STT 기반 비접촉 훈련 제어
- 훈련 기록, 타격 결과 및 자세 데이터를 활용한 코칭 리포트
- ROS 2 기반 UI · Voice · Vision · Robot · Force · Data 통합

---

## 2. System Architecture

<p align="center">
  <img src="./docs/images/ko_system_architecture.png" alt="K.O 전체 시스템 아키텍처" width="100%">
</p>

K.O의 전체 시스템은 **ROS 2 Core를 중심으로 User UI, Voice Module, Vision Module, Robot / Force, Training DB, AI Coaching Report가 연결되는 구조**입니다.

사용자는 UI를 통해 프로필을 등록하고 리치를 측정한 뒤 훈련을 선택합니다. 음성 입력은 Wake Word와 Whisper STT를 통해 훈련 명령으로 변환되며, Vision Module은 3대의 카메라를 이용하여 사용자의 자세와 펀치를 추적하고 3D 위치 및 속도를 산출합니다.

이 정보들은 ROS 2 Core로 전달됩니다. `Session Bridge`는 훈련 세션과 HitResult의 흐름을 관리하고, `Robot Bridge`는 M0609과 UI 및 Vision 사이의 제어 인터페이스를 담당합니다. 로봇은 전달받은 사용자 맞춤형 Target Pose에 따라 미트를 이동시키고, 실제 타격 발생 시 Force 기반 Hit Detection과 Compliance, Rebound, Return 동작을 수행합니다.

훈련 종료 후에는 사용자 정보와 HitResult, 훈련 기록을 SQLite 기반 Training DB에 저장하고, 누적 데이터를 분석해 자세 점수, 성과 요약, 이전 기록 비교 및 코칭 리포트를 생성합니다.

### 핵심 데이터 흐름

```text
사용자
  ↓
UI / Voice / Vision
  ↓
ROS 2 Core
  ↓
Robot / Force
  ↓
HitResult
  ↓
Training DB
  ↓
AI Coaching Report
```

> **핵심 구조:** 사용자 인식 → 판단 → 물리 반응 → 데이터 피드백

---

## 3. Full Execution Sequence

<p align="center">
  <img src="./docs/images/ko_execution_sequence.png" alt="K.O 전체 실행 시퀀스" width="100%">
</p>

시스템 실행 후에는 먼저 카메라, 마이크, 로봇 등 주요 장치의 연결 상태를 확인합니다. 사용자는 기존 프로필을 불러오거나 신규 등록을 진행하며, 신규 사용자는 키와 리치 등 신체 정보를 입력합니다.

훈련 준비 단계에서는 카메라 정렬과 사용자 자세를 확인하고, 로봇은 위빙 대기 동작을 수행합니다. 이후 사용자가 **“Wake Up KO”**를 호출하면 Wake Word가 활성화되고, Whisper STT가 뒤따르는 훈련 명령을 인식합니다.

훈련 명령이 확정되면 Vision Module은 사용자의 자세와 펀치를 추적하고, 3대 카메라를 이용해 손목의 3D 좌표와 속도를 추정합니다. 이를 바탕으로 예상 타점을 생성한 뒤 ROS 2 Core를 통해 M0609에 전달합니다.

로봇은 위빙을 정지하고 펀치 대기 자세로 복귀한 후 사용자 체형과 펀치 종류에 맞는 미트 목표 자세를 생성합니다. 사용자가 실제로 타격하면 Force 기반 Hit Detection이 수행되고, 필요에 따라 Rebound와 Return 동작을 거쳐 다음 타점 또는 다음 콤비네이션 단계로 전환됩니다.

훈련이 종료되면 HitResult와 세션 데이터를 저장하고, 훈련 결과를 분석하여 AI Coaching Report를 생성한 뒤 UI에 최종 결과를 표시합니다.

### 사용자 기준 흐름

```text
시스템 실행
  ↓
사용자 등록 / 불러오기
  ↓
리치 측정 / 신체 정보 입력
  ↓
훈련 시작
  ↓
Wake Up KO
  ↓
훈련 명령
  ↓
자세·펀치 추적
  ↓
예상 타점 생성
  ↓
M0609 미트 이동
  ↓
실제 타격
  ↓
Rebound / Return
  ↓
훈련 기록 저장
  ↓
코칭 보고서
```

---

## 4. Full System Flowchart

<p align="center">
  <img src="./docs/images/ko_full_flowchart.jpg" alt="K.O 통합 플로우차트" width="100%">
</p>

위 플로우차트는 프로젝트 내부의 세부 실행 흐름을 **UI / Calibration / 실시간 Vision / 분석·필터링 / Robot·Force 연동** 단위로 확장하여 정리한 자료입니다.

System Architecture가 모듈 간 관계를 보여주는 상위 구조라면, Full System Flowchart는 각 기능 내부에서 어떤 조건과 데이터가 다음 단계로 전달되는지를 확인하기 위한 상세 설계 자료입니다.

---

## 5. Main Features

| 영역 | 주요 기능 |
|---|---|
| **UI** | 사용자 등록, 리치 측정, 훈련 선택, USER / ADMIN 모드, 결과 리포트 |
| **Voice** | Custom Wake Word, Whisper STT, TTS 안내, 비접촉 훈련 제어 |
| **Vision** | 3-Camera 입력, Person ID 고정, Pose / Fist 추적, Triangulation, EKF |
| **Calibration** | 카메라 Intrinsic 보정, Camera → Robot BASE 직접 정렬 |
| **Robot** | M0609 Weaving, 사용자 맞춤 미트 위치, 펀치별 Position / Orientation |
| **Force** | RT 외력 감지, Hit Detection, Compliance, Rebound, Return |
| **Combination** | 실제 Hit Event 기반 잽·스트레이트·훅·어퍼 시퀀스 |
| **Report** | HitResult 저장, BEST / CHECK, 이전 기록 비교, AI Coaching |
| **Safety** | Hardware Preflight, Critical Node 감시, Stale 데이터 차단 |

---

## 6. Vision Pipeline

Vision의 최종 목표는 단순히 가장 정확한 Pose 모델을 사용하는 것이 아니라, **Stable ID · Low Latency · Robot-ready 3D Coordinate**를 동시에 확보하는 것입니다.

초기에는 MediaPipe Pose 단독 방식을 사용했지만, 지속적인 Person ID가 없어 가림이나 다중 인물 상황에서 타겟이 전환되는 문제가 있었습니다. 이후 YOLO11n + BoT-SORT를 적용하여 사용자 ID를 안정적으로 유지할 수 있었지만, 3대의 카메라에서 모든 Pose 추론을 수행할 경우 처리 지연이 증가했습니다.

최종 구조에서는 **YOLO11n + BoT-SORT가 복서의 ID를 고정하고, MediaPipe Pose가 선택된 ROI 내부의 관절을 빠르게 추론하는 Hybrid 구조**를 적용했습니다.

```text
YOLO11n + BoT-SORT
        ↓
복서 Person ID 고정
        ↓
MediaPipe Pose
        ↓
손목 / 관절 추론
        ↓
3-Camera Synchronization
        ↓
Triangulation
        ↓
RealSense Depth 비교
        ↓
EKF Position + Velocity
        ↓
Robot BASE 3D Coordinate
```

### EKF Tracking

손목의 3D 측정값을 그대로 로봇 제어에 사용하면 카메라 노이즈나 순간 가림으로 인해 좌표가 튈 수 있습니다. 이를 완화하기 위해 EKF를 적용하여 위치와 속도를 동시에 상태값으로 추정합니다.

예측값과 실제 측정값의 차이가 지나치게 큰 경우에는 이상값으로 제거하고, 일정 시간 동안 새로운 좌표가 들어오지 않으면 Stale 상태로 전환하여 오래된 좌표가 로봇 제어에 사용되지 않도록 차단합니다.

---

## 7. Camera ↔ Robot Calibration

세 카메라의 좌표를 하나로 통합하기 위해 초기에는 카메라 간 Pairwise 변환 구조를 검토했습니다. 그러나 여러 변환 행렬을 순차적으로 연결할수록 위치 및 회전 오차가 누적될 수 있다는 문제가 있었습니다.

최종 방식에서는 각 카메라를 다른 카메라와 연결하지 않고 **각각 Robot BASE에 직접 1:1 정렬**했습니다.

```text
Front Camera ─────┐
Left Camera  ─────┼──→ Robot BASE
Right Camera ─────┘
```

먼저 각 카메라의 Intrinsic Calibration을 수행한 후, ChArUco 보드를 로봇 플랜지에 장착하여 다양한 자세에서 `T_base_flange`와 `T_cam_board`를 동시에 수집합니다. 이후 복수 샘플의 위치·회전 오차를 최소화하여 각 카메라별 `T_base_camera`를 독립적으로 계산합니다.

이 구조를 통해 변환 체인을 제거하고, 모든 Vision 결과를 동일한 M0609 Robot BASE 좌표계에서 사용할 수 있도록 구성했습니다.

---

## 8. Robot Mitt Control

K.O의 로봇 미트는 고정된 하나의 좌표만 사용하는 방식이 아니라, **사용자의 키·리치·주손과 펀치 종류에 따라 Target Pose를 재계산**합니다.

```text
Target Pose = [X, Y, Z, A, B, C]
```

잽과 스트레이트는 정면 타격 기준으로 미트 면을 사용자 방향으로 유지합니다. 훅은 측면에서 진입하는 타격이기 때문에 미트 위치를 옆으로 이동시키고 Yaw 방향을 함께 회전합니다. 어퍼컷은 아래에서 위로 진입하므로 타격면이 펀치 진행 방향과 수직이 되도록 각도를 조정합니다.

### Weaving Idle Motion

훈련 대기 상태에서는 로봇이 정지해 있는 대신 복싱의 회피 동작인 **Weaving**을 Idle Motion으로 수행합니다.

훈련 명령이 들어오면 위빙을 즉시 정지하고, 펀치 대기 자세로 전환한 뒤 사용자에게 맞는 미트 위치로 이동합니다.

---

## 9. Force / Hit / Rebound

사용자가 실제로 로봇 미트를 타격하기 때문에 단순 위치 제어만으로는 자연스러운 미트 반응을 만들기 어렵습니다.

K.O는 로봇 외력을 실시간으로 모니터링하여 실제 타격을 감지하고, 타격이 확인되면 미트가 충격 방향으로 밀렸다가 다시 기준 위치로 돌아오는 Rebound / Return 구조를 사용합니다.

```text
WAITING
  ↓
Force / Moment Monitoring
  ↓
HIT DETECTION
  ↓
Impact Analysis
  ↓
REBOUND
  ↓
RETURN
  ↓
WAITING_FOR_HIT
```

타격 순간에는 Peak Force, Impulse, Contact Time, Hit Position, Center Error 등의 정보를 계산하고 HitResult로 저장합니다.

---

## 10. Event-based Combination Control

콤비네이션은 일정 시간이 지나면 다음 타점으로 이동하는 단순 타이머 방식이 아니라, **사용자의 실제 타격이 확인된 경우에만 다음 단계로 진행하는 Event 기반 구조**입니다.

예를 들어 `JAB → STRAIGHT → HOOK`의 경우 다음과 같은 흐름으로 동작합니다.

```text
JAB Target
  ↓
Actual Hit Event
  ↓
Rebound / Return
  ↓
STRAIGHT Target
  ↓
Actual Hit Event
  ↓
Rebound / Return
  ↓
HOOK Target + Orientation
```

이를 통해 사용자의 실제 펀치 속도에 맞춰 콤비네이션이 진행되며, 로봇이 사용자의 행동을 무시한 채 미리 다음 동작으로 이동하는 것을 방지합니다.

---

## 11. Hardware

| 장치 | 구성 |
|---|---|
| Collaborative Robot | **Doosan M0609** |
| Front Camera | **Intel RealSense D435 / D435i** |
| Side Cameras | **Logitech C270 × 2** |
| End Effector | 평면 복싱 미트 + 전용 Tool Adapter |
| Audio | Microphone |
| Control PC | Ubuntu 22.04 / ROS 2 Humble |

기본 Camera Runtime은 `640 × 480 @ 30 FPS`를 기준으로 구성했습니다.

---

## 12. Software Stack

### Platform
- Ubuntu 22.04
- ROS 2 Humble
- Python 3.10

### AI / Vision
- YOLO11n
- BoT-SORT
- MediaPipe Pose
- OpenCV
- Intel RealSense SDK
- NumPy / SciPy
- EKF

### Voice
- openWakeWord
- Whisper STT
- Piper / TTS

### UI / Data
- Flask
- HTML / CSS / JavaScript
- SQLite

### Robot / Force
- Doosan ROS 2
- M0609 Motion Control
- RT Force
- Compliance / Rebound / Return

---

## 13. Repository Structure

```text
KO/
├── calibration/             # 최종 Intrinsic / Robot World 결과
├── calibration_tools/       # 카메라 및 Robot BASE 캘리브레이션 도구
├── config/                  # Camera / Runtime / Mitt 설정
├── data/                    # 훈련 및 Hit 기록
├── force_control/
│   └── boxing_robot_ws/     # ROS 2 Force / Hit / SessionBridge Workspace
├── interfaces/              # 인터페이스 정의
├── models/                  # Vision 관련 모델
├── msg/                     # 메시지 스키마
├── output/
│   └── impacts/             # 타격 이미지 및 Metadata
├── robot_control/           # M0609 Weaving / UI-Robot Bridge
├── sandbag_vision/          # 실시간 3-Camera Vision Runtime
├── tests/                   # 통합 테스트
├── tools/                   # Preflight / Configuration 검사
├── ui/                      # Flask UI / Voice / Reporting / DB
├── setup.sh
├── run_final.sh
├── test_final.sh
├── stop_final.sh
├── FINAL_INTEGRATION.md
└── FINAL_TEST_REPORT.md
```

---

## 14. Setup

권장 환경은 **Ubuntu 22.04 + ROS 2 Humble + Python 3.10**입니다.

```bash
chmod +x setup.sh run_final.sh test_final.sh stop_final.sh
./setup.sh --build-force
```

`setup.sh`는 Vision 및 UI 실행 환경을 준비하고, `--build-force` 옵션을 사용하면 Force ROS 2 Workspace도 함께 빌드합니다.

---

## 15. Verification

### Software Test

```bash
./test_final.sh
```

현재 최종 통합본의 자동 검증 범위는 다음과 같습니다.

```text
Final Integration Contract      PASS
Python Syntax                   105 files / 0 errors
YAML Parse                       15 files / 0 errors
UI / API                         42 / 42 PASS
Force / Rebound / Mitt Logic     94 / 94 PASS
Vision Core                      16 / 16 PASS
```

자동 테스트의 PASS는 **소프트웨어 연결 계약과 로직이 준비되었다는 의미**이며, 실제 M0609의 물리적인 안전성이나 반복 타격에 대한 안정성을 의미하지는 않습니다.

### Hardware Preflight

```bash
./test_final.sh --hardware
```

Hardware Preflight에서는 C270 LEFT / RIGHT 매핑, RealSense RGB + Depth, 마이크 입력, Wake Word 모델, ROS 2 및 Doosan 필수 서비스 등을 확인합니다.

Preflight에 실패하면 실제 프로젝트 실행을 차단하도록 구성했습니다.

---

## 16. Run

### USER Mode

```bash
./run_final.sh
```

일반 사용자가 등록, 측정, 훈련 및 결과 확인을 수행하는 화면으로 실행합니다.

### ADMIN Mode

```bash
./run_final.sh --admin-mode
```

ADMIN Mode에서는 LEFT / FRONT / RIGHT Camera Preview, Pose / Guard / Impact 상태, Robot BASE Position / Velocity 및 시스템 진단 정보를 확인할 수 있습니다.

### Stop

```bash
./stop_final.sh
```

비정상 종료나 중복 실행 상태가 발생한 경우 프로젝트 관련 프로세스를 정리한 뒤 재실행합니다.

---

## 17. Data / Output

### Impact Evidence

```text
output/impacts/
└── impact_XXXXX_YYYYMMDD_HHMMSS_xxxxxx/
    ├── left_raw.jpg
    ├── front_raw.jpg
    ├── right_raw.jpg
    ├── left.jpg
    ├── front.jpg
    ├── right.jpg
    ├── impact_triptych.jpg
    └── impact_metadata.json
```

### Training / Hit Records

```text
data/hit_records/
ui/instance/ko.sqlite3
```

---

## 18. Safety Notes

K.O는 사용자가 실제로 협동로봇의 미트를 타격하는 구조이기 때문에 **소프트웨어 테스트와 물리 안전 검증을 반드시 구분**해야 합니다.

첫 실물 구동 시에는 HOME Pose, MITT / WEAVE Ready Pose, Weaving 궤적, 주변 기구 간섭을 저속으로 확인한 뒤 단일 펀치부터 검증하는 것을 권장합니다. 이후 약한 타격으로 Force와 Rebound를 확인하고, 단일 동작의 안정성을 확인한 후 콤비네이션으로 확장해야 합니다.

---

## 19. Current Limitations & Future Work

현재 시스템은 전체 통합 루프를 구현했지만, 카메라 설치 위치 변화 시 Calibration 재검증이 필요하고 고속 펀치 및 가림 상황에 대한 추가적인 사용자 검증이 필요합니다.

향후에는 훅과 어퍼컷의 사용자별 Target Pose를 더욱 정교하게 튜닝하고, 자세 점수와 Force Accuracy를 결합한 Scoring을 고도화할 계획입니다. 또한 장기 훈련 이력과 관리자 분석 기능을 확장하여 단순 1회성 훈련이 아니라 누적 성과 분석이 가능한 시스템으로 발전시키는 것을 목표로 합니다.

---

## 20. Team

**Team E-3**

- 정용준
- 정진목
- 김승주
- 김윤식

---

## 21. Project Summary

> **K.O는 “사용자 움직임 인식 → 판단 → 물리 반응 → 데이터 피드백”을 하나의 ROS 2 기반 폐루프 시스템으로 연결한 사람 반응형 지능형 복싱 트레이닝 프로젝트입니다.**
