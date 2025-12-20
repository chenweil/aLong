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

![](https://cdn.img2ipfs.com/ipfs/QmW6SGg6y18UQoQAW7cMNTB61N1jVj5qY8hSsWnCVWCHxf?filename=f1873b2e4928ebc46314373ea547f493802c5ae0_2_508x1000.png)

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

	![](https://cdn.img2ipfs.com/ipfs/QmTuR59WTVJGE8oRq67Zpet5yLFZAUUWDeKLUp9WwCng7s?filename=d5f4cfe3341d5d118a28778da218e526f3b7676f_2_582x998.png)

	![](https://cdn.img2ipfs.com/ipfs/QmWkSuVvDf6i3XpAitwrEcVBd3tz6LbthxpEr7yN1S8FAG?filename=e2001e0829bff91314c146374584ca0ec8dcaacd_2_1210x1000.webp)
  每个模型单独文件，通过`models.yaml`加载
  
- **密钥安全**

	![](https://cdn.img2ipfs.com/ipfs/QmeGUm7QZpySt9t6KFhARE6gWib5iDrcGauPxubhKSJvAS?filename=9fa341e6e62789ea17e861caa7ba5cab564457a8_2_1380x932.png)

  用`.env`隔离敏感信息，避免Git泄露

---

### 🔌 客户端接入实战

#### 1. Cherry Studio

配置时注意URL补全问题：

![](https://cdn.img2ipfs.com/ipfs/QmdcrqKrv41w9Atc7UaXEFsLdP9xuQ41BvCF7bkBqLJPnS?filename=a5fbb5a36a0193d0dcbbb7042828bf00e79b4239_2_1172x1000.png)

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

![](https://cdn.img2ipfs.com/ipfs/QmSY7Z6ULdxXdwADKUvf8TrkNNFU1sv94BrtLepA8aVeqQ?filename=56059eb3ecf2176fdf8941f4011030c57c24af4f_2_1380x510.png)


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

![](https://cdn.img2ipfs.com/ipfs/QmWF7QUFA5vzL99QCx6HFqYpXfWV8YpMnABPPdok1pBLQL?filename=9ef06aa785cf8b2b9d22537745511a17365219a4_2_1380x400.png)

端口3456，证明走的ccr。

ccr 切换模型注意语法，他需要 加入name的。

操作效果：

![](https://cdn.img2ipfs.com/ipfs/QmdaVi8PEJnRERgZ9vvPsdobntQxZxWHfk6MPCvWHEjwkF?filename=8d157436d1dd2b1a613c030006ee0d31aff0ac57_2_798x998.png)

哈哈 这家伙的话让我不爽，结果👌。

使用ccr 可以实现，注意litellm貌似再支持claude code的功能上不支持模型大写。这是个bug。 ccr接入litellm没问题，cherry studio 接入没问题。

ccr 配置：
```
{
  "LOG": false,
  "Providers": [
    {
      "name": "litellm",
      "api_base_url": "http://localhost:4000/v1/chat/completions",
      "api_key": "sk-2015",
      "models": [
        "claude-3-5-sonnet",
        "claude-3-7-sonnet",
        "claude-4-sonnet",
        "claude-4-sonnet-think",
        "claude-3-7-sonnet-think",
        "gemini-2.5-flash",
        "gemini-2.5-pro-preview-05-06",
        "gpt-4o",
        "gpt-4o-mini",
        "o4-mini",
        "gpt-4.1",
        "gpt-4.1-mini",
        "doubao-1.6-thinking",
        "kimi-k2",
        "Qwen3-235B-A22B-Instruct-2507",
        "Qwen3-Coder-480B-A35B-Instruct"
      ]
    },
  "Router": {
      "default": "litellm,Qwen3-Coder-480B-A35B-Instruct",
      "background": "litellm,gpt-4.1-mini",
      "think": "deepseek,deepseek-r1",
      "longContext": "litellm,gemini-2.5-flash"
    }
}
```

---

### 💡 小结


这套方案显著降低了多平台API的管理成本——建议有类似需求的朋友尝试！也再次感谢提供福利的各位大佬 🙏

> 项目启动：`docker-compose up -d`
> 排错提示：通过日志观察路由策略
> ![](https://cdn.img2ipfs.com/ipfs/QmYzKtZ3xkR6krQe3hx6VCxCyjf8JiPjeMMRS86JCBGnkx?filename=7a8d07dc2f3551cad7a8de60040fc065fff23cc3_2_1380x608.png)


完结
祝好
🍀
