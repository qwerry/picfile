# (33条消息) 【OpenCV】 OpenCV图像匹配识别滑动验证码缺口_opencv 滑块验证码_Koma_zhe的博客-CSDN博客
**目标**：识别出图片目标缺口的位置，输入一张带有缺口的验证码图片，输出缺口的位置（一般为缺口左侧横坐标）。  
**输入**：  
![](https://img-blog.csdnimg.cn/f737538ab36045d88c5b5e3e99bf1bf4.png#pic_center)
  
**输出**：  
![](https://img-blog.csdnimg.cn/815c5c6dc4aa4c948d6d60dcc876570e.png#pic_center)
  
利用 [OpenCV](https://so.csdn.net/so/search?q=OpenCV&spm=1001.2101.3001.7020) 进行基本的图像处理来实现的，主要步骤包括：

*   对验证码图片进行高斯模糊滤波处理，消除部分噪声干扰。
*   对验证码图片应用边缘检测算法，通过调整相应阈值识别出滑块边缘。
*   对上一步得到的各个边缘轮廓信息，通过对比面积、位置、周长等特征筛选出最可能的轮廓位置，得到缺口位置。

**一、高斯滤波**：  
OpenCV 提供了一个用于实现高斯模糊的方法，叫做 GaussianBlur，经过高斯滤波处理后，图像会变得模糊

```python
def GaussianBlur(src, ksize, sigmaX, dst=None, sigmaY=None, borderType=None)






```

**二、边缘检测**：  
边缘检测应用比较广泛的边缘检测算法是 Canny边缘检测算法，经过边缘检测算法处理后，一些比较明显的边缘信息会被保留下来

```python
def Canny(image, threshold1, threshold2, edges=None, apertureSize=None, L2gradient=None)






```

**三、轮廓提取**：  
轮廓提：取用 `OpenCV` 将边缘轮廓提取出来，这里需要用到 `findContours` 方法（**注意**：这个地方返回的参数在`OpenCV3`返回三个值：处理的图像(**image**)轮廓的点集(**contours**)各层轮廓的索引(**hierarchy**)，与`OpenCV2`返回俩个参数不同。）

```python
def findContours(image, mode, method, contours=None, hierarchy=None, offset=None)




```

**四、外接矩形**：  
提取到轮廓之后，为了方便进行判定，可以将轮廓的外界矩形计算出来，这样方便我们根据面积、位置、周长等参数进行判定，以得出该轮廓是不是目标滑块的轮廓。计算外接矩形使用的方法是 `boundingRect`，经过轮廓信息和外接矩形判定之后就能成功获取各个轮廓的外接矩形。

```python
def boundingRect(array)


```

**五、轮廓面积**：  
已经得到了各个外接矩形，但是很明显有些矩形不是我们想要的，可以根据面积、周长等来进行筛选，这里就需要用到计算面积的方法，叫做 `contourArea`，返回结果就是轮廓的面积。

```python
def contourArea(contour, oriented=None)



```

**六、轮廓周长**：  
周长的计算也有对应的方法，叫做 `arcLength`，返回结果就是轮廓的周长。

```python
def arcLength(curve, closed)



```

最后附上源码：

```python
import cv2

GAUSSIAN_BLUR_KERNEL_SIZE = (5, 5)
GAUSSIAN_BLUR_SIGMA_X = 0
CANNY_THRESHOLD1 = 200
CANNY_THRESHOLD2 = 450

def get_gaussian_blur_image(image):
    
    
    
    
    return cv2.GaussianBlur(image, GAUSSIAN_BLUR_KERNEL_SIZE, GAUSSIAN_BLUR_SIGMA_X)


def get_canny_image(image):
    return cv2.Canny(image, CANNY_THRESHOLD1, CANNY_THRESHOLD2)


def get_contours(image):
    
    image,contours,hierarchy = cv2.findContours(image, cv2.RETR_CCOMP, cv2.CHAIN_APPROX_SIMPLE)
    return contours





def get_contour_area_threshold(image_width, image_height):
	
    contour_area_min = (image_width * 0.15) * (image_height * 0.25) * 0.8
    contour_area_max = (image_width * 0.15) * (image_height * 0.25) * 1.2
    return contour_area_min, contour_area_max
def get_arc_length_threshold(image_width, image_height):
	
    arc_length_min = ((image_width * 0.15) + (image_height * 0.25)) * 2 * 0.8
    arc_length_max = ((image_width * 0.15) + (image_height * 0.25)) * 2 * 1.2
    return arc_length_min, arc_length_max
def get_offset_threshold(image_width):
	
    offset_min = 0.2 * image_width
    offset_max = 0.85 * image_width
    return offset_min, offset_max


def main():
    
    image_raw = cv2.imread('captcha.png')
    image_height, image_width, _ = image_raw.shape
    print(image_height,image_width,_)
    
    image_gaussian_blur = get_gaussian_blur_image(image_raw)
    cv2.imwrite('image_gaussian_blur.png', image_gaussian_blur)
    
    image_canny = get_canny_image(image_gaussian_blur)
    cv2.imwrite('image_canny.png', image_canny)
    
    contours = get_contours(image_canny)
    

    contour_area_min, contour_area_max = get_contour_area_threshold(image_width, image_height)
    arc_length_min, arc_length_max = get_arc_length_threshold(image_width, image_height)
    offset_min, offset_max = get_offset_threshold(image_width)
    offset = None
    for contour in contours:
        
        x, y, w, h = cv2.boundingRect(contour)
        if arc_length_min < cv2.arcLength(contour, True) < arc_length_max and\
                contour_area_min < cv2.contourArea(contour) < contour_area_max \
                and offset_min < x < offset_max:
            
            cv2.rectangle(image_raw, (x, y), (x + w, y + h), (0, 0, 255), 2)
            offset = x
    cv2.imwrite('image_label.png', image_raw)
    print('offset', offset)


if __name__ == '__main__':
    main()

```

[OpenCV利用cv2.rectangle()绘制矩形框](https://blog.csdn.net/helloworld_Fly/article/details/125136735)