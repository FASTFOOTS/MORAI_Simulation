![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/67faeedf-345a-496d-a223-dd678f83fe99)

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/816e9499-c43e-45f5-8432-c6158e322364)


![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/dd747dd2-4d6f-4e43-a5b6-c6571a666fa8)

# 8.sliding_window
{
    각 함수의 결과를 한 줄로 정리하여 나타내도록. 

    flag에 대한 정보를 나타내야함 
}


**1. init**
```python
rospy.init_node("sliding_window_node")
```
- 노드를 초기화하고 "sliding_window_node" 이름의 노드를 생성

**2. publisher**
```python
self.pub = rospy.Publisher("/sliding_windows/compressed", CompressedImage, queue_size=10)
```
- "/sliding_windows/compressed" 이름의 topic에 Compressed message를 publish하는 publisher 생성

**3. subscriber**
```python
rospy.Subscriber("/image_jpeg/compressed", CompressedImage, self.img_CB)
```

- "/image_jpeg/compressed" 이름의 topic에 CompressedImage message를 subscribe하는 subscriber 생성 

**4. flag**
```python
self.nothing_flag = False
```

- flag 하나를 선언하고 이 초기값을 false로 설정 

**5. img_CB function**

```python
img = self.bridge.compressed_imgmsg_to_cv2(data)
```
- ros image data를 open_cv type으로 변환한 뒤 img 변수에 저장
- img: open_cv typ의 image가 저장된 변수

```python
self.nwindows = 10
```
- window의 크기를 10개로 설정한다. 

```python
self.window_height = np.int32(img.shape[0] / self.nwindows)
```
- window의 height를 설정하기 위해 image의 height 정보와 window의 개수 정보를 사용
- img는 (height, width, channel) 형태
- img.shape[0]은 height를 의미
- window_height: 한 window의 height 정보가 저장된 변수

```python
warp_img = self.img_warp(img)
```
- img_wrap() 함수의 입력으로 img를 넣고 함수의 결과를 warp_img에 저장

```python
blend_img = self.detect_color(warp_img)
```
- detect_color() 함수의 입력으로 warp_img를 넣고 함수의 결과를 blend_img에 저장 

```python
binary_img = self.img_binary(blend_img)
```
- img_binary() 함수의 입력으로 blend_img를 넣고 함수의 결과를 binary_img에 저장 

```python
if self.nothing_flag == False:
    self.detect_nothing()
    self.nothing_flag = True
```
- flag가 false인 경우 detect_nothing 함수를 수행
- flag를 True로 변환 

```python
(   sliding_window_img,
    left,
    right,
    center,
    left_x,
    left_y,
    right_x,
    right_y,
) = self.window_search(binary_img)
```
- window_search() 함수 입력으로 binary_img를 넣고 해당 결과의 값들을 해당 변수에 저장 

```python
sliding_window_msg = self.bridge.cv2_to_compressed_imgmsg(sliding_window_img)
```
- open_cv image를 ros image type으로 변환하고 이를 sliding_window_msg에 저장 

```python
self.pub.publish(sliding_window_msg)
```
- ros image type으로 변환된 liding_window_msg를 publish

```python
cv2.namedWindow("img", cv2.WINDOW_NORMAL)
```
- img라는 이름의 창 생성
- cv2.WINDOW_NORMAL 플래그를 사용하여 창의 크기를 조정 가능

```python
cv2.namedWindow("sliding_window_img", cv2.WINDOW_NORMAL)
```
- sliding_window_img라는 이름의 두 번째 창을 생성
- 앞선 코드와 현 코드에서 생성한 2개의 창은 cv2.imshow() 함수를 사용하여 이미지를 표시할 때 사용

```python
cv2.imshow("img", img)
cv2.imshow("sliding_window_img", sliding_window_img)
```
- 두 개의 image를 각각 해당하는 이름의 창에 출력


**6. detect nothing function**
```python
self.nothing_left_x_base = round(self.img_x * 0.140625)
```
- img_x는 img.shape[1]의 결과로 img의 width값을 지님
- img의 width 값에 0.140625의 값을 곱하는 연산 수행
- round() 함수를 사용하여 곱한 값을 반올림된 정수 값으로 표현
- 최종적인 계산 결과는 90이고 이를 nothing_left_x_base 변수에 저장
- nothing_left_x_base: 초기 주행 시 차선이 없는 경우, image의 90값에 임의의 차선 중 왼쪽 차선을 설정하기 위한 변수

```python
self.nothing_right_x_base = self.img_x - round(self.img_x * 0.140625)
```
- 최종적인 계산 결과는 550
- nothing_right_x_base: 임의의 차선 중 오른쪽 차선을 설정하기 위한 변수 

