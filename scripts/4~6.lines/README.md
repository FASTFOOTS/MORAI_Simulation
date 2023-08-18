# 4~6. lines  
이 코드는 원하는 색의 영역을 찾는 mask를 사용하여 도로위의 차선만 찾는 코드이다.
- white line detect : 하얀색만 찾는 코드
- yellow line detect : 노란색만 찾는 코드
- blend line detect : 하얀색과 노란색만 찾는 코드

**0. RGB 색공간과 HSV 색공간**

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108729047/e8a3c4ef-a50a-4b13-9bed-06075ae70c88)

- RGB는 색상으로 이미지를 표현
- HSV는 H(Hue; 색조), S(Saturation; 채도), V(Value; 명도)로 이미지를 표현

- RGB 이미지에서 색 정보를 검출하기 위해서는 R, G, B 세가지 속성을 모두 참고해야함.
- HSV 이미지에서는 H(Hue)가 일정한 범위를 갖는 순수한 색 정보를 가지고 있기 때문에 RGB 이미지보다 쉽게 색 분류 가능
- 특정 색 하나를 가져오는게 아니라 두 색 사이의 범위를 구해야하기 때문에 hsv가 유리



**1. 색공간 변경**
```python
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
```
- cv2.cvtColor함수를 이용하여 img의 색공간을 변경해서 hsv라는 배열 생성
- RGB(cv2는 BGR)의 색공간을 HSV 색공간으로 변경
- img[행 개수 X 열 개수 X 3(RGB)] -> hsv[행 개수 X 열 개수 X 3(HSV)]
  
**2. 색상 범위 지정**
```python
white_lower = np.array([0, 0, 150])
white_upper = np.array([255, 10, 255])

yellow_lower = np.array([20, 100, 100])
yellow_upper = np.array([30, 255, 255])
```
- mask를 만들기 위해 원하는 색의 lower, upper 값 지정
- white_lower (h: 0, s:0, v:150)
- white_upper (h: 255, s: 10, v: 255)
- yellow_lower (h: 20, s:100, v: 100)
- yellow_upper (h: 30, s: 255, v: 255)
- 범위를 지정을 제대로 못할 경우, 마스크에 노이즈가 많이 생김

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108729047/671b1870-a5aa-4f3c-9f67-30eb0483aa99)
- 왼쪽
  - yellow_lower = np.array([20, 100, 100])
    yellow_upper = np.array([30, 255, 255])
- 오른쪽
  - yellow_lower = np.array([15, 80, 0])
    yellow_upper = np.array([45, 255, 255])

**3. mask 생성하기**
```python
white_mask = cv2.inRange(hsv, white_lower, white_upper)

yellow_mask = cv2.inRange(hsv, yellow_lower, yellow_upper)

blend_mask = cv2.bitwise_or(yellow_mask, white_mask)
```
- mask는 hsv 배열에서 원하는 색이 위치한 곳을 찾는 배열
- cv2.inRange 함수를 사용해서 mask 생성
- lower와 upper 사이에 해당하는 값들은 255, 해당하지 않는 값들은 0으로 설정
- blend_mask는 white_mask와 yellow_mask를 cv2.bitwise_or연산하여 둘 중 하나라도 만족하는 곳을 255로 설정

**4. 이미지와 mask 합치기**
```python
white_color = cv2.bitwise_and(img, img, mask=white_mask)

yellow_color = cv2.bitwise_and(img, img, mask=yellow_mask)

blend_color = cv2.bitwise_and(img, img, mask=blend_mask)

```
- cv2.bitwise_and연산을 이용하여 img에 mask 합치기
- @@@_color는 원본이미지에 @@@_mask를 합친 이미지라고 생각해도 무방
- img 중에 mask가 있는 부분은 원본 색상으로 출력, mask가 없는 부분은 검은색으로 출력

**5. imshow로 출력**
```python
cv2.imshow("white_mask", white_mask)
cv2.imshow("white_color", white_color)

cv2.imshow("yellow_mask", yellow_mask)
cv2.imshow("yellow_color", yellow_color)

cv2.imshow("blend_mask", blend_mask)
cv2.imshow("blend_color", blend_color)

```
- cv2.imshow를 사용해서 mask와 color(원본 사진에 mask를 합친 사진) 출력
- mask와 color는 원하는 영역을 하얀색으로 표시할 것인가, 원본 색상으로 출력할 것인가 차이
![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108729047/9272faf4-70b8-431e-977b-2546ac37d486)
- 왼쪽
  - yellow_color
- 오른쪽
  - yellow_mask
