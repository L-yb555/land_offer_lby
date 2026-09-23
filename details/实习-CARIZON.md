# 工作总结：自动驾驶感知 — VLM 微调 · 数据挖掘打标 · 检测蒸馏

> 公司：CARIZON（大众 × 地平线合资智驾）｜感知数据部门 算法实习生
> 作者：intern.bangyi.liang
> 总结时间：2026-09-17
> 实习时间：2026.06-至今（本总结覆盖 2026 年 7 月 – 9 月）
>
> 本文件为实习详情版，简历投递口径见 [`../baseline.md`](../baseline.md)。

---

## 一、背景

### 1.1 业务背景

自动驾驶感知系统需要大量高质量的场景数据来支撑模型训练与验证。团队面临几个核心问题：

- **数据获取效率低**：自动驾驶场景长尾数据（闸机杆、限高杆、垃圾袋、卷帘门、后备箱开启等）在路采数据中占比小，靠人工检索和标注成本极高，需要自动化的数据挖掘（mining）流水线，从海量 clip 中按语义条件筛选出目标场景。
- **标注瓶颈**：挖掘出的数据仍需人工标注才能用于训练。传统的纯 label（分类标签）标注信息量有限，模型在细粒度分类任务上精度遇到瓶颈。
- **检测模型落地成本高**：GroundingDINO 这类 VLM 检测模型精度好但推理成本高，无法直接大规模部署到生产打标流水线，需要蒸馏到轻量模型（YOLOv8）。

### 1.2 我在其中的位置

我负责的方向是打通 **"数据挖掘/搜索 → 人工 CoT 标注平台 → SFT/CoT 训练 → 评测迭代"** 的完整闭环，具体承担三块工作：

1. 在 mining engine 框架下开发多个生产 tagger（文搜+VLM 组合打标器）；
2. 搭建 GroundingDINO → YOLOv8 的伪标签蒸馏管道；
3. 以"闸机杆五分类"为试点，系统化迭代 VLM SFT 方案（LoRA/全参/CoT 混合），并把人工 CoT 标注工具落地到 GT 生产平台。

---

## 二、做了哪些工作

### 2.1 闸机杆分类 VLM SFT —— 核心主线（9 月，v1→v10 共 13 轮实验）

围绕"闸机杆/限高杆五分类"任务，在 LLaMA-Factory 上完成了系统化的 13 轮训练实验迭代（`LLaMA-Factory/examples/` 下 13 个自定义 YAML，`models_test/gatepole_eval/` 完整评测流水线）：

| 版本 | 基座 | 方法 | 关键变化 |
|------|------|------|----------|
| v1 | Qwen3-VL-8B (workcondition_merged) | LoRA r=32 | 初版闸机杆分类 |
| v2 | 同上 | LoRA r=32 | prompt 调整 |
| v3 | 同上 (fixsamples merged) | **全参微调** | 切换全参路线 |
| v4 | 同上 | 全参 | 图片分辨率 262144→1048576 像素 |
| v5 | 同上 | 全参 | 扩充数据（combined v2） |
| v6 | 同上 | 全参 | **解冻 vision_tower + projector** |
| v7/v8 | 同上 | LoRA r=256 | 大 rank LoRA 对比 |
| v9 | **Qwen3.5-9B** | LoRA r=32 | 换基座 + **300 条手写 grounded CoT** |
| v10 | Qwen3.5-9B | 全参 | **300 CoT + 1783 label 混合双 prompt** |

其中 v9 的 CoT 数据生产流程：图片筛选（五类配额 70/70/40/60/60，`selected_300.json`）→ 20 条校准 CoT → 300 条全量 draft → QA 审核 → final 300。

### 2.2 标注平台工具链（9/15，gt_production_platform）

一天内在 GT 生产平台落地一套完整的 CoT 标注系统（8 个 commit），打通"人工标注 → SFT 训练数据"闭环：

