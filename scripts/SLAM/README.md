# SLAM

### 실행 코드 목록
```
    roslaunch rosbridge_server rosbridge_websocket.launch
    roslaunch kw_tf tf_setting.launch
    roslaunch pointcloud_to_laserscan sample_node.launch
    roslaunch gmapping slam_gmapping_pr2.launch
    rosrun map_server map_saver
```
### 실행 코드 설명
```
    roslaunch rosbridge_server rosbridge_websocket.launch
```
- ROS와 외부 application이 통신할 수 있는 기능을 가진 rosbridge_websocket.launch를 실행하여 ROS와 Morai Simulator간의 통신이 가능하도록 연결합니다
```
    roslaunch kw_tf tf_setting.launch
```
- 2개의 노드가 담긴 tf_setting.launch를 실행하여 1. 로봇에 장착된 lidar의 위치 계산 2. odometry 기반으로 현재 로봇의 자세를 추정을 할 수 있도록 합니다  

```
    roslaunch pointcloud_to_laserscan sample_node.launch
```
- 1개의 노드가 담긴 sample_node.launch를 실행하여 lidar에서 받아오는 3D pointcloud 값을 2D laserscan값을 변환할 수 있도록 합니다

```
    roslaunch gmapping slam_gmapping_pr2.launch
```
- 1개의 노드가 담긴 slam_gamapping_pr2.launch를 실행하여 lidar로 부터 값이 들어오면 매핑 알고리즘에 데이터를 전달하여 처리하고, scan data와 odometry정보를 기반으로 map update 및 위치추정을 합니다

```
    rosrun map_server map_saver
```
- map_saver를 실행하여 작성된 map을 저장합니다

![mapping4](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/1af207fc-7c60-4485-bf01-830d8a4a9e33)


