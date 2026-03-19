# openarm_moveit

OpenArm 양팔 로봇을 MoveIt RViz UI로 제어하는 독립 패키지입니다.
RViz의 MotionPlanning 패널에서 interactive marker를 마우스로 드래그해 목표 자세를 설정하고, **Plan & Execute**로 실행합니다.

---

## 환경

| 항목 | 버전 |
|---|---|
| OS | Ubuntu 24.04 |
| ROS | ROS 2 Jazzy |

---

## 디렉토리 구조

```
openarm_moveit/
└── src/
    ├── setup_can.sh                    # CAN 인터페이스 활성화/비활성화 스크립트
    ├── openarm_can/                    # CAN 통신 라이브러리 (Damiao 모터 제어)
    ├── openarm_description/            # 로봇 URDF/xacro 모델, 메쉬 파일
    └── openarm_ros2/
        ├── openarm/                    # 메타 패키지
        ├── openarm_bimanual_moveit_config/  # MoveIt 설정 (핵심)
        │   ├── config/
        │   │   ├── openarm_bimanual.srdf     # Planning group, end-effector 정의
        │   │   ├── openarm_bimanual.urdf.xacro  # MoveIt용 로봇 description
        │   │   ├── kinematics.yaml           # IK solver 설정 (trac_ik)
        │   │   ├── joint_limits.yaml         # 관절 속도/가속도 한계
        │   │   ├── moveit_controllers.yaml   # MoveIt ↔ ros2_control 연결
        │   │   ├── moveit.rviz               # RViz 기본 레이아웃
        │   │   └── sensors_3d.yaml           # 센서 설정
        │   └── launch/
        │       └── demo.launch.py            # 메인 실행 파일
        ├── openarm_bringup/            # ros2_control 컨트롤러 설정
        │   └── config/v10_controllers/
        │       └── openarm_v10_bimanual_controllers.yaml
        └── openarm_hardware/           # 실제 로봇 하드웨어 인터페이스 (CAN 기반)
```

---

## 실행 방법

### 1단계: 의존성 설치

```bash
sudo apt update && sudo apt install -y \
  ros-jazzy-moveit \
  ros-jazzy-moveit-configs-utils \
  ros-jazzy-moveit-kinematics \
  ros-jazzy-moveit-planners \
  ros-jazzy-moveit-planners-ompl \
  ros-jazzy-moveit-planners-chomp \
  ros-jazzy-moveit-planners-stomp \
  ros-jazzy-moveit-ros-move-group \
  ros-jazzy-moveit-ros-visualization \
  ros-jazzy-moveit-ros-planning-interface \
  ros-jazzy-moveit-simple-controller-manager \
  ros-jazzy-trac-ik-kinematics-plugin \
  ros-jazzy-ros2-control \
  ros-jazzy-ros2-controllers \
  ros-jazzy-controller-manager \
  ros-jazzy-joint-state-broadcaster \
  ros-jazzy-joint-trajectory-controller \
  ros-jazzy-forward-command-controller \
  ros-jazzy-gripper-controllers \
  ros-jazzy-robot-state-publisher \
  ros-jazzy-joint-state-publisher \
  ros-jazzy-rviz2 \
  ros-jazzy-xacro \
  ros-jazzy-tf2-ros
```

### 2단계: 빌드

```bash
cd ~/openarm_moveit
colcon build
source install/setup.bash
```

### 3단계: CAN 인터페이스 활성화 (실제 로봇 연결 시 필수)

> 시뮬레이션(`use_fake_hardware:=true`)으로만 사용하는 경우 이 단계는 생략 가능합니다.

```bash
cd ~/openarm_moveit/src
./setup_can.sh up
```

- `can0` → 오른팔
- `can1` → 왼팔
- CAN FD 모드, bitrate: 1Mbps / data bitrate: 5Mbps

종료 시:
```bash
./setup_can.sh down
```

### 4단계: 실행

**시뮬레이션 (fake hardware):**
```bash
ros2 launch openarm_bimanual_moveit_config demo.launch.py use_fake_hardware:=true
```

**실제 로봇:**
```bash
ros2 launch openarm_bimanual_moveit_config demo.launch.py use_fake_hardware:=false
```

---

## RViz 사용법

1. RViz가 열리면 왼쪽 **MotionPlanning** 패널 확인
2. **Planning Group** 드롭다운에서 제어할 그룹 선택:
   - `both_arms` — 양팔 동시 제어 (기본값)
   - `left_arm` — 왼팔만
   - `right_arm` — 오른팔만
3. 3D 뷰에서 팔 끝의 **interactive marker**(구 모양)를 드래그해 목표 자세 설정
4. **Plan & Execute** 클릭

---

## 주요 파라미터

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| `use_fake_hardware` | `false` | `true`: 시뮬레이션, `false`: 실제 로봇 |
| `right_can_interface` | `can0` | 오른팔 CAN 인터페이스 |
| `left_can_interface` | `can1` | 왼팔 CAN 인터페이스 |
| `arm_type` | `v10` | 팔 버전 |

---

## Credits

다음 Enactic, Inc. 오픈소스 저장소를 기반으로 합니다. 라이선스: [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0)

- [openarm_ros2](https://github.com/enactic/openarm_ros2)
- [openarm_description](https://github.com/enactic/openarm_description)
- [openarm_can](https://github.com/enactic/openarm_can)

### 원본 대비 수정 사항

- `openarm_bimanual.urdf.xacro`: MoveItConfigsBuilder가 양팔 모델을 올바르게 로드하도록 기본 인자 수정 (`bimanual=true`)
- `openarm_bimanual.srdf`: URDF와 이름 통일, `both_arms` planning group 추가, end-effector `parent_group` 추가
- `joint_limits.yaml`: trajectory planning에 필요한 가속도 한계 추가
- `moveit.rviz`: 기본 Planning Group을 `both_arms`로 변경

## License

Apache License 2.0 — 각 패키지의 `LICENSE` 파일 참고