- **sft_prefill 导入**：从 `turnstile_gate_pole_v9_sft.json`（LLaMA-Factory sharegpt 格式）按 image basename 预填 label/cot 字段；
- **标注工作台**：label 单选 + thinking 推理链标注，含前端同步 SFT 数据入口；
- **sync_sft_json 回写**：人工确认后手动批量回写 sft.json；
- 输出设计文档与实施计划（`docs/superpowers/specs|plans/2026-09-15-think300-*`）。

关键代码：`backend/dvadmin/datapro/annotation/`（models / services / views / serializers / config / exporters）。

### 2.3 数据挖掘 tagger 开发（data_mining_agent，7 月至今）

在 mining_engine 框架下开发多个 **by_search_tagger（文搜+VLM 组合打标器）**，累计 15+ commits：

- **TollGate 闸机杆**：v0.0.2 → v0.0.3（当前分支 `feature/tollgate_by_search_tagger_v003`）；
- **GarbageBag 垃圾袋**：v0.0.1 → v0.0.2，另开发生产版 `garbage_bag_detection_tagger`（Qwen3-VL-8B，garbage_bag/none/uncertain 三分类，案例 PRO-53817）；
- **RollingDoor 卷帘门、TrunkOpen 后备箱开启**：新增；
- **限高杆系列**：HeightLimiter_by_dino（v001→v002）、LimitGateRed、LimitGateBlackYellow；
- 配套帧提取工具：`extract_camera_front_frames.py`、`extract_tag_frames_csv.py`。

### 2.4 GroundingDINO → YOLOv8 伪标签蒸馏管道（8 月，gd_dino_Distillation，从零自建）

- 用 **autodistill 框架**做伪标签蒸馏：GroundingDINO（教师，uni_det_v4_hdflow checkpoint）在无标注图片上预测框，作为 YOLOv8（学生）的 ground-truth 标签并训练；
- 主入口 `distill.py` 支持 `--label-only` / `--train-only` 分离步骤；
- 自写 GdDino 推理封装（批量推理、可视化）、10 类 ontology（锥桶/水马/AFrame 等，含 prompt/标签漂移一致性校验）、逐类 NMS/box 阈值配置；
- 自建 Docker 镜像（llamafactory baked 基础 + vLLM + autodistill，py3.10 编译 `_C.so` CUDA kernel），参数化 `build.sh`；
- 后续应用：**红绿灯检测**（`models_test/detect_traffic_light.py`，逐帧检测 + 按 clip 分段聚合判定）。

### 2.5 其他工作

- **垃圾袋检测多模型验证**（models_test）：Kimi-VL-A3B / Qwen3-VL-32B / Qwen3-VL-8B / Gemma4-26B few-shot 正负例对比测试，支持 hf/vllm/sglang 三种推理后端；
- **one_caption_model**：Qwen2-VL vLLM 结构化打标引擎（工况+车位 7 维度联合打标），含 GPU 自动选择、并行预处理流水线、分层可视化，README 持续维护至 9/17；
- **LLamaboard 实验**（8/10）：Qwen3-4B-Instruct-2507 LoRA 微调完整训练（1860 步、20 个 checkpoint）；
- **帧提取数据集**：100 个 clip 提取 67 张前视帧 + 535 张搜索结果帧（8 月）。

---

## 三、用什么做成的（技术栈与方法）

| 层面 | 技术 |
|------|------|
| 数据挖掘框架 | 自研 mining_engine（clip 级处理单元、tagger 执行框架、AIDI worker） |
| 打标方案 | by_search_tagger = 文搜（语义检索召回）+ VLM（Qwen3-VL / GroundingDINO 精判）组合 |
| VLM 微调 | LLaMA-Factory（LoRA / 全参 SFT，Qwen3-VL-8B、Qwen3.5-9B、Gemma4-26B 多基座对比） |
| CoT 数据工程 | 手写 grounded CoT（校准→draft→QA 审核→final）、双 prompt 训练（CoT + 短 label） |
| 检测蒸馏 | autodistill 伪标签蒸馏：GroundingDINO（教师）→ YOLOv8（学生） |
| 标注平台 | Django + DRF 后端 + Vue3 前端（GT 生产平台 annotation 模块） |
| 推理后端 | vLLM / sglang / transformers（huggingface）三套可切换 |
| 基础设施 | 自建 Docker 镜像链（llamafactory baked + vLLM + autodistill）、K8s GPU 作业提交、H20 8GPU 集群 |
| 数据存储 | /horizon-bucket 共享桶（carizon_eval_jfs / carizon_perception_jfs5） |

