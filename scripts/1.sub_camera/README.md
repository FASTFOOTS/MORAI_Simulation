# 1. sub_camera
이 코드는 ROS의 cv_bridge 를 이용하여 Morai 시뮬레이션의 camera 토픽을 openCV에서 사용 가능한 이미지 데이터 타입으로 변환하고 출력하는 코드이다.

**1. init**
```python
rospy.init_node('camera', anonymous=True)
```
- 노드 초기화
- camera 노드 생성  
  
  
**2. Subscriber**
```python
self.image_sub = rospy.Subscriber("/image_jpeg/compressed", CompressedImage, self.callback)
```
- Morai 시뮬레이션이 발행한 토픽 구독
- 토픽의 타입은 ROS의 CompressedImage
  
  
**3. 이미지 변환**
```python
comp_img = self.bridge.compressed_imgmsg_to_cv2(data)
```
- ROS의 이미지 데이터 타입을 openCV에서 사용 가능한 데이터 타입으로 변환
- cv_bridge 패키지의 CvBridge 함수 사용

  
**4. 이미지 출력**
```python
cv2.imshow("Image window", comp_img)
```
- 변환된 comp_img 모니터에 출력
- cv2.imshow 함수 사용

  
**5. 창 닫기**
```python
cv2.waitKey(1)
...
except rospy.ROSInterruptException:
        pass
```
- 0.001초만큼 키보드가 입력되면 인터럽트가 발생
- 코드 실행 종료






