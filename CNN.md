卷积神经网络基本结构就是：卷积/池化-->全连接。卷积提取特征，池化层防止过拟合，全连接进行输出或分类。

### 卷积操作   

#### 二维卷积  

使用卷积核对应像素位置相乘求和作为新特征图的像素，有几个卷积核就有几个特征图。

可视化网站：https://ezyang.github.io/convolution-visualizer/index.html 

#### 三维卷积  

卷积核多了一个通道维度，卷积时同样是对应像素位置相乘并求和作为新的特征图的像素点。一个卷积核对应一个特征图

可视化网站：https://thomelane.github.io/convolutions/2DConvRGB.html

![image-20251102182209963](https://raw.githubusercontent.com/Aurora234/picture_bed/main/yolo_v1/202511021828620.png)

#### 池化

包括平均值池化和最大值池化，常用最大值池化
池化的作用包括：

1. 减少参数数量
2. 防止过拟合，增强泛化能力
3. 平移不变性