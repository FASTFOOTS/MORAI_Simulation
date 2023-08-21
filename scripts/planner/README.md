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

## utils.py

```python
if current_waypoint+30 > len(ref_path.poses) :
    last_local_waypoint= len(ref_path.poses)
else :
    last_local_waypoint=current_waypoint+30
```
- current_waypoint변수는 path maker에서 작성된 좌표 리스트의 인덱스를 가지고 있다.
- current_waypoint 로부터 앞으로 30개의 점까지 인식하여 차선을 계산함

### waypoint 10
![w10](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/edf3b41d-8629-4a43-8962-a55e380aa705)
### waypoint 30
![w30](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/50041763-f21e-47c6-96f7-1e5b946d5ece)
### waypoint 50
![w50](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/dd80a121-d9ef-4cf7-9b54-e57dbb555033)


```python
def latticePlanner(ref_path,global_vaild_object,vehicle_status,current_lane):
    out_path=[]
    selected_lane=-1
    lattic_current_lane=current_lane
    look_distance=int(vehicle_status[3]*3.6*0.2*2)
    if look_distance < 3 :
        look_distance=1     #min 5m
    if look_distance > 5 :
        look_distance=5  
    

    ...


    lane_weight=[6,4,2,0,2,4,6] #reference path 
    collision_bool=[False,False,False,False,False,False,False]

    if len(global_vaild_object)>0:

        for obj in global_vaild_object :
            if  obj[0]==2 or obj[0]==1 or obj[0] == 0: 
                for path_num in range(len(out_path)) :
                    
                    for path_pos in out_path[path_num].poses :
                        
                        dis= sqrt(pow(obj[1]-path_pos.pose.position.x,2)+pow(obj[2]-path_pos.pose.position.y,2))

                        if dis<1.5:
                            collision_bool[path_num]=True
                            lane_weight[path_num]=lane_weight[path_num]+100
                            break
    else :
        print("No Obstacle")

    selected_lane=lane_weight.index(min(lane_weight))
    print(lane_weight,selected_lane)
    all_lane_collision=True

```
- 이 코드에서 가장 중요한 부분
- latticePlanner 함수는 path_maker에서 만든 경로와, 시뮬레이션 내의 장애물의 정보, 자동차의 상태(속도, 위치), 선택한 차선을 입력으로 받아 자신이 앞으로 가야할 차선을 새롭게 선택하는 함수이다.
- look_distance는 현재 로봇의 속력(km/h)에 비례하여 전방을 바라본다. 최소값은 3, 최대값을 5로 설정

### look distance 3~5
![w30](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/a12861b9-6ca7-44f4-996b-76499610576e)
### look distance 1~2
![w30l12](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/976d41ee-82f9-47d2-ae70-74060c9ea3bb)

- 7개의 차선들의 각 가중치는 다음과 같다. lane_weight=[6,4,2,0,2,4,6] 이 중 가장 낮은 weight를 갖는 차선을 선택하여 추종을 시작한다.
- 장애물의 갯수가 1개 이상이라면 장애물을 회피하도록 차선을 선택하는 조건문이 작동한다.
- 해당 조건문에서는 7개의 차선들과 장애물 사이의 거리를 각각 계산하고 장애물이 차선과 1.5m 이내에 있으면 해당 차선의 weight를 매우 높여 해당 차선을 선택하지 않도록 한다.


