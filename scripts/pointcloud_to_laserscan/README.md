# 1. launch

### sample_node.launch

```python
<node pkg="pointcloud_to_laserscan" type="pointcloud_to_laserscan_node" name="pointcloud_to_laserscan">
```

- pointcloud_to_laserscan 패키지에서 pointcloud_to_laserscan 이라는 이름의 노드를 생성한다. 이 노드는 scr 폴더 내부에 있는 pointcloud_to_laserscan_node.cpp를 가져온다.

```python
        <remap from="cloud_in" to="lidar3D"/>
        <remap from="scan" to="scan"/>
```
- cloud_in과 scan 토픽을 각각 /lidar3D, /scan 토픽으로 이름을 바꾼다.

```python
        <rosparam>
            target_frame: lidar # Leave disabled to output scan in pointcloud frame
            transform_tolerance: 0.01
            min_height: -0.1
            max_height: 0.5

            angle_min: -3.141592 # -M_PI/2
            angle_max: 3.141592 # M_PI/2
            angle_increment: 0.0087 # M_PI/360.0
            scan_time: 0.3333
            range_min: 0.45
            range_max: 30
            use_inf: true
            inf_epsilon: 1.0

            # Concurrency level, affects number of pointclouds queued for processing and number of threads used
            # 0 : Detect number of cores
            # 1 : Single threaded
            # 2->inf : Parallelism level
            concurrency_level: 1
        </rosparam>
```
- 해당 런치파일에서 실행될 노드에서 사용하는 변수들에 사용될 값을 초기화했다.

# 2. scr

### pointcloud_to_laserscan_node.cpp

