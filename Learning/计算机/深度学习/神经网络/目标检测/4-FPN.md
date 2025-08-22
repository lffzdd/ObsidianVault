![[Pasted image 20240914112033.png]]
### 代码示例
```python
import tensorflow as tf
from tensorflow.keras import layers, models
from tensorflow.keras.applications import ResNet50
def build_fpn(input_shape=(224, 224, 3)):
    # Backbone: ResNet50 (pretrained on ImageNet)
    base_model = ResNet50(include_top=False, input_shape=input_shape)
    # Get feature maps from different layers (C2, C3, C4, C5)
    C2 = base_model.get_layer("conv2_block3_out").output  # Output of stage 2
    C3 = base_model.get_layer("conv3_block4_out").output  # Output of stage 3
    C4 = base_model.get_layer("conv4_block6_out").output  # Output of stage 4
    C5 = base_model.get_layer("conv5_block3_out").output  # Output of stage 5
    # 1x1 Conv to reduce channels (lateral connections)
    P5 = layers.Conv2D(256, (1, 1), padding='same')(C5)
    P4 = layers.Conv2D(256, (1, 1), padding='same')(C4)
    P3 = layers.Conv2D(256, (1, 1), padding='same')(C3)
    P2 = layers.Conv2D(256, (1, 1), padding='same')(C2)
    # Top-down pathway with upsampling
    P4 = layers.Add()([P4, layers.UpSampling2D(size=(2, 2))(P5)])
    P3 = layers.Add()([P3, layers.UpSampling2D(size=(2, 2))(P4)])
    P2 = layers.Add()([P2, layers.UpSampling2D(size=(2, 2))(P3)])
    # 3x3 Conv to smooth the features
    P5 = layers.Conv2D(256, (3, 3), padding='same')(P5)
    P4 = layers.Conv2D(256, (3, 3), padding='same')(P4)
    P3 = layers.Conv2D(256, (3, 3), padding='same')(P3)
    P2 = layers.Conv2D(256, (3, 3), padding='same')(P2)
    # FPN outputs: P2, P3, P4, P5
    model = models.Model(inputs=base_model.input, outputs=[P2, P3, P4, P5])
    return model
# Build the FPN model and show its structure
fpn_model = build_fpn()
fpn_model.summary()
```

1. **Backbone**：使用了预训练的 ResNet50 作为骨干网络，移除了全连接层，只提取卷积特征层。
   - `C2` 对应的是 ResNet 的第二层输出（低语义高分辨率）。
   - `C3`, `C4`, `C5` 对应更高层的输出，随着层数增加，分辨率下降，但语义信息丰富。
2. **Lateral Connections**：我们对每个层进行 \( 1 \times 1 \) 卷积，**降低通道数到 256，保证特征图维度统一**。
3. **Top-Down Pathway**：从最深的特征图 \( P5 \) 开始，**逐步**向上进行上采样并与浅层特征图相加。每一步都对结果进行卷积平滑处理。
4. **FPN 输出**：最后输出四个不同尺度的特征图 \( P2, P3, P4, P5 \)，可以用于目标检测的不同分辨率特征金字塔。
