# 문제 해결과 주의사항

모르는 용어는 [용어집](glossary.md) 을 본다.

## 문제 해결

| 증상 | 확인할 것 |
|---|---|
| `/dev/ttyACM0` 권한 오류 | `dialout` 그룹 반영 여부 (`id -nG`). `setup.sh` 후 재로그인했는지 |
| launch 가 `micro_ros_agent` 를 못 찾음 | `microros_ws/install/local_setup.bash` 를 source 했는지 |
| `/diagnostics` 가 STALE | 제어보드 USB 연결, `serial_port` 인자, agent 로그. 시리얼 모니터가 포트를 잡고 있지 않은지 |
| `/diagnostics` 가 ERROR | 받다가 끊긴 것. USB 케이블, 제어보드 전원, agent 재시작 여부 |
| `/cmd_vel` 을 보내도 안 움직임 | 조종기 스위치가 AUTO 인지, `/drive_mode` 가 `"AUTO"` 인지, 발행 주기가 500 ms 이내인지, `/cmd_vel_out` 이 나오는지 |
| GCS 접속 불가 | `gcs.launch.py` 실행 여부, 로버 IP, 8765/tcp 방화벽 |

GCS 관련 문제는 [gcs/README.md](../gcs/README.md), 제어보드 문제는 [firmware/OpenRB-150/README.md](../firmware/OpenRB-150/README.md) 의 해당 절을 함께 본다.

## 알려진 주의사항

| 항목 | 내용 |
|---|---|
| 바퀴 상수 불일치 | 펌웨어 `WHEEL_BASE` 0.30 m / `WHEEL_RADIUS` 0.065 m 와 `examples/odometry/odometry.yaml` 의 0.915 m / 0.0877 m (실측 보정값) 가 다르다. AUTO 모드에서 Nav2 가 명령한 속도와 실제 속도가 어긋나므로 배포 전에 한쪽으로 통일한다. |
| GCS 발행 제한 미적용 | `client_topic_whitelist` 는 foxglove_bridge 3.5.0 에서 읽히기만 하고 적용되지 않는다. GCS 가 어떤 토픽이든 발행할 수 있다. |
| micro_ros_agent 는 소스 빌드 | apt 패키지가 없어 `microros_ws/` 에 별도 빌드한다. 세 워크스페이스를 모두 source 해야 launch 가 agent 를 찾는다. |
| 시리얼 모니터 금지 | 제어보드 USB 시리얼은 micro-ROS 전용이다. Arduino 시리얼 모니터나 다른 프로그램이 잡고 있으면 agent 가 붙지 못한다. |
| examples 의 `localization.yaml` | `global_costmap` static 오버레이는 어떤 노드에도 로드되지 않는다(비활성). 저장맵 기반 전역 계획을 쓰려면 planner_server 에 넘겨야 한다. |
