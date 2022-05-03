---
title: OpenCV常用代码段-Python
author: Zhang Ge
date: 2021-03-23 10:35:00 +0800
categories: [专业积累, 杂记]
tags: [python, opencv]
pin: true
---



> NOTE：opencv中的位置索引都是**先x方向后y方向**的，比如绘制函数时点的位置表示为（x, y）, resize中新尺寸大小指定格式为（width, height）等等。这个和numpy中先y方向后x方向不同，要注意区分。

### 图像读取与保存

参考：[GeeksforGeeks](https://www.geeksforgeeks.org/python-opencv-cv2-imread-method/), [CSDN](https://blog.csdn.net/weixin_43155243/article/details/100577772)

```python
import cv2

# 读取
img_path = ''
img = cv2.imread(img_path)  # BGR (default)

# 获取尺寸信息
height, width, channel = img.shape

# 保存
cv2.imwrite('save_name.png', img)
```

值得注意的是`cv2.imread(path, flag)`函数在一些场景下可能要指定`flag`，可选的`flag`有

- `cv2.IMREAD_COLOR`：当加载彩色图片时。这是默认值，等价的操作为直接传入`1`；

- `cv2.IMREAD_GRAYSCALE`：当加载灰度图时。等价的操作为直接传入`0`；

  > 读取灰度图若不指定`flag`，则读取的灰度图也会默认转化为3通道！

- `cv2.IMREAD_UNCHANGED`：按照原图的格式读取。等价的操作为直接传入`-1`. 比如要读取16位的`.tiff`数据时。

### 图像变形

[参考1](https://docs.opencv.org/master/da/d54/group__imgproc__transform.html#ga47a974309e9102f5f08231edc7e7529d) [参考2](https://www.geeksforgeeks.org/image-resizing-using-opencv-python/)

> 内插方式有三种：cv2.INTER_AREA, cv2.INTER_CUBIC和cv2.INTER_LINEAR(default)。 缩小图片时，内插方式一般使用cv2.INTER_AREA会最好；放大图片时，使用cv2.INTER_CUBIC会效果最好（但是慢），使用cv2.INTER_LINEAR会较快，并且看起来还OK.

**指定尺寸**

```
resized = cv2.resize(img, dsize=(width, height), interpolation)
```

**指定缩放比例**

```
half = cv2.resize(image, (0, 0), fx=0.5, fy=0.5)
```

### 图像绘制

[参考](https://blog.csdn.net/sinat_41104353/article/details/85171185)

#### 画框

```python
img = cv2.rectangle(img, (x1, y1), (x2, y2), color=(0, 0, 255), thickness=2)
# img = cv2.rectangle(img, (int(x1), int(y1)), (int(x2), int(y2)), color=(0, 0, 255), thickness=2)
# img = cv2.rectangle(img, (int(x), int(y)), (int(x+w), int(y+h)), color=(0, 0, 255), thickness=2)
```

#### 画圆

```python
img = cv2.circle(img, center, radius, color, thickness)
```

#### 画线

```python
img = cv2.line(img, start_point, end_point, color, thickness)
```

#### 写文字

```
img = cv2.putText(img, 'text', (50,150), cv2.FONT_HERSHEY_COMPLEX, 1, (0,0,255), 1)
```

各参数依次是：照片/添加的文字/左上角坐标/字体/字体大小/颜色/字体粗细



### 其他

#### BGR => RGB 

opencv读取的图片是BGR格式的，如果需要RGB格式（如numpy可视化）的话，要做一下转换

```python
# way1
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
# way2
image = image[:, :, ::-1]
```

#### opencv调用摄像头
[参考](https://blog.csdn.net/SilverBullet1997/article/details/103256854)

上demo:

```python
import cv2

# camera_addr = 0   # 本机摄像头
camera_addr = 'rtsp://username:password@ip'  # 网络摄像头

cap = cv2.VideoCapture(camera_addr)
assert cap.isOpened(), 'Failed to open %s' % camera_addr

w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fps = cap.get(cv2.CAP_PROP_FPS)
print('success (%gx%g at %.2f fps).' % (w, h, fps))

while True:
    ret, frame = cap.read()
    cv2.imshow('Video', frame)
    cv2.waitKey(1)
```

[这里](https://blog.csdn.net/qhd1994/article/details/80238707)有一个opencv的参数列表，获取一段视频的总帧数为`frames_num=cap.get(cv2.CAP_PROP_FRAME_COUNT)`
