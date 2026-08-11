# K.O — Intelligent Robotic Boxing Training System

> **AI Vision · ROS 2 · Doosan M0609 · Force Control · Voice Interface · Web UI**  
> **사용자 움직임 인식 → 로봇 미트 반응 → 훈련 데이터 저장 → 코칭 피드백 제공**

---

## 1. 프로젝트 개요

- 프로젝트명
  - **K.O**
  - 지능형 복싱 트레이닝 시스템

- 프로젝트 목적
  - 고정형 샌드백의 한계 보완
  - 실제 미트 훈련의 파트너 의존성 완화
  - 사용자 자세·타격 결과의 정량 분석
  - 훈련 기록 기반 코칭 리포트 제공

- 핵심 컨셉
  - **사용자 인식**
  - **실시간 판단**
  - **물리적 반응**
  - **데이터 피드백**

- 기대 효과
  - 혼자서도 실전감 있는 복싱 훈련 가능
  - 사용자 체형 기반 맞춤형 미트 훈련 제공
  - 자세·정확도·반응 결과를 데이터로 누적 관리

---

## 2. 전체 시스템 아키텍처

<p align="center">
  <img src="./docs/images/ko_system_architecture.png" alt="KO 전체 시스템 아키텍처" width="100%">
</p>

- 구성 요소
  - **User UI**
    - 사용자 등록
    - 리치 측정
    - 훈련 선택
    - 결과 리포트
  - **Voice Module**
    - Wake Word
    - Whisper STT
    - TTS 안내
  - **Vision Module**
    - 3-Camera Input
    - YOLO11n + BoT-SORT
    - MediaPipe Pose
    - 3D 좌표·속도 산출
    - EKF
  - **ROS 2 Core**
    - Session Bridge
    - Robot Bridge
    - Topic / Service
    - State Sync
  - **Robot / Force**
    - Doosan M0609
    - Weaving
    - Mitt Positioner
    - Hit Detection
    - Compliance
    - Rebound / Return
  - **Training DB**
    - SQLite
    - 사용자 정보
    - 훈련 기록
    - Hit Result
  - **AI Coaching Report**
    - 자세 점수
    - 성과 요약
    - 이전 기록 비교
    - 코칭 리포트

- 핵심 데이터 흐름
  - 사용자 → UI
    - 프로필 입력
    - 훈련 선택
  - 사용자 → ROS 2 Core
    - 음성 명령
    - 자세·펀치 정보
  - Voice / Vision → ROS 2 Core
    - 훈련 명령
    - 3D 좌표·속도
  - ROS 2 Core → Robot / Force
    - 미트 목표 자세
    - 제어 명령
  - Robot / Force → ROS 2 Core
    - 로봇 상태
    - Hit Result
  - ROS 2 Core → DB / Report
    - 세션 저장
    - 기록 분석
    - 코칭 결과 생성

- 통합 포인트
  - **ROS 2 Core**가 전체 연결의 중심
  - Session Bridge가 훈련 세션과 HitResult 저장 흐름을 통합 관리
  - UI·Voice·Vision·Robot·Data가 단일 파이프라인으로 연결됨

---

## 3. 전체 실행 시퀀스

<p align="center">
  <img src="./docs/images/ko_execution_sequence.png" alt="KO 전체 실행 시퀀스" width="100%">
</p>

- 사용자 단계
  - 시스템 실행
  - 사용자 등록 / 불러오기
  - 리치 측정 / 신체 정보 입력
  - 훈련 시작
  - 음성 호출
    - 예: **Wake Up KO**
  - 훈련 명령 발화
    - 예: 원투 1분 / 콤비네이션 1
  - 펀치 수행
  - 결과 확인

- UI / 시스템 단계
  - 장치 연결 확인
    - 카메라
    - 마이크
    - 로봇
  - 프로필 저장 / 불러오기
  - 카메라 정렬 및 준비 상태 확인
  - 대기 화면 / 훈련 UI 전환
  - 세션 시작 / 명령 전달
  - 훈련 종료 요청 처리

