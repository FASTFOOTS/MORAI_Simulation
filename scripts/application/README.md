# application.py
설정하고자 하는 robot의 이동 위치 좌표와 방향을 설정하여 action server에게 request하는 코드이다. 


**1. import module**
```python
import rospy
import actionlib
from move_base_msgs.msg import MoveBaseAction, MoveBaseGoal
```
- ROS에서 python 사용을 위한 rospy import
- actionlib은 ROS에서 제공하는 action server와 action client를 구현하기 위한 라이브러리
- actionlib은 비동기적으로 실행되는 작업들을 효과적으로 관리하고, 작업의 상태, 진행 상황 및 결과를 효율적으로 다룰 수 있게 해주는 기능 제공
- move_base_msgs package는 ROS에서 로봇의 이동 관리를 위한 message와 action message를 정의하는 package
- move_base_msgs package는 로봇의 이동을 명령하고 이동 상태를 나타내기 위한 여러 가지 message type을 포함
- MoveBaseAction은 이동 action message의 형식을 정의하는 클래스
- MoveBaseAction class는 로봇의 이동 action에 대한 명세와 결과를 다루는 데 사용
- MoveBaseGoal은 이동 목표 설정 message의 type을 정의하는 클래스
- MoveBaseGoal class는 로봇의 이동 목표를 정의하는 데 사용되며, 로봇이 어디로 이동해야 하는지를 지정하는 데 사용

**2. init node**
```python
rospy.init_node('application', anonymous=True)
```
- application이라는 이름의 node를 생성
- anonymous 매개변수를 True로 사용하여 여러 개의 같은 node를 실행하고자 하는 경우, node이름 뒤에 고유한 식별자를 추가하여 같은 node를 여러번 실행하여도 이름 충돌이 없게 함

**3. create action client**
```python
client = actionlib.SimpleActionClient('move_base',MoveBaseAction)
```
- move_base라는 action server에 대한 action client를 client라는 이름으로 선언
- client는 MoveBaseAction action message type의 action을 통해 로봇의 이동 목표를 설정하고 관리 
- actionlib.SimpleActionClient은 ROS의 actionlib library에서 제공하는 class로서, action server와 간단한 방식으로 통신할 수 있도록 도와주는 도구
- actionlib.SimpleActionClient은 지정된 action server에 request을 보내고, server로부터 response을 받으며 비동기적인 작업으로 작업 request 후 다른 작업을 수행할 수 있으며 action의 상태와 결과 모니터링이 가능 

```python
client.wait_for_server()
```
- wait_for_server method는 action client와 action server간 연결이 설정될 때까지 대기하여 연결 상태 확인 후 안정적 통신을 가능하게 함

**4. create goal instance**
```python
goal = MoveBaseGoal()
```
- 이동 목표 설정 message의 type을 정의하는 클래스의 instance를 goal이라는 이름으로 생성

    **MoveBaseGoal message info image**

    ![Screenshot from 2023-08-21 11-22-53](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108657061/5af36068-7df1-43a5-8d01-77de3649542b)

- header: 메시지 헤더 정보를 담고 있는 부분
    - seq: message의 시퀀스 번호  
    - stamp: message에 대한 어떤 이벤트나 데이터가 발생한 시간을 나타내는 정보를 나타내는 타임스탬프
    - frame_id: 로봇의 이동 목표 위치가 어떤 좌표 프레임에서 표현되는지 지정하는 좌표 frame ID
- pose: 로봇의 이동 목표 위치와 방향을 담고 있는 부분
    - position: 3차원 좌표를 가지는 메시지로서 x, y, z 좌표
    - orientation: 로봇의 방향을 나타내는 quaternion으로 x, y, z, w 값을 지님

**5. change robot motion**
```python
goal.target_pose.header.frame_id = "map"
goal.target_pose.header.stamp = rospy.Time.now()
goal.target_pose.pose.position.x = 1.0
goal.target_pose.pose.orientation.w = 1.0
```
- frame_id를 map으로 설정하여 map frame 기준으로 위치 설정
- 현재 시간을 사용하여 mseeage의 timestamp를 기록 
- 목표 위치를 x축으로 1m 이동하도록 설정
- 로봇의 방향을 w를 사용하여 설정 

**6. activate goal message**
```python
client.send_goal(goal)
wait = client.wait_for_result()
```
- send_goal method는 clent가 goal message를 action server에 전송
- wait_for_result method는 client가 action server의 작업이 마무리될때 까지 대기하며 action server의 동작 수행 성공 여부를 bool 값으로 저장