---

## 四、结果怎么样

### 4.1 闸机杆分类（核心量化成果）

- **v10 测试集准确率 0.8984（128 口径），历史新高**，超越此前最好的 v4（0.8906）；
- 验证了 **双 prompt 方案成立：CoT prompt 比短 prompt 高约 +3pp**——300 条手写 CoT + 1783 条 label 混合训练，显著优于纯 label 训练；
- 方法论上完成了从 LoRA→全参、小图→大图、单基座→Qwen3.5、纯 label→CoT 混合的完整消进路线，为后续其他场景的 SFT 提供了可复用的实验结论。

### 4.2 标注平台工具链

- 打通"人工 CoT 标注 → sharegpt 格式 SFT 数据"的自动化闭环，人工确认即可批量回写训练数据；
- 该工具链直接支撑了 v9/v10 的 CoT 数据生产，可复用到其他分类场景。

### 4.3 数据挖掘 tagger

- 多个 tagger 进入生产版本迭代（TollGate v0.0.3、GarbageBag v0.0.2），覆盖闸机杆、垃圾袋、卷帘门、后备箱、限高杆等场景；
- 垃圾袋检测经多模型（Kimi-VL / Qwen3-VL-32B/8B / Gemma4）few-shot 验证后落成生产 tagger（Qwen3-VL-8B 三分类）。

### 4.4 检测蒸馏

- 建立 GroundingDINO→YOLOv8 蒸馏管道，从零完成推理封装、ontology、阈值调参、Docker 镜像全链路；
- 已应用于红绿灯检测等实际任务，并作为检测类打标的通用基础设施。

---

## 五、时间线

| 时间 | 主线 |
|------|------|
| 7 月 | data_mining_agent tagger 开发起步（GarbageBag 等） |
| 8 月上旬 | GDINO 蒸馏管道 + Docker 镜像 + LLamaboard 实验 |
| 8 月中下旬 | 帧提取、红绿灯检测、垃圾袋多模型验证 |
| 9 月上旬 | 闸机杆 SFT v3–v6 评测迭代 |
| 9 月 14–15 日 | v9 CoT 路线（300 条手写 CoT）+ 标注平台工具链 |
| 9 月 17 日 | **v10 上线：准确率 0.8984 创历史新高** |

---

## 六、关键产出速查

| 类型 | 路径 |
|------|------|
| v10 报告 | `models_test/gatepole_eval/v10_report.md` |
| v10 数据/训练/评测 | `models_test/gatepole_eval/build_v10_dataset.py`、`run_v10_train.sh`、`eval_testset_v10_short.py` |
| 训练配置（13 个） | `LLaMA-Factory/examples/qwen3vl_sft_turnstile_gate_pole_v*.yaml` 等 |
| 标注服务 | `gt_production_platform/backend/dvadmin/datapro/annotation/` |
| 标注设计文档 | `gt_production_platform/docs/superpowers/specs/2026-09-15-think300-jsonl-prefill-annotation-design.md` |
| 蒸馏入口 | `gd_dino_Distillation/distill.py`、`utils/`、`docker/build.sh` |
| 生产 tagger | `data_mining_agent/mining_engine/search_taggers/taggers/`、`mining_engine/taggers/garbage_bag_detection_tagger/` |
| 帧提取工具 | `data_mining_agent/mining_engine/search_taggers/extract_camera_front_frames.py` |

---

# 附篇：感知挖掘模型与工程链路梳理

