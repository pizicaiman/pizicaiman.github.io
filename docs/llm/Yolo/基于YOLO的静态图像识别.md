# 基于YOLO的静态图像识别（推理）示例（以PyTorch YOLOv5为例）

> 前提：已安装 `torch` 和 `yolov5`，并下载好权重文件（如`yolov5s.pt`）

```python
import torch
from PIL import Image
import matplotlib.pyplot as plt

# 加载yolov5模型（例如yolov5s）
model = torch.hub.load('ultralytics/yolov5', 'yolov5s', pretrained=True)

# 指定待检测的静态图片路径
img_path = 'your_image.jpg'

# 加载图片
img = Image.open(img_path)

# 进行推理
results = model(img)

# 打印结果
results.print()      # 显示终端检测结果（如类别、置信度、坐标等）
results.show()       # 自动弹窗展示识别后的图片
# results.save()     # 保存结果图片到runs/detect/目录下

# 也可提取预测框信息
for *box, conf, cls in results.xyxy[0]:
    x1, y1, x2, y2 = [int(v) for v in box]   # 坐标
    score = float(conf)                      # 置信度
    label = int(cls)                         # 类别索引
    print(f'类别: {results.names[label]}, 位置:({x1},{y1},{x2},{y2}), 置信度:{score:.2f}')
```

**说明：**
- 可更换模型为`yolov5m/l/x`或自训练权重`custom.pt`
- YOLO系列可用于目标检测，也支持图片URL、目录批量检测等
- 结果输出包括检测类别、置信度、边界框等信息
