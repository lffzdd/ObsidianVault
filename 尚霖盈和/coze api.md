```python
import os

from volcenginesdkarkruntime import Ark

import base64

  

# 初始化Ark客户端，从环境变量中读取API Key

client = Ark(api_key=os.getenv('ARK_API_KEY'))

  

# 定义方法将本地图片转为Base64编码

def encode_image(img_path):

with open(img_path, "rb") as image_file:

return base64.b64encode(image_file.read()).decode('utf-8')

  

# 图片路径

image_path = "example_image.png"

  

# 将图片转为Base64编码

base64_image = encode_image(image_path)

  

# 调用API进行图片理解

response = client.chat.completions.create(

model="doubao-1-5-vision-pro-32k-250115", # 替换为实际模型ID

messages=[

{

"role": "user",

"content": [

{"type": "text", "text": "请描述图片中的内容"},

{"type": "image_url", "image_url": {"url": f"data:image/png;base64,{base64_image}"}}

],

}

],

)

  

# 输出结果

print("分析结果:", response.choices[0].message.content)
```