> 来源：`../补充实习经历/感知数据挖掘实习经历总结.md`（由 8 张「感知挖掘模型研究整理」文档截图整理）
> 性质：**补充素材稿**，部分内容来自截图 OCR，正式写入简历前请结合原代码/文档核对（见文末「待核实项」）。

## 七、平台定位与两条挖掘路线

感知数据挖掘平台是自动驾驶数据闭环中的一环。上游对接车载摄像头采集的图像/视频数据，下游输出 taggers / 标签，最终用于：

- 感知数据自动化标注
- Corner Case / 边缘场景挖掘
- 数据筛选和检索
- 标签库建设
- 后续模型训练、评估与数据闭环迭代

两条并列的挖掘技术路线：

- **图搜**：基于特征相似度检索，更偏相似样本召回。
- **VLM 刷库**：基于视觉语言模型的语义理解，对图片或视频帧进行语义判断、属性识别和结构化打标，更偏语义理解和开放场景判断。

## 八、核心模型与框架

### 1. One-Caption-Model：结构化打标模型

基于 **Qwen2-VL** 微调的结构化打标模型，核心流程：

```text
读取批量图片 -> 调用 Qwen2-VL -> 生成 caption / JSON 结构化标签
```

位于 Public-VLM 与 Custom-VLM 之间：相比通用 Public-VLM，它经过 fine-tuning，更适合输出稳定的固定标签；相比 GDINO 路线，它不依赖目标检测框的初步筛查，可直接对整张驾驶场景图像输出结构化结果。

可识别的标签属性：`scene_classification`（场景）、`illumination_classification`（光照）、`time_classification`（时间）、`weather_classification`（天气）、`Parkingspot`（车位）。

批量推理脚本的大致逻辑：

```text
run_caption_vlm()
  -> 设置 current_time / task_name
  -> 设置 batch_size、debug、visualize
  -> 指定 image_dir 和 output_dir
  -> 调用 demo/qwen2vl_vlm.py 一类脚本
  -> submit_task 提交任务
```

### 2. GDINO / Grounding DINO：开放词汇目标检测

根据文本目标在图像中定位对应对象，输出检测框、置信度和图像尺寸。适合处理“有没有某类目标”“目标数量是否足够”“目标框在哪里”这类问题。

可识别的支持类别：traffic sign、traffic light、vehicle、person、cone、tripod、loader、bus。

### 3. GDINO + VLM：两阶段目标挖掘流程

体现“专用模型优先、通用模型兜底”的 pipeline：

```text
图片 / 视频抽帧
  -> 判断挖掘目标是否在支持标签中
  -> 如果目标在 GDINO 支持类别内，选择 GDINO 模型检测
  -> GDINO 输出检测框和结果文件
  -> 判断目标是否能被 GDINO 精确支持
  -> 如果需要细粒度属性，裁剪 bbox 内目标图
  -> 对裁剪图调用 Custom-VLM，使用固定 prompt 识别属性
  -> 如果目标不在专用支持范围内，调用 Public-VLM 做开放域描述
  -> 后处理提取最终挖掘目标和标签
```

设计逻辑：能用 GDINO / Custom-VLM 解决的任务优先用专用模型（提高效率和准确率）；GDINO 负责目标定位，Custom-VLM 负责 bbox 内细粒度属性识别；Public-VLM 作为兜底，覆盖长尾、开放域或未被支持类别覆盖的场景；最终通过后处理把模型输出转换成可入库、可检索、可用于数据闭环的标签。

## 九、mining-system 工程调用链

### 1. tagger 与模型映射关系

没有一张集中式「注册表」直接说明某个 tagger 用哪个模型，映射关系分散在多个文件中，需要沿链路排查：

```text
config.yaml
  -> 定义本次任务跑哪些 tagger 和版本

taggers/<name>/*.yaml
  -> 通过 use_* 字段说明使用哪类模型

base_tagger.py
  -> 提供 dino_inference / qwen2_inference / one_caption_model_inference 等推理接口

processer.py
  -> load_model，加载实际模型 checkpoint / wrapper
```

