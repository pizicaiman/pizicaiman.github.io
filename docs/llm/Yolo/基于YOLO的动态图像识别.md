# 基于YOLO的动态图像识别（视频流目标检测，YOLOv5+OpenCV demo）

> 前提：已安装`torch`、`yolov5`和`opencv-python`

```python
import torch
import cv2

# 加载YOLOv5模型（如yolov5s）
model = torch.hub.load('ultralytics/yolov5', 'yolov5s', pretrained=True)

# 打开摄像头（或读取视频文件，可替换为 cv2.VideoCapture('your_video.mp4')）
cap = cv2.VideoCapture(0)  # 0表示默认摄像头

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    # YOLO模型推理（支持np.ndarray）
    results = model(frame)

    # 解析和绘制检测结果
    for *box, conf, cls in results.xyxy[0]:
        x1, y1, x2, y2 = map(int, box)
        label = results.names[int(cls)]
        score = float(conf)
        # 绘制目标框和类别标签
        cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
        cv2.putText(frame, f"{label} {score:.2f}", (x1, y1-10),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)

    # 实时显示
    cv2.imshow("YOLOv5 Video Detection", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

**说明：**
- 修改`cv2.VideoCapture()`参数可实现本地视频的检测。
- 按`q`退出实时画面。
- 可以方便移植到嵌入式设备/工业相机等场景。

