## scout_planner.py

```python
def __init__(self):
    self.position = Vector3()
    self.heading = 0.0
    self.velocity = Vector3()
```
- 클래스에서 사용할 변수 선언.
- Vector3의 구조는 float64 x, float64 y, float64 z

```python
global_path_pub= rospy.Publisher('/global_path',Path, queue_size=1) ## global_path publisher
local_path_pub= rospy.Publisher('/local_path',Path, queue_size=1) ## local_path publisher
ctrl_pub = rospy.Publisher('/cmd_vel',Twist, queue_size=1) ## Vehicl Control
ctrl_msg= Twist()
odom_pub = rospy.Publisher('odom',Odometry, queue_size=1)
self.status_msg = scout_status()
```
- global path와 local path, 로봇 구동을 위한 cmd_vel, 로봇의 움직인 거리 odom 토픽들을 발행

```python
for i in range(1,8):            
    globals()['lattice_path_{}_pub'.format(i)]=rospy.Publisher('lattice_path_{}'.format(i),Path,queue_size=1)  
```
- lattice_path_1~7 까지 차선 7개에 대한 토픽을 발행.

```python
rospy.Subscriber("/Object_topic", ObjectStatusList, self.objectInfoCB) ## Object information Subscriber
rospy.Subscriber("/gps", GPSMessage, self.gpsCB)
self.image_sub = rospy.Subscriber("/imu", Imu, self.imuCB)
self.ego_sub = rospy.Subscriber("/scout_status",ScoutStatus, self.statusCB)
```
- 장애물의 타입과 좌표를 가지고 있는 Object_topic 을 구독.
- 로봇의 위도와 경도를 가지고 있는 gps 토픽을 구독하고 콜백함수 gpsCB를 통해 cartesian coordinate 로 변환
- 로봇의 quaternion을 가지고 있는 imu 토픽을 구독하고 콜백함수 imuCB를 통해 euler coordinate로 변환
- 로봇의 위치, 속도를 가지고 있는 scout_status 토픽을 구독하고 선속도를 받아와 저장

```python
def objectInfoCB
...
```
- /Object_topic 토픽을 통해 장애물을 전부 cartesian coordinate 에 표현함

```python
lattice_current_lane=3
```
- 7개의 차선 중 현재 차선을 3번째 중앙 차선으로 선택함.

```python
local_path,self.current_waypoint=findLocalPath(self.global_path,self.status_msg) 
```
- local_path와 current_waypoint를 가지고 local path를 계산함.

```python
lattice_path,selected_lane=latticePlanner(local_path,global_obj,vehicle_status,lattice_current_lane)
```
- 7개의 차선들과, 선택한 차선을 이용해 추종할 차선을 선택하는 함수를 실행.

```python
pure_pursuit.getPath(local_path) ## pure_pursuit 알고리즘에 Local path 적용
pure_pursuit.getEgoStatus(self.status_msg) ## pure_pursuit 알고리즘에 차량의 status 적용
ctrl_msg.angular.z=-pure_pursuit.steering_angle()
cc_vel = self.cc.acc(local_obj,self.status_msg.velocity.x,vel_profile[self.current_waypoint],self.status_msg) ## advanced cruise control 적용한 속도 계획
target_velocity = cc_vel

ctrl_msg.linear.x= max(target_velocity,0)
```
- 선택한 차선을 추종하기 위해 pure pursuit 알고리즘을 선택
- 해당 알고리즘을 통해 각 바퀴의 속도를 계산

---
---
---