### 2. DINO 推理调用链

从 `tagging_main.py` 到 DINO 模型真正 forward 的完整链路：

```text
tagging_main.py
  -> TaskFactory.get_task('clip')(task_info); task.run()

task_manager/task_manager.py
  -> TaskFactory.get_task -> ClipTaggingTask

task_manager/tagging_task.py
  -> TaggingTask.run()
  -> run_batch()
  -> run_vehicle_day_tagger()
  -> single_run() / multi_run()
  -> run_one_task()

base_tagger.py
  -> BaseTagger.run()
  -> self.process(...)

objs_by_dino_tagger_v011.py
  -> process(...)
  -> self.dino_inference(...)

base_tagger.py
  -> dino_inference
  -> RabbitCli().batch_inference('dino', 'v6.0', ...)

utils/model_processer.py
  -> push('dino', msg) 进入 RabbitMQ 队列

processer.py
  -> dino worker
  -> GdDino.batch_inference(imgs)
  -> 模型真正 forward
```

核心理解：tagger 插件在自己的 `process()` 方法里调用推理接口；`tagging_main` 经过多层任务调度后才进入具体模型推理；DINO 推理通过 RabbitMQ 队列进入对应 worker，真正的模型 forward 发生在 worker 内。

## 十、模型类型与输出格式

| 模型 | 用途 | 输出形式 |
| :--- | :--- | :--- |
| `dino` / Grounding DINO | 目标存在性判断、数量判断、明确目标挖掘（锥桶、水马、柱子、交通参与者） | `{ts: {class_name: {boxes: [...], scores: [...]}}, img_width, img_height}` |
| `model` / `qwen2` | VLM 自由文本描述或问答，tagger 自己写 prompt 并解析生成文本 | 自由文本，需后处理解析 |
| `qwen3` / `qwen3_imgs` | Qwen3-VL 单图理解；`qwen3_imgs` 支持双图输入（参考图 + 目标图验证推理） | 文本 |
| `one_caption_model` | 固定 label 集合的结构化 JSON，适合场景/光照/天气/时间/车位等固定分类 | 结构化 JSON |
| `water_horse_dino` | 专门检测「注水水马」等目标 | 检测框 |
| `tsr_gdino` + `tsr_qwen2` | 交通场景检测 + 逐框属性标注，两阶段输出检测框和属性 | 检测框 + 属性 |

qwen2 prompt 示例方向：

```text
请描述图中路面上除车以外的物体，请列出它们的类型和颜色、形状。
```

**qwen2 与 one_caption_model 的区别**：qwen2 输出自由文本，需要后处理解析；one_caption_model 是微调过的结构化模型，直接输出固定标签，更适合自动化入库和评估。

## 十一、开源微调框架与评估流程

团队使用的开源微调框架主要有 **LLaMA-Factory** 与 **xtuner**。

### 1. LLaMA-Factory 能力

持续预训练、监督微调 SFT、奖励建模、PPO / DPO / KTO / ORPO、全参数微调、冻结层微调、LoRA / QLoRA、GaLore / BAdam / Adam-mini / DoRA / LongLoRA / LLaMA Pro / PiSSA 等算法；OpenAI 风格 API、Gradio UI、CLI、vLLM 加速推理；LlamaBoard、TensorBoard、Wandb、MLflow、SwanLab 实验监控。

### 2. Qwen2-VL 驾驶场景分类微调

微调目标是让 **Qwen2-VL-2B** 成为驾驶场景图像分类器：

```text
输入：一张驾驶场景图片 + prompt 问题
输出：JSON 格式的分类结果
```

评估脚本工作：解析模型输出 JSON 的 `predict` 字段 → 与标注 JSON 的 `label` 字段对比 → 计算 Precision、Recall、F1-score → 生成混淆矩阵图。

可识别的训练配置：

```yaml
learning_rate: 1.0e-5
num_train_epochs: 20
lr_scheduler_type: cosine
warmup_ratio: 0.05
bf16: true
ddp_timeout: 180000000
```

