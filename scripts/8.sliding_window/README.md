![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/67faeedf-345a-496d-a223-dd678f83fe99)

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/816e9499-c43e-45f5-8432-c6158e322364)


![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/dd747dd2-4d6f-4e43-a5b6-c6571a666fa8)

# 8.sliding_window
binary화 된 차선 이미지에 window를 적용하여 차선을 detection하고 차선에 line을 그리며 tracking을 하는 코드이다. 

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
- 이 flag는 차선 인식 여부를 나타내는 flag이다. 

**5. img_CB function**

```python
img = self.bridge.compressed_imgmsg_to_cv2(data)
```
- ros image data를 open_cv type으로 변환한 뒤 img 변수에 저장
- img: open_cv type의 image가 저장된 변수

```python
self.nwindows = 10
```
- window의 크기를 10개로 설정한다. 

```python
self.window_height = np.int32(img.shape[0] / self.nwindows)
```
- window의 height를 설정하기 위해 image의 height 정보와 window의 개수 정보를 사용
- img는 (height, width, channel) 형태로 구성
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
- lane_pixel_y: lane_pixel 중 row값들을 ndarray 형태로 저장하는 변수 -> 차선이라 판단한 pixel
- lane_pixel_x: lane_pixel 중 column값들을 ndarray 형태로 저장하는 변수 -> 차선이라 판단한 pixel 

```python
left_lane_idx = []
right_lane_idx = []
```
- left_lane_idx: 왼쪽 차선 pixel index를 담는 list 변수
- right_lane_idx: 오른쪽 차선 pixel index를 담는 list 변수 

### 반복문 내 코드 설명 

```python
for window in range(nwindows):
```
- window의 수만큼 반복을 진행
- 반복문을 통해 각 window에 해당하는 정보 표현 

```python
win_y_low = binary_line.shape[0] - (window + 1) * window_height
win_y_high = binary_line.shape[0] - window * window_height
win_x_left_low = left_x_current - margin
win_x_left_high = left_x_current + margin
win_x_right_low = right_x_current - margin
win_x_right_high = right_x_current + margi
```
- y축 좌표는 위에서 아래로 가는 방향으로 좌표가 상승하는 특징을 지님
- win_y_low: binary_line의 height 정보를 이용하여 window의 각 y축 기준 하단의 좌표를 설정 변수
- win_y_high: binary_line의 height 정보를 이용하여 window의 각 y축 기준 상단의 좌표를 설정 변수
- win_x_left_low: 구한 차선의 왼쪽 좌표에 margin 값을 빼 좌측 window의 x축 기준 왼쪽 좌표 설정 변수
- win_x_left_high: 구한 차선의 왼쪽 좌표에 margin 값을 빼 좌측 window의 x축 기준 오른쪽 좌표 설정 변수
- win_x_right_low: 구한 차선의 오른쪽 좌표에 margin 값을 더해 우측 window의 x축 기준 왼쪽 좌표 설정 변수 
- win_x_right_high: 구한 차선의 오른쪽 좌표에 margin 값을 더해 우측 window의 x축 기준 오른쪽 좌표 설정 변수  

```pytorch
if left_x_current != 0:
    cv2.rectangle(
        out_img,
        (win_x_left_low, win_y_low),
        (win_x_left_high, win_y_high),
        (0, 255, 0),
        2)
if right_x_current != midpoint:
    cv2.rectangle(
        out_img,
        (win_x_right_low, win_y_low),
        (win_x_right_high, win_y_high),
        (0, 0, 255),
        2)
```
- cv2의 직사각형 모양을 그리는 rectangle을 이용하여 window를 그림
- out_img에 그림을 그리며 사각형의 좌표를 구한 좌표를 사용하여 나타냄
- (0, 0, 255)를 이용하여 window의 색상을 결정
- 2는 window를 그리는 선의 두께를 설정

```python
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
```      
- 차선 픽셀의 값이 설정된 window 내 범위에 들어오는 pixel중 0이 아닌 요소들의 index를 반환
- [0]을 통해 해당 범위 내 pixel들의 row에 대한 값들을 나타내게 된다. 
- good_left_idx: 왼쪽 차선 pixel에 대한 index 저장 변수
- good_right_idx: 오른쪽 차선 pixel에 대한 index 저장 변수

```python
left_lane_idx.append(good_left_idx)
right_lane_idx.append(good_right_idx
```
- 결정된 차선 pixel들의 index를 각 list에 추가

