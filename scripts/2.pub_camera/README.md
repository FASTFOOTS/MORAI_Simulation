# 2. pub_camera
이 코드는 ROS의 cv_bridge 를 이용하여 Morai 시뮬레이션의 camera 토픽을 openCV에서 취할 수 있는 이미지 변환(gray scale image & rgb_image)를 취하고 ros image 형태로 publish하는 코드이다. 

**1. init**
```python
    rospy.init_node('camera', anonymous=True)
    self.bridge=CvBridge()
```

- 노드 초기화 및 노드 이름 생성 
- image와 opencv type간 변환 수행 객체 bridge 생성 

**2. publisher**
```python
self.rgb_pub = rospy.Publisher('/camera_rgb_image', Image, queue_size=10)
self.gray_pub = rospy.Publisher('/camera_gray_img', Image, queue_size=10)    
```
- camera_rgb_image topic에 컬러 이미지 메시지를 발행하는 Publisher 객체 생성
- camera_gray_image topic에 흑백 이미지 메시지를 발행하는 Publisher 객체를 생성


**3. spin()**
```python
rospy.spin()
```

- rospy.spin()는 노드가 메시지나 이벤트를 기다리면서 실행을 계속하게 한다.


**4. rgb_to_Gray**
```python
gray_img = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)
```    

- cv2의 cvtColor 함수를 통해 color image를 gray image로 변환 후 gray_img에 저장

**5. image type translation**
```python
rgb_img_msg=self.bridge.cv2_to_imgmsg(img_bgr, 'bgr8')
gray_img_msg =self.bridge.cv2_to_imgmsg(gray_img)
```

- img_bgr & gray_img는 open_cv type이기에 이를 다시 ros_image type으로 변환 

**6. image publish**
```python
self.rgb_pub.publish(rgb_img_msg)
self.gray_pub.publish(gray_img_msg)
```
- ros image type의 gray image & rgb image를 각각의 publisher에서 발행 
