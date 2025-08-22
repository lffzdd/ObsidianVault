---
aliases: 8-图像分割-FCN
tags: []
date created: 星期二, 七月 23日 2024, 10:23:53 上午
date modified: 星期三, 七月 24日 2024, 1:51:32 凌晨
---

# 全卷积网络（Fully Convolutional Networks, FCN）简介
全卷积网络（Fully Convolutional Networks, FCN）是专门为图像分割任务设计的一类深度神经网络。与传统的卷积神经网络（CNN）不同，FCN 不包含全连接层（Fully Connected Layers），而是完全由卷积层（Convolutional Layers）组成。这样设计的目的是为了使网络能够处理任意尺寸的输入图像，并生成相应尺寸的输出特征图。
## FCN 的工作原理
FCN 的核心思想是将传统的 CNN 中的全连接层替换为卷积层，以便能够生成密集的预测。具体来说，FCN 将整个图像作为输入，通过一系列卷积和下采样操作提取特征，然后通过上采样操作将特征映射恢复到与输入图像相同的尺寸，从而生成每个像素的分类结果。以下是 FCN 的一些关键组件：
1. **卷积层（Convolutional Layers）**：提取图像的空间特征。
2. **池化层（Pooling Layers）**：减少特征图的空间分辨率，同时保留关键信息。
3. **上采样层（Upsampling Layers）**：通过反卷积（也称为转置卷积，Transposed Convolution）将特征图恢复到输入图像的原始尺寸。
## FCN 的架构
一个典型的 FCN 架构可以分为三个部分：
1. **编码器（Encoder）**：类似于传统的 CNN，由一系列卷积层和池化层组成，用于提取图像的高层次特征。
2. **解码器（Decoder）**：通过上采样层逐步恢复特征图的空间分辨率。
3. **跳跃连接（Skip Connections）**：在解码过程中，将编码器中的特征图与解码器中的特征图进行融合，以保留更多的细节信息。
## 示例代码
以下是使用 Keras 实现一个简单的 FCN 的示例代码：

```python
import tensorflow as tf
from tensorflow.keras.layers import Conv2D, MaxPooling2D, Conv2DTranspose, concatenate, Input
from tensorflow.keras.models import Model
def build_fcn(input_shape):
    inputs = Input(shape=input_shape)
    # Encoder
    c1 = Conv2D(64, (3, 3), activation='relu', padding='same')(inputs)
    p1 = MaxPooling2D((2, 2))(c1)
    c2 = Conv2D(128, (3, 3), activation='relu', padding='same')(p1)
    p2 = MaxPooling2D((2, 2))(c2)
    c3 = Conv2D(256, (3, 3), activation='relu', padding='same')(p2)
    p3 = MaxPooling2D((2, 2))(c3)
    # Decoder
    u1 = Conv2DTranspose(256, (3, 3), strides=(2, 2), padding='same')(p3)
    u1 = concatenate([u1, c3])
    u2 = Conv2DTranspose(128, (3, 3), strides=(2, 2), padding='same')(u1)
    u2 = concatenate([u2, c2])
    u3 = Conv2DTranspose(64, (3, 3), strides=(2, 2), padding='same')(u2)
    u3 = concatenate([u3, c1])
    outputs = Conv2D(1, (1, 1), activation='sigmoid')(u3)
    model = Model(inputs, outputs)
    return model
# Example usage
input_shape = (128, 128, 3)
model = build_fcn(input_shape)
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
model.summary()
```

