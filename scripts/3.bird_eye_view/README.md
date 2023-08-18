# 3. bird_eye_vies
이 코드는 차선인식을 위해 다른 배경(로봇 옆 배경이나 하늘 등) 없이 차선만 보도록 화면을 변환하는 코드이다.  
하늘에 떠 있는 새가 아래를 바라보는 것처럼 화면이 변환된다.  


**1. 기준점 입력**
```python
img_size = [640, 480]
src_center_offset = [200, 315]
```
- 죄측 차선위에 있는 점 하나를 선택하여 해당 점을 바탕으로 사다리꼴을 그려 이미지 변환
- 기존 이미지 사이즈 640x480 입력
- 200,315는 죄측 차선 위의 한 점(기준점으로 사용)

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/bb73a083-82fc-4459-9257-763a7a8e4b38)

  
**2. 사다리꼴 영역 설정**
```python
src = np.float32([
                [0, 479],
                [src_center_offset[0], src_center_offset[1]],
                [640 - src_center_offset[0], src_center_offset[1]],
                [639, 479],
            ])
```
- src에 사다리꼴을 그릴 4개의 점의 좌표 지정하기
- 0, 479 : 죄측 차선 하단 점
- src_center_offset[0], src_center_offset[1] : 1번에서 선택한 기준점
- 640 - src_center_offset[0], src_center_offset[1] : 기준점을 뒤집은 우측 차선 위의 기준점
- 639, 479 : 우측 차선 하단 점

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/1ef564c1-d0be-4c07-b72b-c82efe9ce269)


**3. dst offset 설정**
```python
dst_offset = [round(self.img_x * 0.125), 0]
```
- 사다리꼴을 직사각형으로 바꾸기 위해 offset 설정
- 좌측 상단 중 x좌표가 이미지의 0.125(1/8)가 되는 지점에 기준점을 위치시키기
  
**4. dst 영역 설정**
```python
dst = np.float32([
                [dst_offset[0], self.img_y],
                [dst_offset[0], 0],
                [self.img_x - dst_offset[0], 0],
                [self.img_x - dst_offset[0], self.img_y],
            ])
```
- dst에 dst_offset을 바탕으로 변경시킬 직사각형 영역의 좌표 지정하기
- 차례로 좌측 하단, 좌측 상단, 우측 상단, 우측 하단 점

![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/591efabd-f099-4b18-a161-26b69892be9f)




**5. 사다리꼴 영역을 dst 영역으로 투시 변환**
```python
matrix = cv2.getPerspectiveTransform(src, dst)
warp_img = cv2.warpPerspective(img, matrix, (self.img_x, self.img_y))
```
- cv2.getPerspectiveTransform을 사용하여 src영역을 dst로 바꾸기
- src에서 정한 4개의 점과 dst에서 정한 4개의 점을 바탕으로 영상 투시변환하는 원근 맵 행렬 생성
![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/108729047/365cd64c-07e0-4c62-93b2-ad8db2ff715e)
- 해당 사진을 참고하면 이해하기 편리
   
- cv2.warpPerspective를 사용해서 원본 이미지에 matrix(투시변환한 원근 맵 행렬)를 적용
 
- #### (사다리꼴 그린 사진과 변경된 사진 점 위치 표시해서 서로 비교하는 사진 )
