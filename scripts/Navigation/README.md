# Navigation
로봇이 navigation을 진행하기 위해서는 크게 localization과 pathplanning이라는 과정을 거쳐야한다.
- localization
  - 로봇이 주어진 지도 상에서 자신의 위치를 파악하는 기술
- pathplanning
  - 로봇이 지도 상에서 주어진 목표 지점에 도달하기 위한 경로를 생성하는 기술

**launch 파일 실행 목록**

```python
roslaunch rosbridge_server rosbridge_websocket.launch
roslaunch kw_tf tf_setting.launch
roslaunch pointcloud_to_laserscan sample_node.launch
roslaunch kw_tf navigation.launch  
```

### rosbridge_server 실행
- ROS와 다른 어플리케이션(MORAISim) 간의 통신을 가능하게 하는 브릿지 역할

### tf_setting
- scout 메세지를 기반으로 로봇의 위치 추적하여 오돔으로 발행
```python
<launch>
    <node pkg="tf2_ros" type="static_transform_publisher" name="base_link_to_lidar" args="0 0 0.1 0 0 0 1 base_link lidar" />
    <node pkg="kw_tf" type="pub_odom.py" name="pub_odom" output="screen"/>
</launch>
```
- static_transform_publisher
  - ROS의 패키지인 tf2_ros에 포함된 노드 중 하나
  - slam을 통한 
-
- ROS의 패키지인 tf2_ros에 포함된 노드 중 하나입니다. 이 노드는 ROS의 tf2 시스템을 사용하여 로봇 또는 시스템의 정적인(변하지 않는) 변환 관계(static transform)를 생성하고 발행하는 데 사용

### pointcloud_to_laserscan
- 로봇이 레이저를 활용하여 주변 환경 인식, 클라우드 데이터를 레이저 스캔 데이터로 변환

### navigation
- 시뮬레이션 상에서 원하는 위치로 이동하기 위한 navigation 진행
