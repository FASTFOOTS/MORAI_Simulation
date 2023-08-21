# pub_odom.py
변환된 로봇의 위치 및 방향 정보를 구하고 이를 이용해 msg를 재구성 후 message publish하는 코드이다. 

**1. import module**
```python
import os
import sys
import rospy
from math import pi,sin,cos
from scout_msgs.msg  import ScoutStatus
from nav_msgs.msg import Odometry
import tf
```
- os module은 python에서 os와 관련된 작업을 수행하기 위한 기능을 제공 module
- sys module은 python interpreter와 관련된 기능을 제공하는 내장 module
- math module에서 수학 관련 연산들을 import
- ScoutStatus는 로봇의 상태 정보를 담는 메시지로, 로봇의 모터 상태, 이동 관련 정보 등을 포함
- 이는 주로 로봇의 현재 상태 및 module 상태를 실시간으로 전달하여 로봇 제어 및 모니터링에 활용
- nav_msgs는 ROS에서 사용되는 navigation과 관련된 메시지 타입을 정의함
- Odometry는 로봇의 odometry 정보를 담으며 로봇의 위치, 속도, 방향 등을 포함
- tf module은 ros에서 transform 관련 작업을 수행하기 위한 module
- 이는 로봇의 좌표 변환 등과 관련된 작업으로, 로봇의 다양한 부분 간의 상대적인 위치와 방향을 변환 관리하는데 사용
**Message Type**
  - scout_msg message type
    ![Screenshot from 2023-08-21 16-15-19](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108657061/566148d2-85b6-4e79-aff1-76dfffbc36d7)

  - nav_msg message type
    ![Screenshot from 2023-08-21 15-45-37](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108657061/59f3a170-e2b8-4584-84cf-0a4d71d44c5d)

**2. init**
```python
rospy.init_node('scout_odom', anonymous=True)
```
- scout_odom이란 이름의 node 생성

```python
self.prev_time=rospy.get_rostime()
```
- 현재의 ROS 시스템 시간을 가져와서 클래스 내의 self.prev_time 변수에 할당

```python
self.is_status=False
```
- 로봇의 상태 여부를 나타내는 플래그를 false로 선언

```python
self.odom_pub = rospy.Publisher('/odom',Odometry, queue_size=1)
```
- odom이라는 topic에 Odometry라는 message를 publish하는 publisher를 생성

```python
rospy.Subscriber('/scout_status', ScoutStatus, self.status_callback)
```
- scout_status라는 topic에 ScoutStatus라는 message를 subscribe하는 subscriber를 생성 

```python
odom_msg=Odometry()
```
- Odometry class에 대한 instance를 선언

```python
odom_msg.header.frame_id='/odom'
odom_msg.child_frame_id='/base_link'
```
- frame_id를 odom으로 설정하여 odom frame을 이용한 로봇 위치 표현
- child_frame_id는 로봇의 기반 프레임에서 어떤 부분의 위치 정보를 포함하고 있는지를 지정하는 역할
- 즉, parent_frame으로부터의 상대적인 위치와 변화를 나타내는 frame을 의미

**3. Main Code**
```python
current_time = rospy.get_rostime()
```
- Ros system의 현재 시간을 current_time으로 선언

```python
interval_time=(current_time-self.prev_time).to_sec()
```
- 현재 시간과 이전 시간의 차를 초 단위로 나타내어 시간의 차를 interval_time 변수에 선언 

```python
linear_x=self.status_msg.linear_velocity
```
- 선 속도에 대한 값을 linear_x 변수에 저장 

```python
angular_z=self.status_msg.angular_velocity
```
- 각속도에 대한 값을 angular_z 변수에 저장 

```python
x+=linear_x*cos(heading_rad)*interval_time
y+=linear_x*sin(heading_rad)*interval_time
```
- sin과 cos을 이용하여 선 속도를 사용한 현재 robot의 위치에 대한 x, y값 표현 

```python
heading_rad+=angular_z*interval_time
```
- 로봇이 향하는 방향에 대한 값을 각 속도와 시간을 사용하여 head부분의 방향 각도 표현

```python
q= tf.transformations.quaternion_from_euler(0, 0,heading_rad)
```
- 오일러 각도로부터 쿼터니언(quaternion)을 생성하는 역할 수행 
- 각 인자는 변환할 오일러 각도
- 0은 롤(roll) 각도, 0은 피치(pitch) 각도, heading_rad는 요(pitch) 각도

```python
br = tf.TransformBroadcaster()
```
- TransformBroadcaster는 로봇의 transform 정보를 관리하고 broadcast하는 데 사용
- 로봇의 위치와 방향 정보를 broadcast시, 다른 노드나 시스템에서 이 정보를 받아서 필요한 변환 작업이나 위치 추정 작업 수행 가능 

```python
 br.sendTransform((x,y,z), q, current_time, "base_link", "odom")
```
- (x, y, z): 로봇의 위치 표현
- q: 오일러에서 quaternion으로 변환된 정보를 사용 
- 로봇의 변환 정보, 현재 시간, 대상 frame, 출발 frame을 각각의 인자로 넣음
- 위 정보들을 sendTransform 함수를 사용하여 다른 노드나 시스템에서 이 정보를 수신하여 로봇이나 오브젝트 간의 관계를 파악하고 활용하는데 사용

```python
odom_msg.header.stamp=current_time
odom_msg.pose.pose.position.x=x
odom_msg.pose.pose.position.y=y
odom_msg.pose.pose.position.z=z
odom_msg.twist.twist.linear.x=linear_x
odom_msg.twist.twist.angular.z=angular_z
odom_msg.pose.pose.orientation.x=q[0]
odom_msg.pose.pose.orientation.y=q[1]
odom_msg.pose.pose.orientation.z=q[2]
odom_msg.pose.pose.orientation.w=q[3]
```
- odom의 message를 설정한 값들을 이용하여 재구성

```python
self.odom_pub.publish(odom_msg)
```
- 변환된 정보가 저장된 odom_msg를 topic에 publish

```python
self.prev_time=current_time
```
- 사용한 현재 시간을 이전 시간으로 설정 


