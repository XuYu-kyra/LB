# Let’s Buy — Virtual Try-On Shopping Mini Program

[English](#english) · [中文](#中文)

## English

Let’s Buy is an AI-assisted shopping prototype that connects a WeChat mini-program experience to a Python virtual try-on service built around OOTDiffusion. A user can browse products, upload a person image and a garment image, and preview a generated outfit before deciding what to buy.

### The product story

The interesting problem is not only image generation. A useful try-on demo has to connect a heavy vision model to a complete shopping journey: product discovery, upload states, inference progress, result display, cart, orders, and user centre. I treated the project as an integration problem and kept the model service and consumer-facing flow explicit.

### What I built and integrated

- A native WeChat mini-program with home/product browsing, try-on, cart, order, and user-centre pages.
- A Python inference service exposing OOTDiffusion pipelines for half-body and full-body try-on modes.
- Pre-processing components for pose estimation, human parsing, image resizing, and garment/person preparation.
- A Gradio interface and CLI entry points for repeatable local model experiments.
- Clear separation between front-end interaction state and GPU-heavy back-end inference, making the prototype easier to demo and extend.

### Architecture

```text
WeChat mini-program
  -> upload person + garment -> try-on request
  -> Python/OOTDiffusion service
  -> pose + human parsing + diffusion inference
  -> generated preview -> product/cart/order flow
```

### Local model setup

The repository does not bundle large checkpoints. Prepare the required OOTDiffusion, CLIP/VAE, OpenPose, and human-parsing weights under the paths expected by `tryon/1/`, then use the Gradio or CLI runner. A GPU is strongly recommended; start with `768x1024` inputs and 20–40 inference steps, then tune for available memory and latency.

This is a learning/research prototype. Production work would add authenticated storage, image retention controls, queueing, model observability, abuse prevention, and a clearer licence for model checkpoints and generated content.

## 中文

Let’s Buy 是一个 AI 虚拟试衣购物原型：前端是微信小程序，后端是基于 OOTDiffusion 的 Python 推理服务。用户可以浏览商品，上传人物图和服装图，在购买前预览生成的穿搭效果。

### 项目故事

虚拟试衣的难点不只是“把图片生成出来”，还在于如何把一个重量级视觉模型接入完整的购物流程：商品浏览、上传状态、推理等待、结果展示、购物车、订单和个人中心。我把它当成一次端到端产品集成，明确拆开小程序交互层和 GPU 推理层。

### 我的主导工作

- 实现微信小程序首页/商品浏览、虚拟试衣、购物车、订单和个人中心页面；
- 集成 OOTDiffusion 的半身与全身试衣 pipeline，并提供 Gradio 与 CLI 入口；
- 串接姿态估计、人体解析、图像预处理和服装/人物输入准备；
- 让前端交互状态与后端 GPU 推理解耦，便于演示、调参和后续扩展；
- 组织从“选择服装”到“生成预览”再回到购物流程的完整用户路径。

### 运行提示

仓库不包含大型 checkpoint。请按 `tryon/1/` 的路径准备 OOTDiffusion、CLIP/VAE、OpenPose 和 human parsing 权重，再启动 Gradio 或命令行 runner。建议使用 GPU，从 `768x1024`、20–40 steps 开始，根据显存和延迟调整。

这是学习/研究原型；若继续产品化，还需要补充鉴权存储、图片留存策略、任务队列、模型监控、滥用防护，以及 checkpoint 和生成内容的许可证说明。
