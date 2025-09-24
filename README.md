# Deepracer 자율 주행: AUTOSAR 적용 프로젝트
**팀명:** 짱짱맨팀  
**주요 성과:** 제22회 임베디드SW경진대회 자율주행 레이싱 부문 최우수상  

---

## 🎥 프로젝트 소개
[![Deepracer 로봇 데모](https://img.youtube.com/vi/P8s7qXrRdfY/hqdefault.jpg)](https://youtu.be/P8s7qXrRdfY)  
➡ 영상 클릭 시, YouTube 재생  

본 프로젝트는 AWS DeepRacer를 실제 자동차와 같은 다중 ECU(전자제어장치) 환경으로 간주하고, **자동차 소프트웨어 표준 플랫폼 AUTOSAR**를 적용하여 개발 프로세스를 표준화하는 것을 목표로 합니다.  

- Adaptive AUTOSAR(AA) 플랫폼 기반 시스템 아키텍처 재설계  
- 단일 ECU 환경의 DeepRacer를 기능별 ECU처럼 설계  
- ROS2와 AUTOSAR 융합 설계로 기존 ROS2 자산 활용 및 안정성 확보  

---

## 🔧 주요 기능
- 🚗 **AUTOSAR 기반 다중 ECU 아키텍처 설계:** 기능별 ECU가 협력하는 실제 자동차 환경처럼 설계, 각 기능을 Adaptive Application(AA)으로 노드화  
- 🔄 **표준화된 데이터 통신:** AA 간 데이터 통신을 AUTOSAR 표준 데이터 타입과 포트를 통해 규격화  
- 🤝 **ROS2와 AUTOSAR 융합:** ROS2 기능 노드를 AA가 관리·제어하도록 구성  
- 📊 **중앙 집중식 시스템 관리:** 실행 관리자(EM)와 상태 관리자(SM)가 시스템 전체 안정성 확보  
- ✨ **효율적인 데이터 처리:** SensorFusion AA가 Lidar와 Camera 데이터를 처리 후 OUTPUT 반환  

---

## 🚀 전체 실행 순서

시스템은 **AUTOSAR 실행 관리자(EM)**가 중앙에서 통제하며, 각 AA가 ROS2 기능 패키지를 자동으로 실행합니다.

### Adaptive Application → ROS2 패키지
- Camera AA → `aws-deepracer-camera-pkg` 실행  
- Lidar AA → `rplidar_ros` 실행  
- SensorFusion AA → `aws-deepracer-sensor-fusion-pkg` 실행  
- Inference AA → `aws-deepracer-navigation-pkg` 실행  
- Servo AA → 관련 패키지 실행  

---

### 실행 예제 (Bash)
```bash
sudo su -
source /opt/ros/foxy/setup.bash
source /opt/intel/openvino_2021/bin/setupvars.sh
source /media/deepracer/wook/deepracer_ws/install/setup.bash
cd /media/deepracer/wook/deepracer_ws

ros2 launch deepracer_launcher deepracer_launcher.py
ros2 launch status_led_pkg status_led_pkg_launch.py 

ros2 service call /status_led_pkg/led_solid deepracer_interfaces_pkg/srv/SetStatusLedSolidSrv "{led_index: 1, color: 'red', hold: 1.0}"

# node나 topic이 안보일 때
ros2 daemon stop
ros2 daemon start

ros2 launch rplidar_ros rplidar_a1_launch.py 

# GPIO 접근 권한 설정
sudo chmod 666 /sys/class/gpio/gpio445/direction
sudo chmod 666 /sys/class/gpio/gpio446/direction
sudo chmod 666 /sys/class/gpio/gpio443/direction

sudo nano /etc/udev/rules.d/99-gpio.rules
sudo udevadm control --reload-rules
sudo udevadm trigger

# 모델 로딩 및 Inference 실행
ros2 service call /inference_pkg/inference_state deepracer_interfaces_pkg/srv/InferenceStateSrv "{start: 1, task_type: 0}"
ros2 service call /inference_pkg/load_model deepracer_interfaces_pkg/srv/LoadModelSrv "{artifact_path: /opt/aws/deepracer/artifacts/bucket31/model.xml, task_type: 0, pre_process_type: 0, action_space_type: 5}"
ros2 service call /inference_pkg/load_model deepracer_interfaces_pkg/srv/LoadModelSrv "{artifact_path: /opt/aws/deepracer/artifacts/bucket31/model.xml, task_type: 1, pre_process_type: 1, action_space_type: 0}"

# Display 문제 해결
xhost +SI:localuser:root
export DISPLAY=:0

# PARA_SDK 환경 변수 설정
export PARA_SDK=/media/deepracer/wook/Para_SDK
export PARA_CORE=$PARA_SDK
export PARA_CONF=$PARA_SDK/etc
export PARA_APPL=$PARA_SDK/opt
export PARA_DATA=$PARA_SDK/var
export LD_LIBRARY_PATH=$PARA_SDK/lib

# 빌드 및 EM 실행
cd /media/deepracer/wook/DeepRacer
cmake -B build -D CMAKE_INSTALL_PREFIX=$PARA_SDK
cmake --build build -j $(nproc)
cmake --install build
```
⚠️ 참고: OpenCV 버전 경고가 발생할 수 있음
/usr/bin/ld: warning: libopencv_imgproc.so.4.2 may conflict with libopencv_imgproc.so.4.5
cd $PARA_SDK/bin
./EM
