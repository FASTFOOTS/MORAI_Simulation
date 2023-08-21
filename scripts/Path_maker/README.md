## sample_node.launch

```python
<launch>
	<node pkg="scout_ros" type="scout_path_maker.py" name="maker" args="path kw 302473.539105 4123735.71315" output="screen" />
</launch>
```
- scout_ros 패키지 내부에 있는 scout_path_maker.py를 실행하여 maker라는 이름의 노드를 작성. 
- 시뮬레이션에서 gps가 받아오는 데이터의 초기 위치를 맞추기 위해 (x,y) = (302473.539105 4123735.71315)로 초기화

## scout_path_maker.py

```python
rospy.init_node('path_maker', anonymous=True)
```
- path_maker 노드 생성 및 초기화

```python
arg = rospy.myargv(argv=sys.argv)
self.path_folder_name=arg[1]
self.make_path_name=arg[2]
self.x_offset=float(arg[3])
self.y_offset=float(arg[4])
```
- 런치 파일에서 입력한 path kw 302473.539105 4123735.71315 변수들을 arg 배열에 담고 각각을 변수에 저장함.

```python
rospy.Subscriber("/gps", GPSMessage, self.gpsCB)
```
- MORAI 시뮬레이션에서 발행한 /gps 토픽 구독.
- gps 토픽의 데이터 타입은 GPSMessage
- GPSMessage 타입은 morai_msgs에서 직접 만들었다.
- 콜백함수는 self.gpsCB

```python
def path_make(self):
    x=self.xy_zone[0]- self.x_offset
    y=self.xy_zone[1]- self.y_offset
    z=0
    distance=sqrt(pow(x-self.prev_x,2)+pow(y-self.prev_y,2))
    
    if distance > 0.5:
        data='{0}\t{1}\t{2}\n'.format(x,y,z)
        self.f.write(data)
        self.prev_x=x
        self.prev_y=y
        print(x,y)
```
- 로봇이 0.5m 이상 이동했을 경우에만 x,y 좌표를 data에 저장함

```python
def gpsCB(self, data):
    self.xy_zone = self.proj_UTM(data.longitude, data.latitude)
    self.is_gps = True
```
- 구독한 gps 데이터는 위도와 경도로 이루어져 있으며 이 값을 x,y좌표계로 변환하는 함수

![gps_path_making](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/d614078e-61a8-4ec0-9a89-083a7690db92)
