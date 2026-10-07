# Let's Buy — Virtual Try-On Shopping Prototype

[English](#english) · [中文](#中文) · [Demo video](演示视频.mp4) · [OOTDiffusion upstream](https://github.com/levihsu/OOTDiffusion)

**Tech stack:** WeChat Mini Program · Python · PyTorch · Diffusers · Gradio · OpenPose · human parsing · OOTDiffusion

## English

Let's Buy explores how a virtual try-on model can fit into a shopping journey: select a person and garment image, wait for GPU inference, inspect the generated outfit, and return to product, cart, order, and account flows.

> **Ownership note:** OOTDiffusion is an upstream research project. The model architecture, inference pipelines, and most preprocessing code under `tryon/1/` are not presented as my original model work. My project contribution is the product/integration layer around that stack: shaping inputs and outputs into a try-on flow, connecting it to the mini-program concept, and preparing the end-to-end demonstration.

### Demo first

The repository includes a checked-in [MP4 demonstration](演示视频.mp4). It is the clearest record of the intended user flow and current prototype behaviour.

### Product and inference flow

```text
shopping UI
  -> choose/upload person + garment images
  -> select half-body or full-body mode
  -> pose estimation + human parsing + mask preparation
  -> OOTDiffusion inference
  -> generated preview
  -> continue browsing/cart/order flow
```

### What I integrated

- A shopping experience organised around home/product browsing, virtual try-on, cart, order, and user-centre states.
- Person/garment input preparation and the interaction contract around a GPU-heavy inference step.
- OOTDiffusion's half-body (`hd`) and full-body/category-aware (`dc`) runners.
- Gradio and command-line entry points for repeatable local experiments.
- Demo assets that show the model result in the context of a consumer workflow rather than as an isolated notebook output.

### Repository map and current checkout status

| Path | What it contains | Ownership/status |
| --- | --- | --- |
| `tryon/1/ootd/` | OOTDiffusion inference and pipeline code | Upstream model code |
| `tryon/1/preprocess/` | OpenPose and human-parsing dependencies | Upstream/vendored components |
| `tryon/1/run/` | Gradio and CLI runners plus example inputs | Upstream runner adapted for the prototype context |
| `miniprogram (2)/miniprogram` | Gitlink to the mini-program snapshot | The repository is missing `.gitmodules`, so a fresh clone cannot resolve this link automatically |
| `演示视频.mp4` | Product demonstration | Viewable directly from this repository |

### Running the model side

The repository does not include the large model checkpoints or a complete environment lockfile. Prepare the OOTDiffusion, CLIP/VAE, OpenPose, and human-parsing weights expected by `tryon/1/`, then run either:

```bash
cd tryon/1
python run/gradio_ootd.py

# or a single CLI inference
python run/run_ootd.py --model_path PERSON.jpg --cloth_path GARMENT.jpg --model_type hd
```

A CUDA-capable GPU is strongly recommended. The runners use 768×1024 inputs and expose 20–40 inference steps in the Gradio UI.

### What is and is not demonstrated

The video and checked-in runner show the intended try-on experience and local inference path. The current repository does **not** preserve a complete deployable API bridge between the mini-program and inference process, and the unresolved gitlink prevents a clean checkout of the front end. Those repository gaps should be fixed before presenting this as a production-ready system.

## 中文

Let's Buy 探索的是如何把虚拟试衣模型放进完整购物流程：选择人物图和服装图，等待 GPU 推理，查看生成穿搭，再回到商品、购物车、订单和个人中心。

> **署名说明：** OOTDiffusion 是上游研究项目。`tryon/1/` 中的模型架构、推理 pipeline 和大部分预处理代码不属于我的原创模型实现。我的项目工作集中在产品与集成层：定义试衣输入输出流程、把模型能力放进小程序购物场景，并完成端到端演示。

### 演示

仓库中保留了 [MP4 演示视频](演示视频.mp4)，它是当前用户流程和原型效果最直接的证据。

### 我的集成工作

- 围绕首页/商品浏览、虚拟试衣、购物车、订单和个人中心组织购物体验；
- 处理人物图与服装图的输入准备，以及 GPU 推理期间的交互状态；
- 接入 OOTDiffusion 的半身和全身/服装类别推理入口；
- 保留 Gradio 与命令行运行方式，方便本地重复实验；
- 用演示素材把模型输出放回消费场景，而不是只展示离线 notebook 结果。

### 仓库现状

`tryon/1/ootd/` 和 `tryon/1/preprocess/` 主要是上游模型及依赖；`tryon/1/run/` 提供 Gradio/CLI 入口。`miniprogram (2)/miniprogram` 当前是一个 gitlink，但仓库缺少 `.gitmodules`，所以新用户无法通过普通 clone 自动取得小程序源码。演示视频可以直接查看，但仓库目前没有保留完整、可部署的小程序到推理服务 API 桥接。

### 运行与边界

仓库不包含大型 checkpoint，也没有完整锁定的运行环境。准备好 OOTDiffusion、CLIP/VAE、OpenPose 和 human-parsing 权重后，可按上面的命令启动 Gradio 或 CLI；建议使用 CUDA GPU。

这是学习/研究型集成原型，不应描述为自研虚拟试衣模型或生产系统。若继续完善，应补回可解析的小程序来源、明确前后端 API、锁定依赖，并补充图片留存、队列、监控和许可证说明。