```python
self.nothing_pixel_left_x = np.zeros(self.nwindows) + round(640* 0.140625)
```
- np.zeros(self.nwindows)를 이용하여 원소가 0인 nwindow 크기의 배열 생성 
- round(640* 0.140625)의 결과를 배열에 더함
- 최종적인 결과 [90, 90, 90, 90, 90, 90, 90, 90, 90, 90]
- nothing_pixel_left_x: 위의 값을 지니는 배열을 저장하는 변수 

```python
self.nothing_pixel_right_x = (np.zeros(self.nwindows) + self.img_x - roun(self.img_x * 0.140625))
```
- 최종적인 결과 [550, 550, 550, 550, 550, 550, 550, 550, 550, 550]
- nothing_pixel_right_x: 위의 값을 지니는 배열을 저장하는 변수

```python
self.nothing_pixel_y = [round(self.window_height / 2) * index for index in range(0, self.nwindows)]
```
- window의 개수만큼 반복을 진행하며 window_height / 2 결과를 index와 곱하는 연산 수행
- 최종적인 결과 [0, 24, 48, 72, 96, 120, 144, 168, 192, 216]

**7. window_search**
```python
bottom_half_y = binary_line.shape[0] / 2
histogram = np.sum(binary_line[int(bottom_half_y) :, :], axis=0)
```
- bottom_half_y: binary_line img의 높이 절반에 해당하는 값을 저장하는 변수
- binary_line img를 half부터 마지막까지 height를 slicing, 나머지는 모두 사용
- histogram: slicing한 결과에 sum 연산 수행한 값을 지니는 변수  
- 해당하는 코드는 img의 y축 기준 아래를 사용하여 기존 pixel 분포를 구함. 

```python
midpoint = np.int32(histogram.shape[0] / 2)
left_x_base = np.argmax(histogram[:midpoint])
right_x_base = np.argmax(histogram[midpoint:]) + midpoint
```
- midpoint: histogram의 중앙 index을 저장하는 변수
- argmax()함수는 주어진 배열 내 최대값의 index를 반환하는 함수
- left_x_base: histogram 왼쪽 중 최대값의 index를 저장하는 변수
- right_x_base: histogram 오른쪽 중 최대값의 index를 저장하는 변수

```python
if left_x_base == 0:
    left_x_current = self.nothing_left_x_base
else:
    left_x_current = left_x_base

if right_x_base == midpoint:
    right_x_current = self.nothing_right_x_base
else:
    right_x_current = right_x_base
```
- histogram 중 max 값의 index가 0인 경우, 차선 인식을 못한 경우로 판단
- 임의의 차선 값인 nothing_left_x_base을 left_x_current 변수에 저장 
- 아닌 경우 histogram에서 구한 max값 index를 left_x_current 변수에 저장 
- 동일한 task를 오른쪽에도 취함. 
- left_x_current: 임의의 값 혹은 왼쪽 histogram max값 index 저장 변수
- right_x_current: 임의의 값 혹은 오른쪽 histogram max값 index 저장 변수

```python
out_img = np.dstack((binary_line, binary_line, binary_line)) * 255
```
- np.dstack() 함수는 주어진 배열들을 depth 방향으로 병합하여 다차원 배열을 생성
- (binary_line, binary_line, binary_line)는 세 개의 동일한 이진 이미지 배열을 의미하며 각 channel에 동일한 binary image를 줌
- 0과 1의 값을 지닌 binary image에 255를 곱해 rgb image의 값으로 맞춤. 
- out_img: rgb image의 값을 지닌 다차원 배열 정보를 지니는 변수 

```python
nwindows = self.nwindows   
window_height = self.window_height 
margin = 80
min_pix = round((margin * 2 * window_height) * 0.0031) # 24
```
- nwindows: window의 개수를 나타내는 변수
- window_height: window의 height를 나타내는 변수
- margin: window가 옆 차선까지 넘어가지 않도록 window의 너비를 지정 변수
- min_pixel: 차선으로 인식하기 위한 threshold를 나타내는 pixel 개수 지정 변수 

```python
    lane_pixel = binary_line.nonzero()
    lane_pixel_y = np.array(lane_pixel[0])
    lane_pixel_x = np.array(lane_pixel[1])
```
- nonzero()는 NumPy 배열에서 0이 아닌 요소 index 추출 함수 
- lane_pixel: binary_line에서 0이 아닌 값들을 나타내며, (row,column)의 형태
- lane_pixel_y: lane_pixel 중 row값들을 ndarray 형태로 저장하는 변수 
- lane_pixel_x: lane_pixel 중 column값들을 ndarray 형태로 저장하는 변수

```python
left_lane_idx = []
right_lane_idx = []
```
- left_lane_idx: 왼쪽 차선 pixel index를 담는 list 변수
- right_lane_idx: 오른쪽 차선 pixel index를 담는 list 변수 


