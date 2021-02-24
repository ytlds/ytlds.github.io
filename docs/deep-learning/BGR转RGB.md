```python
#coding=utf-8
#OpenCV读进来的图像,通道顺序为BGR， 而matplotlib的顺序为RGB，因此需要转换

import cv2
import numpy as np
from matplotlib import pyplot as plt

img = cv2.imread('./test1.jpg')
B, G, R = cv2.split(img)

#BGR转RGB，方法1
img_rgb1 = cv2.merge([R, G, B])

#BGR转RGB，方法2
img_rgb2 = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

#BGR转RGB，方法3
img_rgb3 = img[:,:,::-1]

plt.figure('BGR_RGB’)

#显示opencv读进来的img， 通道顺序BGR
plt.subplot(3,3,1), plt.imshow(img)
#显示B通道
plt.subplot(3,3,4), plt.imshow(B)
#显示B通道
plt.subplot(3,3,5), plt.imshow(G)
#显示B通道
plt.subplot(3,3,6), plt.imshow(R)
#显示将BGR转为RGB的图像，3种方法
plt.subplot(3,3,7), plt.imshow(img_rgb1)
plt.subplot(3,3,8), plt.imshow(img_rgb2)
plt.subplot(3,3,9), plt.imshow(img_rgb3)
plt.show()
```

