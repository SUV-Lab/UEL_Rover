# 개발 가이드

코드를 고치거나 기능을 추가하는 사람을 위한 문서. 모르는 용어는 [용어집](glossary.md) 을 본다.

## 저장소 구조

```
UEL_Rover/
├── README.md                        # 프로젝트 소개와 빠른 시작
├── docs/
│   ├── getting-started.md           # 초기 설정 (제어보드, 로버 컴퓨터, GCS PC)
│   ├── usage.md                     # 실행, 조종기 스위치, 동작 확인, GCS 조종
│   ├── ros-interface.md             # 노드, 토픽, 파라미터
│   ├── development.md               # 이 문서
│   ├── troubleshooting.md           # 문제 해결, 알려진 주의사항
│   ├── glossary.md                  # 용어집
│   ├── architecture/                # 시스템 개요
│   │   ├── uel-rover.png
│   │   ├── uel-rover-runtime.html
│   │   └── uel-rover-runtime.architecture.json
│   └── assets/
│       └── gcs_sample.png           # GCS(Foxglove) 화면 예시
├── scripts/
│   └── setup.sh                     # 로버(Jetson) 최초 1회 환경 설정
├── firmware/                        # 제어보드 펌웨어
│   └── OpenRB-150/
│       ├── README.md                # 배선, 빌드·업로드, ROS 인터페이스, 드라이브 모드, 확인할 상수
│       └── OpenRB-150.ino           
├── ros2_ws/                         # colcon 워크스페이스
│   └── src/uel_rover/               # ROS 2 패키지
│       ├── README.md                # 노드·파라미터·launch 인자 상세
│       ├── CMakeLists.txt
│       ├── package.xml
│       ├── src/
│       │   ├── commander.cpp        # /cmd_vel → /cmd_vel_out
│       │   ├── monitor.cpp          # 제어보드 링크·모드·바퀴 속도 감시 → /diagnostics
│       │   └── gcs_teleop.cpp       # GCS Teleop 방향 × +/− 로 고른 속도 → /cmd_vel
│       ├── launch/
│       │   ├── bringup.launch.py    # micro_ros_agent + commander + monitor
│       │   └── gcs.launch.py        # foxglove_bridge + gcs_teleop (GCS 연결, 선택)
│       ├── config/
│       │   ├── rover.yaml           # commander / monitor / gcs_teleop 파라미터
│       │   └── foxglove_bridge.yaml # foxglove_bridge 파라미터
│       └── examples/                # ROS2 코드 예제
│           ├── README.md            # 각 예제를 패키지에 넣는 절차
│           ├── odometry/            # odom_publisher.cpp, odometry.yaml
│           ├── description/         # uel_rover.urdf, description.launch.py
│           ├── sensors/             # RealSense D455 → /scan
│           └── navigation/          # Nav2, slam_toolbox, AMCL launch/yaml
└── gcs/
    ├── README.md                    # Foxglove 설치·접속·패널·조종·문제 해결
    └── foxglove/
        └── uel_rover_gcs.json       # Foxglove 레이아웃
```

디렉터리별 상세는 각 README 를 본다.

| 알고 싶은 것 | 문서 |
|---|---|
| 제어보드 배선, 펌웨어 빌드·업로드, 드라이브 모드 규칙, 배포 전 확인할 상수 | [firmware/OpenRB-150/README.md](../firmware/OpenRB-150/README.md) |
| commander / monitor 노드 동작, 파라미터, launch 인자, 빌드·실행 | [ros2_ws/src/uel_rover/README.md](../ros2_ws/src/uel_rover/README.md) |
| 오도메트리, URDF, RealSense, Nav2 를 패키지에 넣는 방법 | [ros2_ws/src/uel_rover/examples/README.md](../ros2_ws/src/uel_rover/examples/README.md) |
| Foxglove 앱 설치, 접속, 패널 구성, Teleop, 문제 해결 | [gcs/README.md](../gcs/README.md) |