**rest**
```python

        # Step through the windows one by one
        for window in range(nwindows):
            # window boundary를 지정합니다. (가로)
            win_y_low = binary_line.shape[0] - (window + 1) * window_height
            win_y_high = binary_line.shape[0] - window * window_height
            # print("check param : \n",window,win_y_low,win_y_high)

            # position 기준 window size
            win_x_left_low = left_x_current - margin
            win_x_left_high = left_x_current + margin
            win_x_right_low = right_x_current - margin
            win_x_right_high = right_x_current + margin

            # window 시각화입니다.
            if left_x_current != 0:
                cv2.rectangle(
                    out_img,
                    (win_x_left_low, win_y_low),
                    (win_x_left_high, win_y_high),
                    (0, 255, 0),
                    2,
                )
            if right_x_current != midpoint:
                cv2.rectangle(
                    out_img,
                    (win_x_right_low, win_y_low),
                    (win_x_right_high, win_y_high),
                    (0, 0, 255),
                    2,
                )

            # 왼쪽 오른쪽 각 차선 픽셀이 window안에 있는 경우 index를 저장합니다.
            good_left_idx = (
                (lane_pixel_y >= win_y_low)
                & (lane_pixel_y < win_y_high)
                & (lane_pixel_x >= win_x_left_low)
                & (lane_pixel_x < win_x_left_high)
            ).nonzero()[0]
            good_right_idx = (
                (lane_pixel_y >= win_y_low)
                & (lane_pixel_y < win_y_high)
                & (lane_pixel_x >= win_x_right_low)
                & (lane_pixel_x < win_x_right_high)
            ).nonzero()[0]

            # Append these indices to the lists
            left_lane_idx.append(good_left_idx)
            right_lane_idx.append(good_right_idx)

            # window내 설정한 pixel개수 이상이 탐지되면, 픽셀들의 x 좌표 평균으로 업데이트 합니다.
            if len(good_left_idx) > min_pix:
                left_x_current = np.int32(np.mean(lane_pixel_x[good_left_idx]))
            if len(good_right_idx) > min_pix:
                right_x_current = np.int32(np.mean(lane_pixel_x[good_right_idx]))

        # np.concatenate(array) => axis 0으로 차원 감소 시킵니다.(window개수로 감소)
        left_lane_idx = np.concatenate(left_lane_idx)
        right_lane_idx = np.concatenate(right_lane_idx)

        # window 별 좌우 도로 픽셀 좌표입니다.
        left_x = lane_pixel_x[left_lane_idx]
        left_y = lane_pixel_y[left_lane_idx]
        right_x = lane_pixel_x[right_lane_idx]
        right_y = lane_pixel_y[right_lane_idx]

        # 좌우 차선 별 2차 함수 계수를 추정합니다.
        if len(left_x) == 0 and len(right_x) == 0:
            left_x = self.nothing_pixel_left_x
            left_y = self.nothing_pixel_y
            right_x = self.nothing_pixel_right_x
            right_y = self.nothing_pixel_y
        else:
            if len(left_x) == 0:
                left_x = right_x - round(self.img_x / 2)
                left_y = right_y
            elif len(right_x) == 0:
                right_x = left_x + round(self.img_x / 2)
                right_y = left_y

        left_fit = np.polyfit(left_y, left_x, 2)
        right_fit = np.polyfit(right_y, right_x, 2)
        # 좌우 차선 별 추정할 y좌표입니다.
        plot_y = np.linspace(0, binary_line.shape[0] - 1, 100)
        # 좌우 차선 별 2차 곡선을 추정합니다.
        left_fit_x = left_fit[0] * plot_y**2 + left_fit[1] * plot_y + left_fit[2]
        right_fit_x = right_fit[0] * plot_y**2 + right_fit[1] * plot_y + right_fit[2]
        center_fit_x = (right_fit_x + left_fit_x) / 2

        # # window안의 lane을 black 처리합니다.
        # out_img[lane_pixel_y[left_lane_idx], lane_pixel_x[left_lane_idx]] = (0, 0, 0)
        # out_img[lane_pixel_y[right_lane_idx], lane_pixel_x[right_lane_idx]] = (0, 0, 0)

        # 양쪽 차선 및 중심 선 pixel 좌표(x,y)로 변환합니다.
        center = np.asarray(tuple(zip(center_fit_x, plot_y)), np.int32)
        right = np.asarray(tuple(zip(right_fit_x, plot_y)), np.int32)
        left = np.asarray(tuple(zip(left_fit_x, plot_y)), np.int32)

        cv2.polylines(out_img, [left], False, (0, 0, 255), thickness=5)
        cv2.polylines(out_img, [right], False, (0, 255, 0), thickness=5)
        sliding_window_img = out_img
        return sliding_window_img, left, right, center, left_x, left_y, right_x, right_y
```
