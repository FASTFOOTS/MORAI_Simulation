# 7.binary_line

이 코드는 6.blend_line에서 원하는 차선만 출력한 후 해당 이미지를 이진화(0,1)하는 코드이다.

**1. img_binary**
```python
def img_binary(self, blend_line):
    bin = cv2.cvtColor(blend_line, cv2.COLOR_BGR2GRAY)
    binary_line = np.zeros_like(bin)
    binary_line[bin >127] = 255 # 1
    return binary_line
...

blend_img = self.detect_color(warp_img)
binary_img = self.img_binary(blend_img)
```
- cv2.cvtColor를 사용하여 원하는 색깔의 이미지만 뽑아낸 데이터를 GrayScale로 변환하여 bin에 저장한다.
- np.zeros_like 함수를 이용하여 0으로 채워진 bin과 같은 크기의 binary_line배열을 생성한다.
- bin 배열 중에서 값이 127보다 큰 위치에 해당하는 인덱스를 찾아 binary_line 배열에 1을 넣어 이진화를 진행한다. (이진화 배열의 시각화를 위해 255를 넣었다)


![image](https://github.com/FASTFOOTS/MORAI_Simulation/assets/80691076/7d589279-99c0-42d1-8f44-446e11b58b8e)