## 十二、可写进简历的三版表达

> 三版差异在于「参与程度」的表述强度，按真实参与边界取舍。

### 版本 A：智驾数据挖掘方向

感知数据挖掘与 VLM 结构化打标实习经历
`自动驾驶数据闭环 / Qwen2-VL / Grounding DINO / RabbitMQ / LLaMA-Factory`

1. 参与感知数据挖掘平台相关模型链路梳理，围绕 `one_caption_model`、`dino/gdino`、`gdino_vlm` 等框架整理批量图像打标、开放词汇目标检测、VLM 语义理解和两阶段目标挖掘流程，支撑自动驾驶图像/视频数据的标签生成、数据筛选与 Corner Case 挖掘。
2. 梳理基于 Qwen2-VL 的结构化打标模型 One-Caption-Model，明确批量图片输入、Prompt 调用、JSON 结构化标签输出和下游评估流程，覆盖场景、光照、时间、天气、车位属性等多类驾驶场景标签。
3. 分析 Grounding DINO 在开放词汇目标检测中的应用，整理检测框、置信度、图像尺寸等输出格式，并结合 Custom-VLM / Public-VLM 形成“专用检测优先、通用 VLM 兜底”的目标挖掘 pipeline。
4. 梳理 `mining-system` 中从 `config.yaml`、tagger 配置、`base_tagger.py` 推理接口到 `processer.py` 模型 worker 的调用链路，明确 tagger 与模型类型、模型版本、实际 checkpoint 的映射关系，提高复杂挖掘任务的排查和交接效率。
5. 调研 LLaMA-Factory 与 xtuner 等多模态微调框架，理解 Qwen2-VL-2B 在驾驶场景分类任务中的 SFT 配置、JSON 输出评估、Precision / Recall / F1-score 计算和混淆矩阵分析流程。

### 版本 B：感知模型场景理解方向

感知模型场景理解与自动化标注实习经历
`VLM 场景理解 / 结构化打标 / 开放词汇检测 / 多模型协同`

1. 围绕自动驾驶场景图像理解任务，梳理 Qwen2-VL 结构化打标、Grounding DINO 开放词汇检测和 Public-VLM 开放域描述在感知数据挖掘中的协同方式，理解从原始图像/视频帧到可入库标签的自动化处理链路。
2. 参与 One-Caption-Model 结构化标签体系整理，关注场景分类、光照、时间、天气和车位属性等固定标签的生成与评估，理解微调 VLM 相比自由文本 VLM 在标签稳定性和下游解析上的优势。
3. 梳理 GDINO + VLM 两阶段 pipeline：先通过开放词汇检测定位目标，再对 bbox 裁剪图调用 Custom-VLM 识别属性；对未覆盖的长尾目标使用 Public-VLM 兜底描述，并通过后处理提取目标标签。
4. 对比 dino、qwen2、qwen3、qwen3_imgs、one_caption_model、water_horse_dino、tsr_gdino + tsr_qwen2 等模型的输出格式和适用场景，沉淀模型选型与 tagger 解析规则。
5. 基于 LLaMA-Factory 微调和评估流程，理解多模态分类模型从训练配置、推理输出、标注对比到指标计算的闭环，为后续感知模型迭代和数据质量分析提供基础。

### 版本 C：更保守的实习补充写法

感知数据挖掘模型调研与工程链路梳理
`Qwen2-VL / Grounding DINO / VLM 刷库 / 自动化标注`