## 무엇을 어디에 넣는가

| 추가할 것 | 위치 | 추가로 할 일 |
|---|---|---|
| 새 C++ 노드 | `ros2_ws/src/uel_rover/src/*.cpp` | `CMakeLists.txt` 에 `add_executable` / `target_link_libraries` 추가, `install(TARGETS ...)` 목록에 추가. 새 메시지 패키지를 쓰면 `find_package` 와 `package.xml` `<depend>` 도 추가 |
| launch 파일 | `launch/*.launch.py` | 없음. 디렉터리째 설치된다 |
| 파라미터 yaml | `config/*.yaml` | 없음. 디렉터리째 설치된다 |
| 새 디렉터리 (예: `urdf/`) | 패키지 루트 | `CMakeLists.txt` 의 `install(DIRECTORY launch config ...)` 에 추가 |
| 외부 ROS 패키지 의존 | `package.xml` | `<depend>` 또는 `<exec_depend>` 로 선언하면 `setup.sh` 의 rosdep 단계가 설치한다 |
| 제어보드 동작 변경 | `firmware/OpenRB-150/OpenRB-150.ino` | 아래 "펌웨어를 수정할 때" 참고 |
| GCS 화면 | `gcs/foxglove/uel_rover_gcs.json` | Foxglove 에서 편집 후 **Layouts → Export** 로 덮어쓴다 |

수정 후에는 `ros2_ws` 에서 `colcon build --symlink-install` 을 다시 돌리고 `install/setup.bash` 를 source 한다.
`--symlink-install` 덕분에 launch 와 yaml 은 수정만 하면 바로 반영되지만, 파일을 새로 추가했을 때는 다시 빌드해야 한다.

## 참고 구현(examples) 되살리기

빌드되지 않는 참고 구현이 네 묶음 있다. 필요한 것만 골라 넣는다.
절차는 [examples/README.md](../ros2_ws/src/uel_rover/examples/README.md), 패키지 설치는 `setup.sh --with-examples`.

| 디렉터리 | 추가되는 것 | 넣는 방법 |
|---|---|---|
| `odometry/` | `/wheel_velocity` → `/odom`, `odom→base_link` TF | cpp 를 `src/` 로, yaml 을 `config/` 로 복사하고 CMake / package.xml 에 nav_msgs, tf2, tf2_ros 추가 |
| `description/` | URDF, `base_link→camera_link` 및 바퀴 정적 TF | urdf 를 `urdf/` 에 두고 install 목록에 추가, launch 복사 |
| `sensors/` | RealSense D455 → `/scan` | launch / yaml 복사 |
| `navigation/` | Nav2, slam_toolbox 매핑, map_server + AMCL | launch / yaml 복사 |

전체를 되살리는 실행 순서는 bringup → odometry → description → sensors → nav2 → mapping 또는 localization 이다.
TF 트리는 `map → odom → base_link → camera_link, 바퀴`.

## 상위 제어기(조종·자율주행 노드)를 만들 때

- **`/cmd_vel` 에 `geometry_msgs/Twist` 로 발행한다.** `TwistStamped` 가 아니다. Nav2 를 쓰면 `enable_stamped_cmd_vel: false` 가 필요하다.
- **주기적으로 발행한다.** commander 는 명령을 반복 발행하지 않는다. 제어보드 워치독 500 ms 안에 다음 명령이 와야 계속 움직이므로 10 Hz 정도로 보낸다.
- 발행을 멈추면 로버가 선다. commander 가 `command_timeout`(0.5 s) 후 정지 명령을 한 번 보내고, 제어보드 워치독도 같은 시간에 정지한다.
- `linear.x`, `angular.z` 만 쓰인다. 나머지 성분은 버려진다. 상한을 넘는 값은 `max_linear_velocity` / `max_angular_velocity` 로 잘리고, NaN / Inf 는 정지 명령이 된다.
- **`/cmd_vel_out` 에 직접 발행하지 않는다.** commander 의 클램프와 타임아웃을 건너뛴다.
- 드라이브 모드 게이팅은 제어보드가 한다. 소프트웨어에서 `/drive_mode` 를 읽어 다시 막을 필요는 없지만, AUTO 가 아닐 때 명령이 무시된다는 점은 상위 로직에서 알고 있어야 한다.

