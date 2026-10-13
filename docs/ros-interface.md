# ROS 2 노드와 인터페이스

로버 컴퓨터에서 도는 노드, 주고받는 토픽, 조정할 수 있는 파라미터. 모르는 용어는 [용어집](glossary.md) 을 본다.

## 노드

| 노드 | 실행되는 곳 | 하는 일 |
|---|---|---|
| `commander` | 로버 컴퓨터 | `/cmd_vel` 을 받아 속도 상한과 타임아웃을 적용해 `/cmd_vel_out` 으로 내보낸다 |
| `monitor` | 로버 컴퓨터 | 제어보드 링크·모드·바퀴 속도를 감시해 `/diagnostics` 로 알린다 |
| `gcs_teleop` | 로버 컴퓨터 | GCS 의 방향 입력 × 속도 단계를 `/cmd_vel` 로 바꾼다 |
| `micro_ros_agent` | 로버 컴퓨터 | 제어보드의 USB 시리얼 통신을 ROS 2 토픽으로 중계한다 |
| `foxglove_bridge` | 로버 컴퓨터 | 토픽을 WebSocket 으로 GCS 에 내보낸다 |
| `openrb150` | 제어보드 (펌웨어) | 바퀴를 구동하고 `/wheel_velocity`, `/drive_mode` 를 내보낸다 |

## 토픽

| 토픽 | 타입 | 발행 → 구독 | 비고 |
|---|---|---|---|
| `/cmd_vel` | `geometry_msgs/Twist` | 상위 제어기 → commander | 사용자가 발행하는 유일한 입력 |
| `/gcs/teleop`, `/gcs/speed_step` | `geometry_msgs/Twist`(방향 ±1), `std_msgs/Int8`(±1) | GCS → gcs_teleop | GCS 조종 입력. gcs_teleop 이 `/cmd_vel` 로 바꾼다 |
| `/gcs/speed` | `std_msgs/String` | gcs_teleop → GCS | 현재 고른 속도 표시, 1 Hz |
| `/cmd_vel_out` | `geometry_msgs/Twist` | commander → 제어보드 | 클램프된 명령. 직접 발행하지 않는다 |
| `/wheel_velocity` | `std_msgs/Float32MultiArray` `[vL_rpm, vR_rpm]` | 제어보드 → monitor | 50 Hz |
| `/drive_mode` | `std_msgs/String` `"RC"/"AUTO"/"STOP"` | 제어보드 → monitor | 50 Hz |
| `/diagnostics` | `diagnostic_msgs/DiagnosticArray` | monitor → GCS, rqt | 상태명 `uel_rover/control_board`, 1 Hz |

## 파라미터

`config/rover.yaml` 에서 바꾼다. 값 사이의 관계는 [개발 가이드](development.md#파라미터를-바꿀-때) 참고.

| 노드 | 파라미터 | 기본값 | 설명 |
|---|---|---|---|
| commander | `max_linear_velocity` | 0.306 m/s | RC 풀 입력 = 바퀴 45 rpm, 서보 Velocity Limit 바로 아래 |
| commander | `max_angular_velocity` | 2.04 rad/s | 위와 같음 |
| commander | `command_timeout` | 0.5 s | |
| monitor | `link_timeout` | 0.5 s | |
| monitor | `max_wheel_rpm` | 100 | |
| monitor | `diagnostics_period` | 1.0 s | |
| gcs_teleop | `max_linear_speed` | 0.306 m/s | |
| gcs_teleop | `max_angular_speed` | 2.04 rad/s | |
| gcs_teleop | `speed_levels` | 10 | |
| gcs_teleop | `initial_level` | 4 | |

자세한 동작은 [ros2_ws/src/uel_rover/README.md](../ros2_ws/src/uel_rover/README.md).