## 原理解释
1. **编码器部分**：输入图像经过一系列卷积层和最大池化层，逐步减小空间分辨率，同时增加通道数，以提取高层次的特征。
2. **解码器部分**：使用转置卷积层（即反卷积）逐步恢复特征图的空间分辨率，并通过跳跃连接融合编码器中的相应特征图，以保留更多的细节信息。
3. **输出层**：最终通过一个 1 x 1 卷积层将特征图转换为与输入图像相同尺寸的输出图，每个像素点的值表示该像素点属于目标类别的概率。
### 参考文献
- Long, J., Shelhamer, E., & Darrell, T. (2015). Fully Convolutional Networks for Semantic Segmentation. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR) (pp. 3431-3440).
- [Keras Documentation](https://keras.io/api/layers/convolution_layers/convolution2d/)
- [TensorFlow Documentation](https://www.tensorflow.org/tutorials/images/segmentation)
通过 FCN，可以在图像分割任务中实现像素级的分类，广泛应用于医学图像处理、自动驾驶、遥感影像分析等领域。
# [[8.1-转置卷积|转置卷积]]
#### 2. **展开操作**
转置卷积在进行卷积操作前，对输入矩阵进行 " 展开 " 操作（也称为零填充）。对于一个简单的 1D 转置卷积：
- 给定输入向量 $[a, b]$ 和卷积核 $[w_1, w_2]$，卷积结果是 $[a \cdot w_1 + b \cdot w_2]$。
- 转置卷积将输出 $[a, b]$ 扩展为 $[a, 0, b]$，然后进行卷积操作。
例如，对于一个 1D 输入 $[a, b]$ 和一个核 $[w_1, w_2]$，转置卷积过程是：
1. **填充输入**：
   $[a, b] \rightarrow [a, 0, b]$
2. **卷积操作**：
   - 输出结果：$[a \cdot w_1, a \cdot w_2 + b \cdot w_1, b \cdot w_2]$
#### 3. **2D 转置卷积**
在 2D 情况下，转置卷积过程类似：
1. **输入扩展**：对于输入矩阵，每行和列之间填充零。假设步长为 2：
   $$
   \begin{bmatrix}
   1 & 2 \\
   3 & 4
   \end{bmatrix}
   \rightarrow
   \begin{bmatrix}
   1 & 0 & 2 & 0 \\
   0 & 0 & 0 & 0 \\
   3 & 0 & 4 & 0 \\
   0 & 0 & 0 & 0
   \end{bmatrix}
   $$
2. **卷积操作**：对扩展后的矩阵进行卷积操作。
   假设卷积核 $K$ 为：
   $$
   \begin{bmatrix}
   w_1 & w_2 \\
   w_3 & w_4
   \end{bmatrix}
   $$
   则输出 $Y$ 为：
   $$
   \begin{bmatrix}
   1 \cdot w_1 & 1 \cdot w_2 + 2 \cdot w_1 & 2 \cdot w_2 \\
   1 \cdot w_3 & 1 \cdot w_4 + 2 \cdot w_3 + 3 \cdot w_1 & 2 \cdot w_4 + 3 \cdot w_2 \\
   3 \cdot w_3 & 3 \cdot w_4 + 4 \cdot w_3 & 4 \cdot w_4
   \end{bmatrix}
   $$
   这里，转置卷积通过卷积核对输入的每个非零元素计算卷积，并将结果累加到输出矩阵中。
## 转置卷积的特点
1. **上采样**：通过对输入进行扩展和卷积运算，实现从低分辨率到高分辨率的映射。
2. **步长与填充**：转置卷积的步长和填充与常规卷积相反，目的是增加输出尺寸而不是减少。
3. **参数共享**：卷积核参数在计算中共享，减少了模型参数数量。
## 转置卷积的实现
在深度学习框架中，转置卷积通常作为一层实现，如 TensorFlow 中的 `Conv2DTranspose`：

```python
import tensorflow as tf
from tensorflow.keras.layers import Conv2DTranspose, Input
# 输入特征图的形状
input_shape = (None, 8, 8, 64)  # 假设输入是 8x8 的 64 个通道
# 创建一个模型
input_tensor = Input(shape=(8, 8, 64))
x = Conv2DTranspose(filters=32, kernel_size=(3, 3), strides=(2, 2), padding='same', activation='relu')(input_tensor)
# 输出形状将是 (16, 16, 32)，即空间分辨率放大了一倍
model = tf.keras.Model(inputs=input_tensor, outputs=x)
# 打印模型结构
model.summary()
```

### 总结
转置卷积通过对输入进行扩展和卷积运算，实现从低分辨率到高分辨率的映射。这种机制在需要上采样的任务中具有广泛应用。理解其数学原理有助于设计有效的深度学习模型，特别是在图像分割和生成对抗网络等应用中。
# ![[4-add和concatenate的区别#FCN 中的区别|concatenate不会导致特征图尺寸增加]]

