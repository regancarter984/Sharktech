# 使用 Poe-API-wrapper 整合 DALLE 和 ChatGPT，实现批量 AI 绘图与文字创作

Poe 是一个备受用户喜爱的平台，尤其适合那些希望通过 API 接口进行高效创作的用户。作为 Poe 的忠实用户，我曾经在多个项目中利用其 API 接口进行深度开发。今天，我将分享如何通过 `poe-api-wrapper` 工具，连接 DALLE-3 和 ChatGPT，实现批量 AI 绘图与文字创作。



## 什么是 `poe-api-wrapper`？

`poe-api-wrapper` 是 Poe 的一个 Python API 接口库，基于 Httpx 技术开发。通过它，你可以免费连接到多种 AI 服务，包括 ChatGPT、DALLE-3、Claude、Mistral 和 Gemini 等。更重要的是，该工具共享你在 Poe 的使用额度，不会产生额外费用。



## 如何使用 `poe-api-wrapper`？

### 1. 安装与初始化
首先，你需要获取 Poe 的 API token。以下是初始化代码示例：

python
from poe_api_wrapper import PoeApi

token = "yourtoken"
client = PoeApi(token)


### 2. 批量生成图片
通过 `poe-api-wrapper`，你可以轻松批量生成 AI 绘图。以下是一个生成多张图片的代码示例：

python
from poe_api_wrapper import PoeApi
import requests
import os

# 初始化 PoeApi 客户端
token = "yourtoken"
client = PoeApi(token)

# 从文件中读取提示列表
with open("draw-input.txt", "r") as file:
    prompts = file.readlines()

# 去掉每行末尾的换行符，并过滤掉空行
prompts = [prompt.strip() for prompt in prompts if prompt.strip()]

# 定义保存图片的文件夹路径
save_folder = "/Users/yourpath"
# 确保保存图片的文件夹存在
if not os.path.exists(save_folder):
    os.makedirs(save_folder)

# 初始化 chatCode
chatCode = None

# 循环遍历提示列表，并发送给 DALL-E 3
for prompt in prompts:
    print(f"正在请求: {prompt}")
    for _ in range(4):  # 重复生成4次图片
        for chunk in client.send_message("DALL-E-3", prompt, chatCode=chatCode, timeout=60):
            # 如果是新的对话，保存 chatCode
            if not chatCode and 'chatCode' in chunk:
                chatCode = chunk['chatCode']
                print(f"新对话创建 | chatCode: {chatCode}")

            # 检查 chunk 中是否包含图片 URL
            if 'response' in chunk and chunk['response'].startswith('!['):
                image_url = chunk['response'].split('(')[1].split(')')[0]
                print(f"生成的图片 URL: {image_url}")
                
                # 获取图片的内容
                response = requests.get(image_url)
                if response.status_code == 200:
                    # 从 URL 中提取图片文件名
                    filename = os.path.basename(image_url.split('?')[0])
                    # 定义图片保存路径
                    file_path = os.path.join(save_folder, filename)
                    
                    # 保存图片
                    with open(file_path, 'wb') as file:
                        file.write(response.content)
                    print(f"图片已保存到: {file_path}")
                else:
                    print(f"下载图片失败: {response.status_code}")


### 3. 连续生成对话
除了绘图，你还可以使用 `poe-api-wrapper` 进行连续对话。以下是一个生成对话的代码示例：

python
from poe_api_wrapper import PoeApi
import datetime

# 初始化 PoeApi 客户端
token = "yourtoken"
client = PoeApi(token)

bot = "Claude-2-100k"

# 从文件中读取消息列表
with open("chat-input.txt", "r") as file:
    messages = [line.strip() for line in file if line.strip()]  # 过滤掉空行

# 初始化 chatCode
chat_code = None

# 创建文件用于保存结果
now = datetime.datetime.now()
timestamp = now.strftime("%Y%m%d%H%M%S")
result_file = open(f"/Users/yourpath/result_{timestamp}.md", "w")

# 循环发送消息
for message in messages:
    if chat_code:
        response_gen = client.send_message(bot, message, chatCode=chat_code)
    else:
        response_gen = client.send_message(bot, message)

    for chunk in response_gen:
        if 'chatCode' in chunk and not chat_code:
            chat_code = chunk['chatCode']
        
        # 写入去除了前导空格的回应
        result_file.write(chunk["response"].lstrip())  # 使用 lstrip() 移除左侧空格
        result_file.flush()
    
    # 在每个结果后增加一个额外的空行
    result_file.write("\n\n")

# 关闭结果文件
result_file.close()


## 应用场景

### 1. 批量绘图
通过 `poe-api-wrapper`，你可以批量生成多张 AI 绘图。只需将提示词存储在 `draw-input.txt` 文件中，运行代码即可自动生成图片。



### 2. 连续对话
你可以利用 `poe-api-wrapper` 与多个 AI 模型进行连续对话。通过 `chat-input.txt` 文件存储对话关键词，程序将自动批量处理这些关键词。



### 3. 批量写作
想象一下，你需要在短时间内完成多篇作文或文章。通过 `poe-api-wrapper`，你可以批量生成多篇不同版本的文稿，极大提高了写作效率。



---

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bbtdd.com/WildCard)

## 总结

通过 `poe-api-wrapper`，你可以充分利用 Poe 的 API 接口，实现批量 AI 绘图与文字创作。无论是批量生成图片、连续对话，还是批量写作，该工具都能显著提高你的工作效率。希望本文能为你提供新的创作灵感，助你在 AI 领域取得更多成就。