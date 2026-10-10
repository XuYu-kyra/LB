# Let's Buy

**AI Virtual Try-On Shopping Mini Program · AI 虚拟试衣购物小程序**

[中文](#中文) · [English](#english) · [演示视频 / Demo](演示视频.mp4) · [OOTDiffusion](https://github.com/levihsu/OOTDiffusion)

Let's Buy 将 OOTDiffusion 的虚拟试衣能力放进微信小程序购物流程：用户从商品浏览进入试衣，选择人物图与服装图，生成穿搭预览，再继续查看商品、购物车和订单。

Rather than presenting virtual try-on as an isolated model demo, Let's Buy places it inside a WeChat Mini Program shopping journey—from product discovery and image selection to generated previews, cart, orders, and account flows.

---

## 中文

### 项目简介

Let's Buy 是一个围绕 AI 虚拟试衣设计的购物小程序原型。项目关注的不只是“模型能否生成一张换装图片”，而是如何把这项能力转化为用户能够理解和操作的购物体验：

1. 浏览商品并进入虚拟试衣；
2. 选择人物图片和服装图片；
3. 根据服装类型选择半身或全身试衣模式；
4. 等待姿态识别、人体解析和扩散模型推理；
5. 查看生成结果，并继续完成商品、购物车、订单或个人中心相关操作。

项目使用 [OOTDiffusion](https://github.com/levihsu/OOTDiffusion) 作为虚拟试衣模型基础。我独立完成了微信小程序页面与交互流程，并在模型侧验证半身和全身试衣能力；演示视频按照设计好的端到端调用链路，呈现了从商品浏览、图片选择到试衣结果展示的目标体验。

### 演示

仓库中提供了 [项目演示视频](演示视频.mp4)，展示虚拟试衣能力在购物场景中的交互方式和整体页面流程。

演示重点不是单独运行一个模型 notebook，而是呈现从用户选择人物与服装，到获得试衣结果并返回购物流程的完整体验。

> **实现范围：** 小程序页面、购物流程、虚拟试衣交互和模型侧推理验证均由我独立完成。我同时设计了小程序与 OOTDiffusion 推理服务之间的 HTTP 接口契约；最终交付分别验证小程序体验和模型推理，演示视频按这套契约呈现完整的端到端流程。

### 我在项目中的工作

我的工作重点是把已有的虚拟试衣模型能力转化为一个面向用户的应用原型，而不是重新实现 OOTDiffusion 模型本身。

- **小程序独立实现**：完成首页、商品浏览、虚拟试衣、购物车、订单和个人中心等页面及其交互流程，让试衣结果能够自然地回到购物决策中。
- **交互信息建模**：把人物图、服装图、试衣模式、服装类别、生成状态和结果整理为用户可理解的操作步骤，避免直接暴露复杂的模型调用过程。
- **调用方案设计**：围绕 OOTDiffusion 的半身（HD）与全身/分类别（DC）能力，设计图片上传、任务创建、状态查询和结果返回的接口契约。
- **模型侧验证**：使用上游提供的 Gradio 和命令行入口检查不同人物、服装和推理参数组合，验证半身、上装、下装和连衣裙等试衣路径。
- **端到端展示**：整理示例素材和演示视频，将模型输出放回真实的购物页面与用户操作上下文，而不是只展示离线生成结果。

这部分工作覆盖了小程序实现、AI 能力集成设计、产品流程拆解和前后端边界设计：模型负责生成，应用层负责把输入、等待状态、类别约束和输出结果转化为连续的用户体验。

### 从购物操作到试衣结果

```text
微信小程序购物界面
    │
    ├── 浏览商品 / 进入虚拟试衣
    │
    ├── 选择人物图、服装图和服装类别
    │
    ▼
试衣输入
    │
    ├── OpenPose：提取人体关键点
    ├── Human Parsing：识别人体与服装区域
    ├── Mask Preparation：构建需要重绘的区域
    │
    ▼
OOTDiffusion
    │
    ├── HD：上半身服装试衣
    └── DC：上装、下装和连衣裙试衣
    │
    ▼
生成试衣预览
    │
    └── 返回商品浏览、购物车、订单等购物流程
```

这条链路中，姿态关键点和人体解析结果用于确定服装应当覆盖与保留的区域；扩散模型根据人物、服装和掩码生成试衣结果；小程序则负责承接用户操作、展示生成状态和组织结果页面。

### 小程序如何调用 OOTDiffusion

我为小程序与模型服务设计的是异步任务式接口。虚拟试衣属于耗时明显高于普通页面请求的 GPU 推理任务，因此不让小程序持续等待一个长连接，而是把“上传素材”“创建任务”和“查询结果”拆开：

```text
wx.chooseMedia
    │  选择人物图和服装图
    ▼
wx.uploadFile
    │  分别上传图片，获得 person_asset_id / garment_asset_id
    ▼
POST /api/v1/try-on/jobs
    │  提交图片 ID、试衣模式和服装类别
    ▼
Python inference service
    │  OpenPose → Human Parsing → Mask → OOTDiffusion
    ▼
GET /api/v1/try-on/jobs/{task_id}
    │  小程序使用 wx.request 查询任务状态
    ▼
result_url
       在试衣结果页展示，并返回商品/购物流程
```

创建任务时，小程序只需要提交与产品交互相关的信息：

```json
{
  "person_asset_id": "person_xxx",
  "garment_asset_id": "garment_xxx",
  "mode": "dc",
  "category": "dress"
}
```

服务端将 `mode=hd` 映射为半身上装试衣；将 `mode=dc` 与 `upperbody`、`lowerbody` 或 `dress` 组合映射到全身推理入口。任务创建后先返回 `task_id`，小程序根据 `processing / completed / failed` 状态更新等待页面，完成后使用 `result_url` 展示生成图片。

这个接口划分把小程序交互与模型实现解耦：前端不需要了解 OpenPose、人体解析或掩码生成细节，模型服务也不依赖具体页面结构。演示视频按这一交互契约呈现了最终用户流程；仓库中的 Gradio/CLI 则用于独立验证模型侧输入与输出。

#### 推理等待如何处理

虚拟试衣很难做到与普通页面接口一样“立即返回”。设计重点因此不是把等待时间隐藏掉，而是同时缩短实际计算时间，并让等待过程保持可理解、可恢复：

- **模型常驻显存**：服务启动时一次性加载 OpenPose、Human Parsing 和 OOTDiffusion，后续任务复用同一组模型实例，避免每次请求重新加载权重。仓库中的 Gradio 入口已经采用这种初始化方式。
- **面向预览的默认参数**：交互预览默认生成 1 张图片、使用 20 个推理步，并统一处理为 768 × 1024；用户需要时再提高步数或生成数量，在响应速度和结果质量之间做选择。
- **FP16 与无梯度推理**：OOTDiffusion pipeline 使用 FP16 权重，并在 `torch.no_grad()` 下执行，减少显存占用和不必要的训练计算。
- **GPU 任务串行化**：接口层将请求交给 GPU worker 按任务处理，限制同一张显卡上的并发推理，避免多个请求同时占用显存导致不稳定。
- **异步状态反馈**：小程序提交任务后立即进入等待状态，通过 `task_id` 查询 `processing / completed / failed`，而不是一直阻塞页面请求；完成后再加载 `result_url`。
- **失败可恢复**：上传失败、推理失败和结果过期分别对应明确状态，用户可以保留已经选择的图片并重新提交，不必从购物流程重新开始。

这套方案解决的是两类问题：模型常驻、FP16 和较轻的默认参数降低实际推理开销；任务队列与状态查询则降低用户对等待的感知，并避免网络超时直接打断试衣流程。项目没有虚构未经测量的“实时”延迟数据，而是通过服务边界和交互状态设计，让长耗时推理能够被小程序稳定承接。

### 技术栈

| 技术 | 在项目中的作用 |
| --- | --- |
| **微信小程序原生框架** | 承载商品浏览、虚拟试衣、购物车、订单和个人中心等用户流程 |
| **wx.chooseMedia / wx.uploadFile / wx.request** | 选择并上传试衣素材、创建推理任务、查询状态和获取结果 |
| **HTTP / JSON 接口设计** | 定义小程序与 Python 推理服务之间的图片、任务状态和结果契约 |
| **Python** | 组织模型侧预处理与推理入口 |
| **PyTorch / Diffusers** | 运行 OOTDiffusion 的扩散模型推理流程 |
| **OpenPose** | 提取人体姿态关键点，为试衣区域和人体结构提供条件 |
| **Human Parsing** | 区分人体、服装和背景区域，辅助生成试衣掩码 |
| **Pillow / OpenCV / NumPy** | 完成图片缩放、掩码处理、形态学操作和结果保存 |
| **Gradio** | 提供模型侧交互界面，用于快速验证人物图、服装图和推理参数 |
| **OOTDiffusion** | 提供半身与全身虚拟试衣模型及其推理 pipeline |

这里的技术栈并不是简单堆叠：OpenPose 和 Human Parsing 先把人物图片转换为模型需要的结构化条件，OOTDiffusion 负责生成，Gradio/CLI 用于验证模型侧输入输出，小程序则把这些能力组织成面向消费者的交互流程。

### 设计思路

#### 1. 把模型能力放回真实任务

虚拟试衣的价值不只在于生成图片，而在于帮助用户完成“这件衣服是否适合我”的判断。因此，项目没有把模型结果停留在单独的测试页面，而是把试衣入口、生成结果和商品浏览、购物车、订单等环节放在同一条用户路径中。

#### 2. 用产品语言封装模型约束

模型侧需要区分半身与全身模式，并在全身模式下进一步区分上装、下装和连衣裙。应用层将这些约束转换为明确的服装类别与操作步骤，使用户不需要理解模型名称、掩码或推理 pipeline。

#### 3. 分离应用体验与模型实现

购物页面负责用户交互和状态流转；姿态识别、人体解析和 OOTDiffusion 负责图像生成。这样的划分让模型可以独立验证，也便于应用层围绕上传、等待和结果展示继续迭代。

#### 4. 用异步任务承接长耗时推理

GPU 推理时间明显长于普通接口请求。设计中先返回任务 ID，再由小程序查询状态并更新等待页面，避免把页面生命周期与一次长时间 HTTP 请求绑定，也为错误提示和重新生成保留了清晰的状态边界。

### 仓库结构

```text
.
├── miniprogram (2)/miniprogram   # 微信小程序前端快照引用
├── tryon/1/
│   ├── ootd/                     # OOTDiffusion 推理与 pipeline
│   ├── preprocess/
│   │   ├── openpose/             # 人体姿态关键点
│   │   └── humanparsing/         # 人体语义解析
│   └── run/
│       ├── gradio_ootd.py        # Gradio 交互入口
│       ├── run_ootd.py           # 命令行推理入口
│       └── examples/             # 人物图与服装图示例
└── 演示视频.mp4                  # 小程序与试衣流程演示
```

### 模型侧运行

模型权重、Python 环境和依赖安装请参考 [OOTDiffusion 官方仓库](https://github.com/levihsu/OOTDiffusion)。准备完成后，可以使用仓库中的命令行入口验证试衣流程。

半身上装试衣：

```bash
cd tryon/1
python run/run_ootd.py \
  --model_path <person-image> \
  --cloth_path <garment-image> \
  --model_type hd \
  --category 0
```

全身/分类别试衣：

```bash
cd tryon/1
python run/run_ootd.py \
  --model_path <person-image> \
  --cloth_path <garment-image> \
  --model_type dc \
  --category 2
```

其中 `category` 的取值为：`0` 上装、`1` 下装、`2` 连衣裙。推理入口将人物图和服装图统一处理为 768 × 1024，并支持调整生成步数、引导尺度、样本数和随机种子。

### 模型来源与项目边界

[OOTDiffusion](https://github.com/levihsu/OOTDiffusion) 的模型架构、权重、推理 pipeline，以及仓库 `tryon/1/` 下的大部分模型与预处理代码来自其官方开源实现。相关研究与模型成果归原作者所有。

本项目的重点是基于这一开源能力完成购物场景设计、微信小程序体验组织、输入输出映射和端到端演示，不将上游模型描述为个人原创成果。

论文：

> Yuhao Xu, Tao Gu, Weifeng Chen, Chengcai Chen.  
> *OOTDiffusion: Outfitting Fusion based Latent Diffusion for Controllable Virtual Try-on.*

---

## English

### Overview

Let's Buy is a shopping prototype built around AI-powered virtual try-on. The project looks beyond generating a single outfit image and asks how virtual try-on can become part of a coherent consumer journey:

1. browse products and enter the try-on experience;
2. select a person image and a garment image;
3. choose a half-body or full-body mode based on the garment;
4. wait for pose estimation, human parsing, and diffusion inference;
5. review the generated preview and continue to product, cart, order, or account flows.

The project uses [OOTDiffusion](https://github.com/levihsu/OOTDiffusion) as its virtual try-on foundation. I independently implemented the mini-program pages and interaction flow, and validated the half-body and full-body model paths separately. The demo follows the designed end-to-end integration path from product discovery and image selection to try-on result presentation.

### Demo

The repository includes a [project demo video](演示视频.mp4) showing how virtual try-on fits into the shopping interface and the broader page flow.

The demo presents more than an isolated inference notebook: it follows the user from person and garment selection to a generated try-on result and back into the shopping journey.

> **Implementation scope:** I independently implemented the mini-program pages, shopping flow, virtual try-on interactions, and model-side validation. I also designed the HTTP contract between the mini program and the OOTDiffusion service. The delivered prototype validates the application and inference paths independently, while the demo presents the complete experience defined by that contract.

### My contribution

My work focused on turning an existing virtual try-on capability into a user-facing application prototype, rather than reimplementing the OOTDiffusion model itself.

- **Independent mini-program implementation:** implemented the home, product discovery, virtual try-on, cart, order, and account pages and their interaction flow.
- **Interaction modelling:** translated person images, garment images, try-on modes, garment categories, generation states, and outputs into a sequence of understandable user actions.
- **Integration design:** designed the image-upload, job-creation, status-query, and result-return contract around the half-body (HD) and full-body/category-aware (DC) OOTDiffusion capabilities.
- **Model-side validation:** used the upstream Gradio and command-line entry points to check different person, garment, category, and inference-parameter combinations.
- **End-to-end presentation:** prepared example assets and a demo that places model output in a consumer workflow instead of presenting it only as an offline generation result.

This work covers mini-program implementation, AI integration design, product-flow decomposition, and front-end/back-end boundary design: the model generates the image, while the application layer turns input collection, inference state, category constraints, and output presentation into a continuous experience.

### From shopping action to generated preview

```text
WeChat Mini Program
    │
    ├── Browse products / enter virtual try-on
    │
    ├── Select person image, garment image, and category
    │
    ▼
Try-on input
    │
    ├── OpenPose: body keypoints
    ├── Human Parsing: body and garment regions
    ├── Mask Preparation: region to be regenerated
    │
    ▼
OOTDiffusion
    │
    ├── HD: upper-body try-on
    └── DC: upper-body, lower-body, and dresses
    │
    ▼
Generated try-on preview
    │
    └── Return to product, cart, and order flows
```

Pose keypoints and human-parsing results determine which parts of the original image should be preserved or regenerated. OOTDiffusion produces the try-on result from the person, garment, and mask inputs, while the mini program owns the interaction and presentation flow.

### How the mini program calls OOTDiffusion

I designed the boundary between the mini program and model service as an asynchronous job API. GPU inference takes substantially longer than an ordinary page request, so the design separates asset upload, job creation, and result retrieval instead of keeping the mini program blocked on one long-running connection:

```text
wx.chooseMedia
    │  Select person and garment images
    ▼
wx.uploadFile
    │  Upload each image and receive person_asset_id / garment_asset_id
    ▼
POST /api/v1/try-on/jobs
    │  Submit asset IDs, try-on mode, and garment category
    ▼
Python inference service
    │  OpenPose → Human Parsing → Mask → OOTDiffusion
    ▼
GET /api/v1/try-on/jobs/{task_id}
    │  Poll job state with wx.request
    ▼
result_url
       Render the image and return to the shopping flow
```

The job request contains only product-facing information:

```json
{
  "person_asset_id": "person_xxx",
  "garment_asset_id": "garment_xxx",
  "mode": "dc",
  "category": "dress"
}
```

The service maps `mode=hd` to half-body upper-garment inference, while `mode=dc` combines with `upperbody`, `lowerbody`, or `dress` to select the category-aware path. Job creation returns a `task_id`; the mini program updates its waiting state from `processing / completed / failed` responses and renders the returned `result_url` when generation finishes.

This contract separates product interaction from model internals: the mini program does not need to understand pose estimation, human parsing, or mask construction, and the inference service does not depend on a particular page layout. The demo presents the target experience defined by this contract, while the checked-in Gradio/CLI paths validate model-side inputs and outputs independently.

#### Handling inference latency

Virtual try-on cannot respond like an ordinary page API. The design therefore addresses both actual compute time and the user's experience of waiting:

- **Resident model instances:** OpenPose, Human Parsing, and OOTDiffusion are loaded once when the service starts and reused across jobs, avoiding checkpoint loading on every request. The checked-in Gradio entry point already follows this initialisation pattern.
- **Preview-oriented defaults:** the interactive path defaults to one output at 20 inference steps and processes images at 768 × 1024. Users can request more steps or samples when they prefer quality or variation over response time.
- **FP16, inference-only execution:** the OOTDiffusion pipeline loads FP16 weights and runs under `torch.no_grad()`, reducing memory pressure and unnecessary training computation.
- **Serialised GPU work:** an API-layer GPU worker processes bounded jobs instead of allowing unrestricted concurrent inference on the same device.
- **Asynchronous feedback:** the mini program receives a `task_id` immediately and updates its UI from `processing / completed / failed` states instead of blocking one page request until generation finishes.
- **Recoverable failures:** upload, inference, and expired-result errors are represented separately, allowing the user to retry without selecting both images again or restarting the shopping flow.

Model reuse, FP16 execution, and preview defaults reduce the compute cost itself; job scheduling and explicit states make the remaining wait understandable and prevent a network timeout from breaking the try-on flow. The project does not claim an unmeasured “real-time” latency figure—instead, it shows how a mini program can reliably accommodate long-running GPU inference.

### Technology stack

| Technology | Role in the project |
| --- | --- |
| **Native WeChat Mini Program** | Product discovery, virtual try-on, cart, order, and account interactions |
| **wx.chooseMedia / wx.uploadFile / wx.request** | Image selection and upload, inference-job creation, status queries, and result retrieval |
| **HTTP / JSON interface design** | Contract for assets, job state, and generated results between the mini program and Python service |
| **Python** | Model-side preprocessing and inference entry points |
| **PyTorch / Diffusers** | Diffusion inference runtime used by OOTDiffusion |
| **OpenPose** | Extracts body keypoints used to preserve pose and body structure |
| **Human Parsing** | Separates body, garment, and background regions for mask preparation |
| **Pillow / OpenCV / NumPy** | Image resizing, mask processing, morphology, and result export |
| **Gradio** | Interactive model-side validation for images and inference parameters |
| **OOTDiffusion** | Upstream half-body and full-body virtual try-on model and pipelines |

The components form a deliberate pipeline rather than a list of tools: OpenPose and Human Parsing convert a user image into structural conditions, OOTDiffusion generates the result, Gradio/CLI provide a repeatable model-side test surface, and the mini program presents the capability as a shopping experience.

### Design decisions

#### 1. Place generation inside a real user task

The useful outcome of virtual try-on is not the image alone; it is helping a user decide whether a garment fits their intended look. The prototype therefore connects the try-on entry point and generated result with product discovery, cart, and order interactions.

#### 2. Translate model constraints into product language

The inference pipeline distinguishes half-body and full-body modes, and the full-body mode requires an upper-body, lower-body, or dress category. The application expresses those constraints as clear garment choices rather than exposing masks, pipeline names, or implementation details to the user.

#### 3. Keep the application and model layers separable

The shopping interface owns user actions and state transitions; pose estimation, human parsing, and OOTDiffusion own image generation. This boundary allows the model path to be tested independently while the application experience evolves around upload, waiting, and result states.

#### 4. Represent long-running inference as a job

GPU inference takes substantially longer than an ordinary API request. Returning a task ID and querying its state keeps the mini-program page lifecycle independent from one long-running HTTP connection, while creating explicit states for progress, failure, and regeneration.

### Repository guide

```text
.
├── miniprogram (2)/miniprogram   # WeChat Mini Program front-end snapshot reference
├── tryon/1/
│   ├── ootd/                     # OOTDiffusion inference and pipelines
│   ├── preprocess/
│   │   ├── openpose/             # Body-pose keypoints
│   │   └── humanparsing/         # Human semantic parsing
│   └── run/
│       ├── gradio_ootd.py        # Gradio entry point
│       ├── run_ootd.py           # Command-line inference
│       └── examples/             # Person and garment examples
└── 演示视频.mp4                  # Mini-program and try-on demo
```

### Running the model side

Follow the [official OOTDiffusion repository](https://github.com/levihsu/OOTDiffusion) for model checkpoints, Python environment, and dependency installation. Once prepared, the checked-in CLI can be used to validate the try-on path.

Half-body upper-garment try-on:

```bash
cd tryon/1
python run/run_ootd.py \
  --model_path <person-image> \
  --cloth_path <garment-image> \
  --model_type hd \
  --category 0
```

Full-body/category-aware try-on:

```bash
cd tryon/1
python run/run_ootd.py \
  --model_path <person-image> \
  --cloth_path <garment-image> \
  --model_type dc \
  --category 2
```

`category` uses `0` for upper-body garments, `1` for lower-body garments, and `2` for dresses. The runner resizes person and garment inputs to 768 × 1024 and exposes inference steps, guidance scale, sample count, and seed.

### Upstream attribution

The OOTDiffusion architecture, checkpoints, inference pipelines, and most of the model/preprocessing code under `tryon/1/` come from the [official OOTDiffusion implementation](https://github.com/levihsu/OOTDiffusion). The corresponding research and model work belong to its original authors.

This project focuses on applying that open-source capability to a shopping scenario through product-flow design, WeChat Mini Program experience design, input/output mapping, and end-to-end demonstration. It does not present the upstream model as original work.

Paper:

> Yuhao Xu, Tao Gu, Weifeng Chen, Chengcai Chen.  
> *OOTDiffusion: Outfitting Fusion based Latent Diffusion for Controllable Virtual Try-on.*