- Voice / Vision 단계
  - Wake Word 감지
  - Whisper STT 명령 인식
  - 사용자 자세·펀치 추적
  - 3D 손목 좌표 / 속도 추정
  - 펀치 예상 타점 생성

- ROS 2 / Robot 단계
  - 로봇 준비 상태 확인
  - 위빙 대기 동작
  - 위빙 정지 → 펀치 대기 자세 복귀
  - 미트 목표 자세 생성
  - M0609 미트 이동
  - 타격 대응 / 콤비네이션 반복

- Data / Report 단계
  - 훈련 기록 저장
  - Hit Result / 세션 데이터 정리
  - AI 코칭 보고서 생성
  - 훈련 결과 표시

- 핵심 흐름 요약
  - **사용자 준비**
  - **음성 명령**
  - **자세·펀치 추적**
  - **예상 타점 생성**
  - **로봇 미트 이동**
  - **기록 저장**
  - **코칭 보고서 생성**

---

## 4. 전체 플로우차트

<p align='center'>\n  <img src='./docs/images/ko_full_flowchart.jpg' alt='KO 통합 플로우차트' width='100%'>\n</p>

- 플로우차트 범위
  - UI
  - Calibration
  - 실시간 Vision
  - 분석·필터링
  - Robot / Force 연동

- 활용 목적
  - 전체 모듈 관계 파악
  - 입력 / 출력 흐름 확인
  - 개발 단계별 의존성 확인
  - 디버깅 및 발표 자료 보조

---

## 5. 주요 기능

- UI
  - 사용자 등록
  - 리치 측정
  - 훈련 종류 선택
  - USER / ADMIN 모드 분리
  - 결과 리포트 표시

- Voice
  - Custom Wake Word
  - Whisper STT
  - 핸즈프리 훈련 시작
  - 음성 기반 화면 전환 및 훈련 명령

- Vision
  - 3-Camera 입력
  - 사용자 ID 고정
  - 자세 / 손목 추적
  - 3D 손목 좌표 생성
  - 속도 벡터 추정
  - EKF 안정화

- Calibration
  - 카메라 Intrinsic 보정
  - Camera → Robot BASE 직접 정렬
  - 변환 체인 제거
  - 좌표 정합 오차 최소화

- Robot
  - M0609 위빙 대기 동작
  - 사용자 체형 기반 미트 위치 보정
  - 잽 / 스트레이트 / 훅 / 어퍼 대응
  - Position + Orientation 동시 제어

- Force
  - 실시간 외력 감지
  - 실제 타격 판정
  - Compliance
  - Rebound / Return
  - 타격 결과 저장

- Report
  - HitResult 저장
  - 훈련 이력 누적
  - BEST / CHECK 산출
  - 이전 기록 비교
  - AI 코칭 리포트 생성

---

## 6. Vision 파이프라인

- 목표
  - Stable ID 확보
  - Low Latency 유지
  - Robot-ready 3D 좌표 생성

- 처리 흐름
  - YOLO11n + BoT-SORT
    - 사용자 ID 고정
  - MediaPipe Pose
    - ROI 내부 관절 추론
  - 3-Camera 동시 처리
  - 2대 이상 카메라 기반 삼각측량
  - RealSense Depth 비교 / 보조
  - EKF 기반 위치·속도 추정
  - Robot BASE 좌표 발행

- Hybrid 구조 선정 이유
  - MediaPipe 단독
    - 장점: 빠름
    - 한계: Person ID 불안정
  - YOLO Pose 단독
    - 장점: ID 안정성
    - 한계: 3Cam 동시 처리 시 지연 증가
  - 최종 구조
    - YOLO + BoT-SORT → 대상 고정
    - MediaPipe → 빠른 관절 추론
    - Latest Frame 처리 → 지연 완화