## 파라미터를 바꿀 때

`config/rover.yaml` 을 고치거나, 복사본을 만들어 `params_file:=` 로 넘긴다.
값 사이에는 다음 관계가 있으므로 한쪽만 바꾸지 않는다.

| 항목 | 값이 묶여 있는 곳 |
|---|---|
| commander `command_timeout` (0.5 s) | 펌웨어 `WATCHDOG_MS` (500). 같게 유지한다 |
| monitor `max_wheel_rpm` (100) | 펌웨어 `MAX_RPM` (50) 의 2배 |
| monitor `link_timeout` (0.5 s) | 제어보드 발행 주기 50 Hz 기준 25 주기 누락 |
| commander 속도 상한 | 펌웨어 `MAX_RPM` 과 바퀴 반지름으로 정해지는 물리 한계 이내 |

## 펌웨어를 수정할 때

- **`Serial.print` 를 쓰지 않는다.** USB 시리얼이 micro-ROS 트랜스포트라 출력이 통신을 깨뜨린다. 시리얼 모니터도 열지 않는다.
- micro_ros_arduino 는 ROS 2 **Jazzy** 릴리스를 쓴다. 로버 쪽 agent 도 `jazzy` 브랜치로 빌드된다.
- 토픽 이름이나 메시지 타입을 바꾸면 commander / monitor / GCS 레이아웃도 같이 바꾼다. 토픽은 `/cmd_vel_out`, `/wheel_velocity`, `/drive_mode`, 노드 이름은 `openrb150`.
- `MAX_RPM` 을 바꾸면 `rover.yaml` 의 `max_wheel_rpm` 을 2배로 맞춘다. `WHEEL_BASE` / `WHEEL_RADIUS` 를 바꾸면 오도메트리 쪽 값과 통일한다 ([알려진 주의사항](troubleshooting.md#알려진-주의사항)).
- 드라이브 모드 판정과 RC / STOP 제어는 micro-ROS 상태 머신보다 먼저, 무조건 실행되는 구조다. 이 순서를 유지해야 조종기가 데드맨 스위치로 작동한다.

## GCS 를 확장할 때

- 패널 추가는 Foxglove 앱에서 하고 레이아웃을 `gcs/foxglove/uel_rover_gcs.json` 으로 내보낸다.
- 로버에서 발행 가능한 토픽을 제한하려는 `client_topic_whitelist` 가 `config/foxglove_bridge.yaml` 에 있지만, apt 의 foxglove_bridge 3.5.0 은 적용하지 않는다. 그래서 **레이아웃에 `/gcs/teleop`, `/gcs/speed_step` 외의 발행 패널을 두지 않는다**는 규칙으로 대신한다.
- 무선 대역폭이 부족하면 `topic_whitelist` 로 GCS 에 보낼 토픽을 줄인다.
- 게임패드 (`teleop_twist_joy`), 카메라 영상, Nav2 목표점 지정 등 확장 예는 [gcs/README.md](../gcs/README.md) 의 "확장" 절.

## 문서 관리

디렉터리마다 README 가 있고, 각 README 는 그 디렉터리의 구성 트리와 인터페이스 표를 담고 있다.
파일이나 토픽, 파라미터를 추가하면 해당 README 의 표도 같이 고친다. 이 문서의 저장소 구조 트리와
[ros-interface.md](ros-interface.md) 의 토픽·파라미터 표도 마찬가지다.
