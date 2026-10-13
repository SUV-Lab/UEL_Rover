# 초기 설정

세 곳을 각각 준비한다. 제어보드 → 로버 컴퓨터 → GCS PC 순서가 편하다.
모르는 용어는 [용어집](glossary.md) 을 본다. 준비를 마치면 [사용법](usage.md) 으로 넘어간다.

## 1. 제어보드 (OpenRB-150)

Arduino IDE 에서 펌웨어를 올린다. 자세한 절차는 [firmware/OpenRB-150/README.md](../firmware/OpenRB-150/README.md).

1. Boards Manager 에서 **OpenRB-150** 보드 패키지 설치.
2. 라이브러리 설치: **DynamixelShield** (Library Manager), **micro_ros_arduino** (ROS 2 **Jazzy** 릴리스 zip 수동 설치).
3. `firmware/OpenRB-150/OpenRB-150.ino` 상단 `#define` 상수를 실기에 맞게 확인한다.
   모터 ID (`DXL_LEFT_ID` / `DXL_RIGHT_ID`), 우측 모터 반전 (`RIGHT_MOTOR_INVERTED`), `MAX_RPM`,
   `WHEEL_BASE`, `WHEEL_RADIUS`.
4. 보드를 OpenRB-150 으로 선택하고 업로드.
5. 업로드 후 **시리얼 모니터를 열지 않는다.** USB 시리얼이 micro-ROS 트랜스포트다.

수신기 배선: CH1 → D6, CH2 → D7, CH7 → D8, GND 공통. 다이나믹셀은 TTL 포트 직결 (ID 1 좌, 2 우).

## 2. 로버 컴퓨터 (Jetson)

전제: Ubuntu 24.04 (noble), sudo 사용 가능, 인터넷 연결.

```bash
git clone https://github.com/Porofly/UEL_Rover.git
cd UEL_Rover
./scripts/setup.sh --bashrc                      # 기본 구성
./scripts/setup.sh --with-examples --bashrc      # examples/ 용 패키지까지 설치할 때
```

스크립트가 하는 일. 이미 끝난 단계는 건너뛰므로 재실행해도 안전하다.

| 단계 | 내용 |
|---|---|
| 1 | ROS 2 Jazzy 설치 (`/opt/ros/jazzy` 가 없을 때만) |
| 2 | 빌드 도구 설치, rosdep 초기화 |
| 3 | `ros2_ws` 의존성 설치 (rosdep). `foxglove_bridge` 포함 |
| 4 | `--with-examples` 일 때 RealSense, Nav2, slam_toolbox 등 설치 |
| 5 | micro-ROS agent 소스 빌드 → `microros_ws/` |
| 6 | `ros2_ws` colcon 빌드 (Release) |
| 7 | 사용자를 `dialout` 그룹에 추가 (제어보드 USB 시리얼 접근) |

| 옵션 | 효과 |
|---|---|
| `--with-examples` | examples/ 에 필요한 ROS 패키지도 설치 |
| `--skip-agent` | micro-ROS agent 빌드 생략 |
| `--skip-build` | ros2_ws 빌드 생략 |
| `--bashrc` | `~/.bashrc` 에 환경 source 블록 추가 (한 번만 추가됨) |

완료 후 **재로그인**(또는 `newgrp dialout`) 해야 `/dev/ttyACM0` 에 접근할 수 있다.
`--bashrc` 를 쓰지 않았다면 새 셸마다 다음을 source 한다. 세 줄 모두 필요하다.
`microros_ws` 를 빼면 launch 가 `micro_ros_agent` 를 찾지 못한다.

```bash
source /opt/ros/jazzy/setup.bash
source ~/UEL_Rover/microros_ws/install/local_setup.bash
source ~/UEL_Rover/ros2_ws/install/setup.bash
```

스크립트 없이 수동으로 빌드하려면 [ros2_ws/src/uel_rover/README.md](../ros2_ws/src/uel_rover/README.md) 의 "빌드" 절을 따른다.

## 3. GCS PC

ROS 2 가 필요 없다. 브릿지는 로버에서 돈다.

1. Foxglove 데스크톱 앱 설치 (https://foxglove.dev/download). 웹 버전은 `ws://` 접속이 안 되므로 데스크톱 앱을 쓴다.
2. 로버와 같은 네트워크에 연결하고 로버 IP 를 확인한다.
3. 앱에서 **Layouts → Import from file** 로 `gcs/foxglove/uel_rover_gcs.json` 을 불러온다.