- EKF 역할
  - 손목 위치 / 속도 상태 추정
  - 순간 튐 완화
  - 이상값 제거
  - Stale 데이터 차단

---

## 7. Calibration 방식

- 기본 원칙
  - 카메라끼리 Pairwise 연쇄 연결 사용 안 함
  - 각 카메라를 Robot BASE에 직접 정렬

- 처리 절차
  - 카메라별 Intrinsic Calibration
  - ChArUco 보드 장착
  - 로봇 여러 자세에서 샘플 수집
  - `T_base_flange`, `T_cam_board` 계산
  - 카메라별 `T_base_camera` 산출
  - 최종 Robot BASE 통합

- 장점
  - 변환 체인 제거
  - 누적 오차 차단
  - 좌표계 단순화
  - 실시간 로봇 제어에 직접 활용 가능

---

## 8. Robot / Force 제어

- Weaving
  - 대기 상태에서 위빙 수행
  - 훈련 명령 수신 시 위빙 정지
  - 펀치 대기 자세로 전환

- 사용자 맞춤 미트 Pose
  - 입력 정보
    - 키
    - 좌우 리치
    - 주손
    - 훈련 종류
    - 펀치 종류
  - 출력
    - `Target Pose = [X, Y, Z, A, B, C]`

- 펀치별 대응
  - JAB / STRAIGHT
    - 정면 타격면 유지
  - HOOK
    - 측면 위치 보정
    - Yaw 회전
  - UPPERCUT
    - 상향 진입 방향에 맞는 각도 조정

- Force 처리 흐름
  - WAITING
  - Force / Moment Monitoring
  - Hit Detection
  - Impact Analysis
  - Rebound
  - Return
  - 다음 펀치 대기

- 저장 정보
  - Peak Force
  - Impulse
  - Contact Time
  - Hit Position
  - Center Error
  - HitResult

---

## 9. 콤비네이션 제어

- 기본 방식
  - 단순 타이머 기반 진행 아님
  - **실제 타격 Event 기반 진행**

- 흐름
  - 타점 A 생성
  - 실제 타격 감지
  - Rebound / Return 수행
  - 다음 사용자 맞춤 미트 Pose 생성
  - HitTest 재시작
  - WAITING_FOR_HIT
  - 타점 B 진행

- 장점
  - 사용자 속도 반영 가능
  - 훈련 현실감 증가
  - 불필요한 시간 대기 감소

---

## 10. 하드웨어 구성

- 로봇
  - **Doosan M0609**

- 카메라
  - Front
    - Intel RealSense D435 / D435i
  - Side
    - Logitech C270 × 2

- 말단 장치
  - 평면 복싱 미트
  - 전용 Tool Adapter

- 오디오
  - Microphone

- 제어 PC
  - Ubuntu 22.04
  - ROS 2 Humble
  - Python 3.10

- 기본 카메라 Runtime
  - `640 × 480 @ 30 FPS`

---

## 11. 소프트웨어 스택

- Platform
  - Ubuntu 22.04
  - ROS 2 Humble
  - Python 3.10

- AI / Vision
  - YOLO11n
  - BoT-SORT
  - MediaPipe Pose
  - OpenCV
  - Intel RealSense SDK
  - NumPy / SciPy
  - EKF

- Voice
  - openWakeWord
  - Whisper STT
  - Piper / TTS

- UI / Data
  - Flask
  - HTML / CSS / JavaScript
  - SQLite

- Robot / Force
  - Doosan ROS 2
  - Motion Control
  - RT Force
  - Compliance / Rebound / Return

---

## 12. Repository 구조