1. 系统整理感知数据挖掘平台中的多模型方案，包括 Qwen2-VL 结构化打标、Grounding DINO 目标检测、GDINO + VLM 两阶段目标挖掘和 Public-VLM 开放域描述，形成面向团队交接的技术梳理文档。
2. 梳理 `mining-system` 中 tagger 到模型推理的工程调用链路，明确配置文件、tagger 版本、推理接口、RabbitMQ 队列和模型 worker 的关系，提升复杂任务排查效率。
3. 总结不同模型输出格式和下游解析方式，包括检测框/置信度、自由文本描述、双图 VLM 推理、结构化 JSON 标签等，为后续标签库建设、数据筛选和模型评估提供参考。
4. 调研 LLaMA-Factory / xtuner 等开源微调框架，整理 Qwen2-VL 驾驶场景分类任务的训练配置、JSON 预测解析、Precision / Recall / F1-score 和混淆矩阵评估流程。

## 十三、面试表达素材

**问「这段实习具体做了什么」**：

> 我主要参与的是感知数据挖掘平台中多模型链路的梳理和结构化打标流程理解。平台上游接入车载图像/视频数据，下游通过不同 tagger 输出标签，用于自动化标注、Corner Case 挖掘和标签库建设。我重点梳理了三类模型：一类是基于 Qwen2-VL 微调的 One-Caption-Model，用于直接输出场景、光照、天气等结构化 JSON 标签；一类是 Grounding DINO，用于开放词汇目标检测和 bbox 定位；还有一类是 GDINO + VLM 的两阶段流程，先检测目标，再对裁剪图调用 Custom-VLM 或 Public-VLM 做属性识别和兜底描述。同时我也梳理了 mining-system 中从 config、tagger、base_tagger 推理接口到 RabbitMQ worker 的调用链，帮助理解某个 tagger 最终调用哪个模型，以及模型输出如何被下游解析。

**问「这和普通调研有什么区别」**：

- 不是只看模型论文，而是结合工程仓库梳理配置、tagger、推理接口和 worker 的真实调用关系。
- 不是只关注单模型效果，而是理解多模型协同：GDINO 负责定位，Custom-VLM 负责细粒度属性，Public-VLM 负责开放域兜底。
- 不是只做文本总结，而是沉淀了可用于交接、排查和后续数据闭环的模型输出格式与调用链说明。

**问「和自动驾驶数据闭环的关系」**：

> 这套系统的核心价值是把海量采集数据转化成可检索、可筛选、可训练的结构化标签。通过 VLM 刷库和开放词汇检测，可以更快发现长尾场景和模型薄弱样本，再将这些数据用于标注、训练和评估，从而支撑自动驾驶感知模型的数据闭环迭代。

## 十四、关键词

自动驾驶数据闭环、感知数据挖掘、VLM 刷库、结构化打标、Qwen2-VL、One-Caption-Model、Grounding DINO / GDINO、开放词汇目标检测、Custom-VLM / Public-VLM、Prompt 设计、JSON 标签输出、tagger 解析、RabbitMQ 异步推理、LLaMA-Factory、xtuner、SFT、LoRA / QLoRA、Precision / Recall / F1-score、混淆矩阵、Corner Case 挖掘、标签库建设。

## 十五、待核实项 ⚠️

以下信息来自截图 OCR 或局部可见内容，**正式写入简历前建议结合原代码/文档再确认**：

1. 具体模型 checkpoint 路径。
2. 部分 tagger 文件名和版本号，例如 `objs_by_dino_tagger_v011.py`、`animal_by_model_tagger_v002.yaml` 等。
3. 具体任务表名、数据库表名和项目内部路径。
4. `tsr_gdino + tsr_qwen2` 的类别数量、阶段命名和实际输出字段。
5. **参与边界**：调研梳理、工程排查、脚本开发、模型微调、评估脚本使用，或多项都有参与 —— 后续按真实参与程度取舍版本 A/B/C。
6. ~~**`one_caption_model` 基座口径冲突**~~ —— **已确认并修正**：`one_caption_model` 是 **Qwen2-VL** 系列（Qwen2-VL-2B 用于驾驶场景分类；Qwen2-VL vLLM 用于工况+车位 7 维打标）。`../baseline.md` 中「泊车障碍物识别模型」原先误挂了 `OneCaptionModel` 标签，实际基座为 **Qwen3-VL-8B**，两者是不同组件，已拆开表述。
