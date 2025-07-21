---
title: "一站式解决多API管理痛点"
date: 2025-07-21 02:31:11
categories:
  - AI
tags:
  - API网关
  - 开发工具
type: "post"
---

虽然许多平台提供丰富的API福利，但大多数被闲置了：有些有效期短暂，有些注册后就被遗忘。更麻烦的是不同平台的模型命名混乱、接入方式各异。

最近发现**LiteLLM**这个项目深得我心——它能统一管理所有AI模型API！通过搭建代理网关，只需一个URL和Key就能调用任意模型，就像把法宝都装进乾坤袋✨

> 📌 **解决的核心问题：**
>
> 1. **自动容错**：API失效时自动冷却重试，避免手动切换
> 2. **福利整合**：集中管理所有API Key，告别Cherry Studio重复注册
> 3. **名称统一**：自定义模型组名（如`gemini-2.5-flash`兼容多版本）
> 4. **跨工具兼容**：通过统一接口接入ChatGPT/Claude等客户端

---

### 🛠 搭建实录（本地版）

> 因代理环境在本地，选择本地部署。服务器部署需注意规避网络限制问题

#### 📂 目录结构

```
.
├── docker-compose.yml
├── .env            # 密钥管理
└── config/
    ├── config.yaml # 主配置
    ├── router.yaml # 路由策略
    └── models/     # 模型配置集
```

#### ⚙️ 关键配置

**1. Docker-Compose**（精简版）

```yaml
services:
  litellm:
    image: ghcr.io/berriai/litellm:main-stable
    volumes:
      - ./config:/app/config   # 挂载整个配置目录
    command: --config=/app/config/config.yaml
    ports:
      - "4000:4000"
    env_file:
      - .env  # 安全加载密钥
```

**2. 配置分层设计**

- `config.yaml` 引用子配置，核心参数示例：

```yaml
include: [models.yaml, router.yaml] # 模块化加载

litellm_settings:
  num_retries: 3     # 失败重试
  request_timeout: 10
  drop_params: true  # 清理冗余参数
```

- `router.yaml` 路由策略（关键！）

```yaml
router_settings:
  routing_strategy: latency-based-routing # 延迟优先
  fallbacks:
    - "*": ["gemini-2.5-flash"]  # 全局备用模型
  model_groups:                 # !!!模型名称标准化!!!
    - model_name: "claude-3-5-sonnet"
      model_list: # 聚合不同命名格式
        - "claude-3-5-sonnet-20241022"
        - "claude-3.5-sonnet"
```

- **模型配置**
  ![](https://s2.loli.net/2025/07/22/YMwVufQy5nqioSm.png)
  每个模型单独文件，通过`models.yaml`加载
- **密钥安全**
  ![](https://s2.loli.net/2025/07/22/3ua7dzSUEcp1GHl.png)
  用`.env`隔离敏感信息，避免Git泄露

---

### 🔌 客户端接入实战

#### 1. Cherry Studio

配置时注意URL补全问题：
![](https://s2.loli.net/2025/07/22/6GbOwiNZPMRxnmL.png)
✅ 推荐填写 `http://localhost:4000` 或 `http://localhost:4000/v1/`

#### 2. Claude Code

通过环境变量指向LiteLLM网关：

```json
// claude.json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:4000",
    "ANTHROPIC_AUTH_TOKEN": "sk-2015"
  }
}


```
claude 切换模型：
![](https://s2.loli.net/2025/07/22/D8PuYnF1dCOgpJG.png)


#### 3. CCR路由进阶

使用Claude Code Router实现动态模型分配：

```json
{
  "Providers": [
    {
      "name": "litellm",
      "api_base_url": "http://localhost:4000/v1",
      "models": ["claude-3-7-sonnet", "gemini-2.5-pro"]
    },
    {
      "name": "deepseek",
      "api_base_url": "http://localhost:4000/v1",
      "models": ["deepseek-r1"]
    }
  ],
  "Router": {
    "default": "litellm,claude-3-7-sonnet",
    "think": "deepseek,deepseek-r1" # 指定场景路由
  }
}
```

启动：
![](https://s2.loli.net/2025/07/22/1EKP7rqVYTilZCo.png)

端口3456，证明走的ccr。

ccr 切换模型注意语法，他需要 加入name的。

操作效果：
![](https://s2.loli.net/2025/07/22/8mjsbGoe7HFB4QC.png)

哈哈 这家伙的话让我不爽，结果👌。

---

### 💡 小结


这套方案显著降低了多平台API的管理成本——建议有类似需求的朋友尝试！也再次感谢提供福利的各位大佬 🙏

> 项目启动：`docker-compose up -d`
> 排错提示：通过日志观察路由策略
> ![](https://s2.loli.net/2025/07/22/5lu4DQLga81P6sj.png)


完结
祝好
🍀