```text
KO/
├── calibration/             # 최종 Intrinsic / Robot World 결과
├── calibration_tools/       # 카메라 및 Robot BASE 캘리브레이션 도구
├── config/                  # Camera / Runtime / Mitt 설정
├── data/                    # 훈련 및 Hit 기록
├── force_control/
│   └── boxing_robot_ws/     # ROS 2 Force / Hit / SessionBridge Workspace
├── interfaces/              # 인터페이스 정의
├── models/                  # Vision 모델
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

## 13. 설치 방법

- 권장 환경
  - Ubuntu 22.04
  - ROS 2 Humble
  - Python 3.10

- 초기 준비
```bash
chmod +x setup.sh run_final.sh test_final.sh stop_final.sh
./setup.sh --build-force
```

- 설명
  - `setup.sh`
    - Vision / UI 환경 구성
  - `--build-force`
    - Force ROS 2 workspace 포함 빌드

---

## 14. 검증 / 테스트

- 소프트웨어 테스트
```bash
./test_final.sh
```

- 자동 검증 범위
```text
Final Integration Contract      PASS
Python Syntax                   105 files / 0 errors
YAML Parse                       15 files / 0 errors
UI / API                         42 / 42 PASS
Force / Rebound / Mitt Logic     94 / 94 PASS
Vision Core                      16 / 16 PASS
```

- 하드웨어 프리플라이트
```bash
./test_final.sh --hardware
```

- 주요 검사 항목
  - C270 LEFT / RIGHT 매핑
  - RealSense RGB + Depth
  - 640×480 유효 프레임 수신
  - 마이크 입력
  - Wake Word 모델 초기화
  - ROS 2 / Doosan 필수 서비스
  - Force workspace overlay

- 동작 원칙
  - 하드웨어 프리플라이트 실패 시
    - 실제 프로젝트 실행 차단

---

## 15. 실행 방법

- USER Mode
```bash
./run_final.sh
```

- ADMIN Mode
```bash
./run_final.sh --admin-mode
```

- ADMIN 화면 제공 정보
  - LEFT / FRONT / RIGHT Camera Preview
  - Pose / Guard / Impact 상태
  - Robot BASE Position / Velocity
  - 시스템 상태 / 개발 진단 정보

- 종료
```bash
./stop_final.sh
```

---

## 16. 데이터 / 출력

- Impact Evidence
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

- Training / Hit Records
```text
data/hit_records/
ui/instance/ko.sqlite3
```

---

## 17. 안전 사항

- 유의점
  - 실제 사용자가 협동로봇 미트를 타격하는 구조
  - 소프트웨어 PASS와 물리 안전 검증은 별개

- 첫 실물 구동 권장 순서
  - 작업영역에서 사람 제거
  - HOME Pose 저속 확인
  - MITT / WEAVE Ready Pose 확인
  - Weaving 궤적 및 간섭 확인
  - Wake Word Soft Stop 확인
  - 단일 펀치 위치 이동 확인
  - WAITING_FOR_HIT 상태 확인
  - 약한 타격으로 Force / Rebound 확인
  - 단일 펀치 검증 후 콤비네이션 진행

---

## 18. 한계점 / 향후 과제

- 한계점
  - 카메라 설치 위치 변화 시 Calibration 재검증 필요
  - 고속 펀치 및 가림 환경에서 추가 사용자 검증 필요
  - 실제 반복 타격 안전성 데이터 추가 확보 필요

- 향후 과제
  - 훅 / 어퍼컷 사용자별 Target Pose 정교화
  - 자세 점수 / Force Accuracy 기반 Scoring 고도화
  - 장기 훈련 이력 분석 기능 강화
  - 관리자 로그 및 통계 기능 확장

---

## 19. 팀 구성

- **Team E-3**
  - 정용준
  - 정진목
  - 김승주
  - 김윤식

---

## 20. 프로젝트 한 줄 요약

- **K.O는 “사용자 인식 → 판단 → 물리 반응 → 데이터 피드백”을 하나의 ROS 2 기반 폐루프 시스템으로 연결한 지능형 복싱 트레이닝 프로젝트입니다.**
