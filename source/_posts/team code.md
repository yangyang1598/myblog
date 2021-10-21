-----
title: team프로젝트 코드
-----
### team project main code

- 차선 검출 및 고속도로 주행 내 객체 인식
- 개발 환경: YOLOV3, python ,opencv


+ 중요 함수
```python

def Yolo(img, score, nms):
    height, width = img.shape[:2]
    yolo_net = cv2.dnn.readNet('YOLO/yolov3.weights',
                               'YOLO/yolov3.cfg')
    layer_names = yolo_net.getLayerNames()
    output_layers = [layer_names[i[0] - 1] for i in
                     yolo_net.getUnconnectedOutLayers()]

    blob = cv2.dnn.blobFromImage(img, 0.00392, (416, 416), (0, 0, 0),
                                 True, crop=False)
    yolo_net.setInput(blob)
    outs = yolo_net.forward(output_layers)
    class_ids = []
    confidences = []
    boxes = []
    for out in outs:
        for detection in out:
            scores = detection[5:]
            class_id = np.argmax(scores)
            confidence = scores[class_id]
            # 검출 신뢰도
            if confidence > 0.5:
                # Object detected
                # 검출기의 경계상자 좌표는 0 ~ 1로 정규화되어있으므로 다시 전처리
                center_x = int(detection[0] * width)
                center_y = int(detection[1] * height)
                dw = int(detection[2] * width)
                dh = int(detection[3] * height)
                # Rectangle coordinate
                x = int(center_x - dw / 2)
                y = int(center_y - dh / 2)
                boxes.append([x, y, dw, dh])
                confidences.append(float(confidence))
                class_ids.append(class_id)
    indexes = cv2.dnn.NMSBoxes(boxes, confidences, score, nms)
    for i in range(len(boxes)):

        if i in indexes:
            x, y, w, h = boxes[i]
            label = str(classes[class_ids[i]])
            score = confidences[i]
            text = f'{label} {score:.2f}'
            # 경계상자와 클래스 정보 투영
            cv2.rectangle(img, (x, y), (x + w, y + h), (50, 50, 50), 2)
            cv2.rectangle(img, (x, y - 19), (int(x + len(text) * 2 * 4.5), y - 18 + len(text) * 2), (50, 50, 50), -1)
            cv2.putText(img, text, (x, y - 5), cv2.FONT_ITALIC, 0.5, (255, 255, 255), 2)
    return img
    pass

classes = ["person", "bicycle", "car", "motorcycle",
           "airplane", "bus", "train", "truck", "boat", "traffic light", "fire hydrant",
           "stop sign", "parking meter", "bench", "bird", "cat", "dog", "horse",
           "sheep", "cow", "elephant", "bear", "zebra", "giraffe", "backpack",
           "umbrella", "handbag", "tie", "suitcase", "frisbee", "skis",
           "snowboard", "sports ball", "kite", "baseball bat", "baseball glove", "skateboard",
           "surfboard", "tennis racket", "bottle", "wine glass", "cup", "fork", "knife",
           "spoon", "bowl", "banana", "apple", "sandwich", "orange", "broccoli", "carrot", "hot dog",
           "pizza", "donut", "cake", "chair", "couch", "potted plant", "bed", "dining table",
           "toilet", "tv", "laptop", "mouse", "remote", "keyboard",
           "cell phone", "microwave", "oven", "toaster", "sink", "refrigerator",
           "book", "clock", "vase", "scissors", "teddy bear", "hair drier", "toothbrush"]

```

- 차선 인식 오류
```python
    if len(left_lane_inds) == 0:
        for i in range(len(leftx_)):
            if len(leftx_[-(i + 1)]) != 0:
                leftx = leftx_[-(i + 1)]
                lefty = lefty_[-(i + 1)]
                print("왼쪽 에러")
                break
                pass
            pass
    if len(right_lane_inds) == 0:
        for i in range(len(rightx_)):
            if len(rightx_[-(i + 1)]) != 0:
                rightx = rightx_[-(i + 1)]
                righty = righty_[-(i + 1)]
                print("오른쪽 에러")
                break
                pass
            pass
        pass

 

```

- 차선 이탈 검출 및 출력
```python
    left_center = np.int_(np.mean(left_fitx))
    right_center = np.int_(np.mean(right_fitx))
    mid_center = np.int_(np.mean(mid_fitx))
    road_center = width // 2

    road_pixel = road_width / (right_center - left_center)
    error = mid_center - road_center
    error_pixel = error * road_pixel

    if error > 0:
        cv2.putText(img_result, f'right : {abs(error_pixel):.2f}m', (width // 2 - 50, height // 2 + 200), cv2.FONT_ITALIC, 0.5,
                    (0, 0, 255), 2)
    elif error < 0:
        cv2.putText(img_result, f'left : {abs(error_pixel):.2f}m', (width // 2 - 50, height // 2 + 200), cv2.FONT_ITALIC, 0.5,
                    (0, 0, 255), 2)
    elif error == 0:
        cv2.putText(img_result, f'center', (width // 2, height // 2), cv2.FONT_ITALIC, 0.5,
                    (0, 0, 255), 2)
    cv2.putText(img_result, f'{frame} / {frame_end}', (20, height - 40), cv2.FONT_ITALIC, 0.5,
                (255, 255, 255), 2)
    cv2.putText(img_result, f'{frame_count} / {frame_end * 3}', (20, height - 20), cv2.FONT_ITALIC, 0.5,
                (255, 255, 255), 2)

   
```
- 코드 알고리즘
![](img_3.png)

- 실행 영상

[차선인식 프로젝트 영상](https://youtu.be/BCT4i2_pSEI)

- 참고문헌 및 사이트


[1]  S.W. Jeon, D.S. Kim, “YOLO-based lane detection system”, Journal of the Korea Institute of Information and 
Communication Engineering, Vol. 25, No. 3, pp. 464-470, Mar. 2021.

[2] 큐 Queue, 「[Python3]OpenCV 곡선 차선 인식 프로젝트 - 차선 인식(1)」, 『큐의 Qrirosity Log』,
https://m.blog.naver.com/PostView.naver?blogId=hirit808&logNo=221486800161&proxyReferer=,  2021.10.02검색

[3] 봉식이 누나, 「Python으로 OpenCV를 사용하여 YOLO Object detection」,『봉식이와 캔따개』, 
https://bong-sik.tistory.com/m/16, 2021.10.08 검색

[4] 「 [분석] YOLO 」, 『 Hello Blog!』, https://curt-park.github.io/2017-03-26/yolo/  ,2021.10.13 검색
