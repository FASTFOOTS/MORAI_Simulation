# 9.LKAS (Lane Keeping Assistant System)

이 코드는 8. Sliding window에서 구한 양쪽의 차선을 통해 로봇의 속도 토픽인 cmd_vel을 publish(발행)하는 코드이다.

**1. 로봇 구동을 위한 cmd_vel 계산**
```python
def img_CB(self, data):
        img = self.bridge.compressed_imgmsg_to_cv2(data)
        self.nwindows = 10
        self.window_height = np.int32(img.shape[0] / self.nwindows)
        warp_img = self.img_warp(img)
        blend_img = self.detect_color(warp_img)
        binary_img = self.img_binary(blend_img)
        if self.nothing_flag == False:
            self.detect_nothing()
            self.nothing_flag = True
        (
            sliding_window_img,
            left,
            right,
            center,
            left_x,
            left_y,
            right_x,
            right_y,
        ) = self.window_search(binary_img)

        power = [(center[0,0]-self.img_x//2)*0.8,
                 (center[1,0]-self.img_x//2)*1.0,
                 (center[2,0]-self.img_x//2)*0.5,
                 (center[3,0]-self.img_x//2)*0.2,
                 (center[4,0]-self.img_x//2)*0.1]
        
        center_power = (power[0]+power[1]+power[2]+power[3]+power[4])/600
        
        self.cmd.linear.x = 1.0
        self.cmd.angular.z =-center_power

        self.pub.publish(self.cmd)
```
- center 배열은 중앙 차선 윈도우의 좌표를 가지고 있다.
- center 배열과 self.img_x//2(이미지의 중심) 의 차이에 가중치를 곱해 power 배열을 만든다.
- power 배열의 요소들을 각각 더해 cmd_vel 토픽의 angular.z 값으로 입력한다.
- 계산된 cmd_vel 토픽을 발행하여 Morai 시뮬레이션의 로봇을 동작시킨다.
