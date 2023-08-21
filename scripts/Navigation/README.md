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
  - 센서, 액추에이터, 프레임 등 로봇의 하드웨어 변환에 대한 코드
  - 로봇과 물체 사이의 정확한 거리 탐지를 위해 실행
- pub_odom
  - 로봇의 센서데이터를 기반으로 odometry을 발행
  - 정확한 로봇의 위치를 파악하기 위해 실행
  
### pointcloud_to_laserscan
- 3D 포인트 클라우드 데이터를 2D 레이저 스캔 데이터로 변환
- 레이저를 활용하여 로봇이 주변 환경 탐지 및 장애물 감지, 회피할 수 있게 하기 위해 실행
  
### navigation
- 시뮬레이션 상에서 원하는 위치로 이동하기 위한 navigation 진행
```python
<launch>
  <!-- map file -->
  <arg name="map_file" default="$(find kw_tf)/maps/map.yaml"/>
    
  <!-- map server -->
  <node name="map_server" pkg="map_server" type="map_server" args="$(arg map_file)" />
  
  <!-- amcl -->
  <include file="$(find scout_mini_2dnav)/launch/amcl.launch" />
  
  <!-- move base -->
  
  <include file="$(find scout_mini_2dnav)/launch/move_base_only.launch" />
  
  <!-- run rviz -->
  <node name="rviz" pkg="rviz" type="rviz" args="-d $(find kw_tf)/rviz/navigation.rviz" />
</launch>
```
- map file & map server
  - navigation에 사용할 맵의 경로를 지정하고, 해당 맵 파일을 ROS 네트워크에 게시
- amcl
  - 파라미터를 기반으로 로봇의 현재 위치를 추정
  - 로봇의 센서 데이터로 얻는 정보와 맵이 일치하도록 수정
  - 이를 바탕으로 로봇의 경로 추적
- move base
  - 
- run rviz
  
