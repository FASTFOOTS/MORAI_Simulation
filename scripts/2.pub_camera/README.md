# 2. sub_camera
이 코드는 ROS의 cv_bridge 를 이용하여 Morai 시뮬레이션의 camera 토픽을 openCV에서 취할 수 있는 이미지 변환(gray scale image & rgb_image)를 취하고 ros image 형태로 publish하는 코드이다. 

**1. init**
```python
def __init__(self):
    rospy.init_node('camera', anonymous=True)
    self.bridge=CvBridge()
    self.image_sub = rospy.Subscriber("/image_jpeg/compressed",CompressedImage, self.callback)
    self.rgb_pub = rospy.Publisher('/camera_rgb_image', Image, queue_size=10)
    self.gray_pub = rospy.Publisher('/camera_gray_img', Image, queue_size=10)    
    rospy.spin()
```
- 노드 초기화 및 노드 이름 생성 
- image와 opencv type간 변환 수행 객체 생성 
- image_jpeg/compressed topic에서 CompressedImage message를 구독하는 Subscriber 생성
- Subscriber에서 message 구독 시 callback 함수 호출
- camera_rgb_image topic에 컬러 이미지 메시지를 발행하는 Publisher 객체 생성
- camera_gray_image topic에 흑백 이미지 메시지를 발행하는 Publisher 객체를 생성

**2. Callback**
```python
def callback(self, data):
    img_bgr = self.bridge.compressed_imgmsg_to_cv2(data)
    
    gray_img = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)
    rgb_img_msg=self.bridge.cv2_to_imgmsg(img_bgr, 'bgr8')
    gray_img_msg =self.bridge.cv2_to_imgmsg(gray_img)
    
    self.rgb_pub.publish(rgb_img_msg)
    self.gray_pub.publish(gray_img_msg)
```
- color image를 gray image로 변환 후 gray_img에 저장
- img_bgr & gray_img는 open_cv type이기에 이를 다시 ros의 image type으로 변환 
- 변환된 형태를 각각 _msg 이름의 변수 저장 및 각각을 publish
