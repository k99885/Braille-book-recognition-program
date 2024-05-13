# Braille Translation Program for Beginners
시각 장애인들이 점자책을 이용하는데 여러 가지 불편함이 존재할 것 같아 점자책을 영상처리 기술을 사용하여 인식하고 이를 기반으로 음성파일로 변환 시켜주는 알고리즘을 개발하였습니다.

## 1. 영상 취득
![9_low](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/8c15d613-2ba9-4703-a8d2-60fd70057967)

점자책을 직접 촬영하여 얻은 영상입니다. 책을 카메라로 찍었을때 대상이 평평하지않고 불균일 하기때문에 이러한 점을 고려하여 진행 하였습니다.

![10_hi](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/c7b5573b-ae7a-403f-9ec9-2395d5ddd73c)

또한 실사용을 고려하여 회전된 이미지도 사용하였습니다.

## 2. 영상 처리
![점자 규격](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/77ab501a-2c68-4a74-9b58-35117cc69ff8)

점자는 일정한 규격을 가지므로 일정하게 정렬시켜 주어야 합니다.

### 2.1 특징점 검출

```
params = cv2.SimpleBlobDetector_Params()

params.minThreshold = 10
params.maxThreshold = 240
params.thresholdStep = 1

params.minDistBetweenBlobs=5

params.filterByArea = True
params.minArea = 2
params.maxArea = 6.5

params.filterByColor = True
params.blobColor=255

params.filterByInertia = True
params.minInertiaRatio = 0.1

params.filterByCircularity = True
params.minCircularity = 0.4

params.filterByConvexity = True
params.minConvexity = 0.7

detector = cv2.SimpleBlobDetector_create(params)
keypoints = detector.detect(gray)
```

opencv의 SimpleBlobDetector_Params() 함수를 사용하여 blob 특징점을 검출하여 keypoint로 나타내는 작업을 진행하였습니다.

```
img = cv2.drawKeypoints(image, keypoints,None, (0,0,255),cv2.DRAW_MATCHES_FLAGS_DEFAULT)
```

drawKeypoints() 를 사용하여 검출한 keypoint들을 시각화 하였습니다.

![책 BLOB](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/6c59d503-264f-45b9-a6e5-bef59893366b)

### 2.2 특징점 리스트화

```
    contours,_=cv2.findContours(img2, mode=cv2.RETR_EXTERNAL, method=cv2.CHAIN_APPROX_SIMPLE)

    for i, cnt in enumerate(contours):

        x,y,width_fst,height_fst = cv2.boundingRect(cnt)

        list1_1[i]=i,x+40,y+20
```

2.1 에서 검출한 blob로 원을 그려 새롭게 나타낸 이미지에서 cv2.findContours()을 사용하여 원의 특징점을 저장하고

cv2.boundingRect() 함수를 사용하여 원의 시작 좌표들을 추출하여 리스트에 저장하였습니다.

```
    for i in range(len(list1_1)):
        x = list1_1[i][1]
        y = list1_1[i][2]
        img2_2[y, x] = 255
```

list1_1 (리스트)에 저장한 점자들의 좌표로 새로운 이미지(img2_2)에 나타내었습니다.

![img2_2](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/e2ce3c98-b613-4491-8fb6-3089587d76cf)

이때 이미지를 임의로 회전시켜서 회전된 이미지에 대한 보정도 진행 하였습니다.

### 2.3 이미지 회전보정

```
    lines_z = cv2.HoughLines(img2_2, rho_z, theta_z, 10)
    img3_2_4,_,angle = draw_houghLines_1(img3_2_4, lines_z, 0, 3.14)
    average = sum(angle) / len(angle)


```
![img3_2_4](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/6104a72e-c74a-4588-b4c6-a8000da347b1)

img2_2를  cv2.HoughLines()의 허프변환 함수를 사용하여 일정 thres 개수를 넘어가는 line들을 저장해주고 이들의 평균 각도를 구하였습니다.

```
hei_1, wid_1 = img3_2_4.shape[:2]
rotation_center = (wid_1 // 2, hei_1 // 2)
rotation_angle = math.degrees(average)

# 회전 매트릭스 계산
rotation_matrix = cv2.getRotationMatrix2D(rotation_center, -(90-rotation_angle), 1.0)

# 이미지 회전
img3_2_3 = cv2.warpAffine(img3_2_3, rotation_matrix, (wid_1, hei_1))
```
회전된 평균 각도를 이용하여 cv2.getRotationMatrix2D() 함수로 회전 매트릭스를 구하고  cv2.warpAffine() 매스릭스를 이용한 어파인 변환으로 이미지를 회전시켜주었습니다.

![img3_2_3](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/b329ffc1-f4ea-4ef3-b6f3-2100d57df6dd)

img3_2_3 또한 이전의 방법과 같이  리스트(list1)에 점자들의 좌표를 저장했습니다.

### 2.4 불균인한 영상 보정 

이전 단계(2.3) 에서 얻은 img3_1은 이미지의 휘어짐과 왜곡이 존재하기 떄문에 일정 각도,길이 이내의 좌표들을 연결하보았습니다.

![img3_1](https://github.com/k99885/Braille-Translation-Program-for-Beginners/assets/157681578/483b48de-9048-471c-a14e-c35e7b141441)

책의 가운데 부분(영상의 왼쪽)은 두께로인한 휘어짐이 발생하므로 이부분에 대한 보정이 필요하였고

또한 점자는 일정한 규격과 규칙을 가지고 있기 떄문에 전체적으로 일정한 규격으로 만들어주어야합니다.


```
        img3_3_1 = img3_1[0:heigt_total, div6 * 0:div6 * 1]
        img3_3_2 = img3_1[0:heigt_total, div6 * 1:div6 * 2]
        img3_3_3 = img3_1[0:heigt_total, div6 * 2:div6 * 3]
        img3_3_4 = img3_1[0:heigt_total, div6 * 3:div6 * 4]
        img3_3_5 = img3_1[0:heigt_total, div6 * 4:div6 * 5]
        img3_3_6 = img3_1[0:heigt_total, div6 * 5:div6 * 6]
        img3_3_7 = img3_1[0:heigt_total, div6 * 6:div6 * 7]
```

우선 이미지를 여러개의 조각으로 나눠주었고


```
