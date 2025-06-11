---
date: 2025-06-11
lastmod: 2025-06-11
showTableOfContents: false
title: RaycastAI Proxy
type: post
---
一直在重度使用Raycast，AI方面之前一直使用chatGPT插件但是他有个缺点，不能继续对话。

前天reddit发现一个项目： [raycast-ai-openrouter-proxy](https://github.com/miikkaylisiurunen/raycast-ai-openrouter-proxy.git)

就是直接使用官方的AI功能，通过一个服务来实现接入其他平台的LLM API。

*也有个缺点就是不展示思考信息。*

看图：
输入内容 tab

![](https://s2.loli.net/2025/06/11/fLgcZpzUxAyRt7X.png)

继续对话：
command+J  *切换模型有提示 *

![](https://s2.loli.net/2025/06/11/at6ZRziu3hwfGec.png)

选择模型和配置prompt：
![](https://s2.loli.net/2025/06/11/LgIF2jqBhVPcZ3U.png)

## 部署+配置
### 克隆项目后
我稍微调整了一下compose
```compose
services:
  raycast-ai-proxy:
    restart: unless-stopped
    build: .
    ports:
      - '11435:3000'
    volumes:
      - ./models.json:/app/models.json:ro
    environment:
	  # 配置通过 .env 导入
      - API_KEY=${API_KEY}
      - BASE_URL=${BASE_URL}

```
### 配置参数
```.env
API_KEY="sk-xxx" # 你的key
BASE_URL="https://ark.cn-beijing.volces.com/api/v3"   # 我这里配置的火山
# BASE_URL="https://openrouter.ai/api/v1" # 默认写的 openrouter
```


### 配置模型 model.json
默认model.json提供了一些案例。
```json
{
    "name": "Doubao-1.5-vision-pro-32k",
    "id": "ep-xxx-xxx",
    "contextLength": 10240,
    "capabilities": ["vision", "tools"],  # 支持多模态和tools
    "temperature": 0.8
  },
  {
    "name": "DeepSeek-V3-0324",
    "id": "ep-xxx-xxx",
    "contextLength": 10240,
    "capabilities": ["tools"],
    "temperature": 0.6
  }
```

### 启动服务
这些搞定后启动服务：  `docker compose up -d`

## 配置Raycast

![](https://s2.loli.net/2025/06/11/dNVuhj4RW1CAFc2.png)
填写端口号， 点击sync models。

调整默认模型：

![](https://s2.loli.net/2025/06/11/dNVuhj4RW1CAFc2.png)

之后接可以聊天了，Raycast AI 支持MCP。 我没体验。

特别喜欢通过配置文件配置API的程序，之前也使用过一阵子 [intellibar](https://intellibar.app/) 但这个软件竟然对话换行有BUG。 弃了～

`cherry studio` 也在用，但有时候几句话或者一句话对话时。我更习惯通过Raycast或者小一点的窗口进行对话。

完结
祝好
