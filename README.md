# MORAI 시뮬레이션, Camera, Path_Making, PathPlanning

## 1. sub_camera.py
- Morai 시뮬레이션에서 발행된 CompressedImage 타입의 이지미 토픽을 구독하고, openCV 에서 사용 가능한 데이터 타입으로 변환

## 2. pub_camera.py
- 1 에서 변환된 이미지를 GrayImage로 변환

## 3. bird_eye_view.py
- 정면에서 바라본 이미지를 상단에서 바라본 이미지로 변환

## 4~6. line_detect.py, blend_line.py
- 흰색과 노란색에 대한 이미지 마스크를 만들고 합쳐서 원본 이미지에 마스크를 씌워 이미지에서 원하는 부분만 추출

## 7. binary_line.py
- 6 에서 추출된 이미지를 이진화 진행

## 8. sliding_window.py
- 7 에서 이진화된 배열에서 sliding window 알고리즘을 적용하여 차선 인식

## 9. LKAS.py
- 8 에서 인식한 차선을 기준으로 MORAI 시뮬레이션에서 사용할 cmd_vel 토픽 발행

## 10. path_maker.launch
- MORAI 시뮬레이션에서 GPS 센서를 이용하여 경로를 저장함.

## 11. planner.launch
- 10 에서 저장된 경로를 기반으로 경로를 추종하는 cmd_vel 토픽을 발행함.
