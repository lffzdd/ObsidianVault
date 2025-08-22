[Site Unreachable](https://zhuanlan.zhihu.com/p/123211148)
![[Pasted image 20250414155713.png]]
```Python
import torch  
import torch.nn as nn  
from torch import Tensor  
  
  
class SimpleRNN(nn.Module):  
    def __init__(self, input_size, hidden_size, output_size):  
        super(SimpleRNN, self).__init__()  
        self.hidden_size = hidden_size  
  
        # 标准 RNN 计算逻辑：  
        self.i2h = nn.Linear(input_size + hidden_size, hidden_size)  # 输入 + 之前的隐藏状态 -> 新的隐藏状态  
        self.h2o = nn.Linear(hidden_size, output_size)  # 隐藏状态 -> 输出  
        self.activation = nn.Tanh()  # RNN 经典的激活函数（可以换成 ReLU）  
  
    def forward(self, x: Tensor) -> Tensor:  
        """  
        x: (batch_size, seq_length, input_size)        """        batch_size, seq_length, _ = x.shape  
  
        # 初始化隐藏状态 h0 (batch_size, hidden_size)        hidden = torch.zeros(batch_size, self.hidden_size, device=x.device)  
  
        # 遍历时间步  
        for t in range(seq_length):  
            xt = x[:, t, :]  # 取出当前时间步的输入 (batch_size,  ,input_size)            
            combined = torch.cat((xt, hidden), dim=1)  # 拼接当前输入和之前的隐藏状态  
            hidden = self.activation(self.i2h(combined))  # 计算新的隐藏状态  
  
        # 最后一个隐藏状态作为最终输出  
        output = self.h2o(hidden)  # (batch_size, output_size)  
        return output  
  
  
# 测试  
batch_size = 2  
seq_length = 5  # 5 个时间步  
input_size = 3  # 每个时间步的输入有 3 维  
hidden_size = 4  # 隐藏状态的大小  
output_size = 2  # 输出 2 维  
  
rnn = SimpleRNN(input_size, hidden_size, output_size)  
x = torch.randn(batch_size, seq_length, input_size)  # 生成一个随机序列输入  
output = rnn(x)  
  
print("输入形状:", x.shape)  # (batch_size, seq_length, input_size)  
print("输出形状:", output.shape)  # (batch_size, output_size)
```