```python
if len(good_left_idx) > min_pix:
    left_x_current = np.int32(np.mean(lane_pixel_x[good_left_idx]))
if len(good_right_idx) > min_pix:
    right_x_current = np.int32(np.mean(lane_pixel_x[good_right_idx])
```
- 왼쪽 차선에 포함된 pixel들의 수가 차선 판단 최소 개수보다 많은 경우 left_x_current를 해당 pixel들 x좌표 평균값으로 대체
- 오른쪽 차선에 포함된 pixel들의 수가 차선 판단 최소 개수보다 많은 경우 right_x_current를 해당 pixel들 x좌표 평균값으로 대체

```python
left_lane_idx = np.concatenate(left_lane_idx)
right_lane_idx = np.concatenate(right_lane_idx)
```
- np.concatenate() 함수는 주어진 배열들을 연결하여 하나의 배열로 합치는 함수
- index들이 저장된 lef_lane_idx & right_lane_idx의 차원을 하나 줄인다. 

```python
left_x = lane_pixel_x[left_lane_idx]
left_y = lane_pixel_y[left_lane_idx]
right_x = lane_pixel_x[right_lane_idx]
right_y = lane_pixel_y[right_lane_idx]
```
- 차선 pixel의 해당 index값을 사용하여 차선의 pixel값 좌표를 각 변수에 저장 
- left_x: 왼쪽 차선의 x좌표를 나타내는 변수
- left_y: 왼쪽 차선의 y좌표를 나타내는 변수
- right_x: 오른쪽 차선의 x좌표를 나타내는 변수
- right_y: 오른쪽 차선의 y좌표를 나타내는 변수

```python
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
```
- 왼쪽 차선과 오른쪽 차선의 x,y의 값이 없다고 판단하는 경우 임의의 값을 사용
- 두 차선 중 한쪽만 인식이 된 경우, 인식된 차선을 기준으로 반대쪽 차선을 img_x/2(240픽셀) 만큼 좌측 혹은 우측에 생성

```python
left_fit = np.polyfit(left_y, left_x, 2)
right_fit = np.polyfit(right_y, right_x, 2)
```   
- np.polyfit() 함수를 사용하여 이 점들에 가장 잘 맞는 2차 다항식을 추정
- 각 변수에는 추정된 2차 다항식의 계수들을 저장      
    
```python
plot_y = np.linspace(0, binary_line.shape[0] - 1, 100)
```
- np.linspace()는 주어진 범위 0부터 binary_line의 height - 1 범위 값을 일정한 간격으로 나누어 100개의 숫자를 생성

```python
left_fit_x = left_fit[0] * plot_y**2 + left_fit[1] * plot_y + left_fit[2]
right_fit_x = right_fit[0] * plot_y**2 + right_fit[1] * plot_y + right_fit[2]
center_fit_x = (right_fit_x + left_fit_x) / 2
```
- 2차 다항식을 사용하여 좌측 차선 위 곡선을 표현하는 좌표를 표현 
- left_fit_x: 좌측 차선 위 곡선 x좌표를 나타내는 변수
- right_fit_x: 우측 차선 위 곡선 x좌표를 나타내는 변수 
- center_fit_x: 둘의 중앙값을 나타내는 변수


<p align="left"><img width="550" src="https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/6cb7e673-13b0-4cd5-9866-843c399d9078.png"><p>



```python
center = np.asarray(tuple(zip(center_fit_x, plot_y)), np.int32)
right = np.asarray(tuple(zip(right_fit_x, plot_y)), np.int32)
left = np.asarray(tuple(zip(left_fit_x, plot_y)), np.int32)
```
- zip() 함수는 literable 객체를 쌍으로 묶어주는 역할 
- 곡선 위의 한 점의 쌍을 zip으로 묶고 이를 tuple로 묶음
- np.asarray(..., np.int32): 생성한 tuple을 data type으로 np.int32를 사용하여 NumPy 배열로 변환
- 최종적으로 곡선 위의 점들을 좌표 형태로 표현

```pytorch
cv2.polylines(out_img, [left], False, (0, 0, 255), thickness=5)
cv2.polylines(out_img, [right], False, (0, 255, 0), thickness=5)
```
- cv2.polylines() 함수는 OpenCV 라이브러리를 사용하여 이미지 위에 다각선을 그리는 함수
- 우측, 좌측에 대한 차선을 나타내기 위해 out_img에 곡선을 그림
- false는 다각선의 시작점과 끝점을 열린 형태로 두어 연결되지 않도록 설정
- 이하의 인자들은 색과 두께를 설정

```python
return sliding_window_img, left, right, center, left_x, left_y, right_x, right_y
```
- 최종적으로 다음 변수들을 반환
- sliding_window_img: 차선과 window가 그려진 image를 담는 변수
- left, right, center: 각각 차선에 대한 좌표 쌍 변수
- left_x, left_y, right_x, right_y: 각각 차선에 대한 좌표 변수

![Untitled](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/4af240e1-f886-46d5-9298-14d1b8a36b0c)
