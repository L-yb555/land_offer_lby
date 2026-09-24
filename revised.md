# 自动驾驶感知工程师简历优化与审查指南 (Resume Optimization for Autonomous Driving Perception Engineer)

## 1. 目标与定位 (Target & Positioning)
* **目标岗位**：自动驾驶感知工程师 (Autonomous Driving Perception Engineer)
* **核心优势**：
  1. 大厂智驾实习经验（CARIZON/地平线：数据闭环、VLM/Grounding DINO、Tagger挖掘、长尾 Corner Case 识别）。
  2. 算法与端侧部署能力（TensorRT、ONNX、Jetson AGX Orin、FP16/INT8 量化、CUDA 多机多卡分布式训练/SFT）。
  3. 扎实的科研与大模型/融合感知功底（1篇 IEEE Review 已中 + 1篇 JCR一区Top 在修 + 1篇系统闭环在投）。
* **优化方向**：突出智驾相关感知与数据闭环经验；强化学术/通用技能（如 Transformer/Conformer、多模态融合、ROS2）向智驾领域的迁移属性；补充量化成果（mAP、FPS、延迟 ms）；优化视觉布局与信息密度。

---

## 2. Agent 审查与修改规则 (Rules & Directives for Agent)

### 规则 1：突出主线与去弱化 (Domain Focus)
- **强关联模块**：将智驾实习（CARIZON）、端侧部署（优奇智能/TensorRT）、多模态大模型（Qwen-VL/SFT/CoT）置于视觉焦点。
- **抽象泛化处理**：项目中的 `sEMG`（肌电）、触觉等非智驾强相关词汇，提炼并强调其背后的**“多通道高频传感器同步”、“微秒级时间戳对齐”、“时序 Transformer/Conformer 融合”**等通用工程与算法能力。

### 规则 2：强制补全量化数据 (Quantitative Metrics Insertion)
- 所有模型训练、端侧部署与数据闭环工作，必须按照以下格式提示用户或自动填充量化占位符：
  - **端侧部署**：明确“推理延迟从 `XX ms` 降至 `XX ms`，FPS 提升至 `XX`，精度损失控制在 `X%` 以内”。
  - **数据挖掘**：明确“召回率/准确率提升 `XX%`”、“单标签样本规模 `XX`、标签准确率 `XX%`”、“人效 `XX` 标签/人/天”。

### 规则 3：单栏优雅排版架构 (Single-Column Structural Rules)
- **禁用多栏交错布局**：顶部 Header 集中展示“姓名、求职意向、电话、邮箱、GitHub、学历/院校”。
- **模块分割顺序**：
  1. 个人信息 (Header)
  2. 教育背景 (Education)
  3. 专业技能 (Technical Skills - 提炼分类关键词，避免长句段落)
  4. 实习经历 (Internship Experience - 智驾感知与部署优先)
  5. 科研与项目经历 (Projects & Research - 突出融合感知与 ROS2)
  6. 论文与研究成果 (Publications)
  7. 荣誉与综合素质 (Honors)

---

## 3. 标准简历 MarkDown 模版 (Standard Optimized Markdown)

```markdown
# 梁邦一
**求职意向：自动驾驶感知工程师**  
📱 (+86) 15515602693 | ✉️ yuejinai55@gmail.com | 🌐 [github.com/L-yb555](https://github.com/L-yb555)  
🎓 江南大学（控制科学与工程 硕士） | 英语：CET-6  

---

### 🎓 教育背景
* **江南大学** | 控制科学与工程 硕士（与中科院沈阳自动化所全国重点实验室联合培养） `2024.06 - 2027.06`
* **郑州轻工业大学** | 自动化 学士 `2020.09 - 2024.06`
* **荣誉奖项**：中国机器人大赛暨 RoboCup 机器人世界杯中国赛 国家三等奖 | 校一等奖学金（2025、2026） | 省/校优秀毕业生

---

### 🛠 专业技能
* **视觉感知与大模型**：精通 Grounding DINO 开放词汇检测、YOLO 系列（YOLOv8/11）目标检测与实例分割；具备多模态大模型（Qwen-VL 系列）全参数 SFT、CoT 混合后训练及 Bad Case 归因诊断能力。
* **模型部署与加速**：精通 NVIDIA Jetson/Orin 车端平台部署，熟练掌握 PyTorch → ONNX → TensorRT 模型转换、FP16/INT8 量化与算子优化，具备实时推理闭环工程实践。
* **数据闭环与数据挖掘**：具备自动化 Tagger 挖掘算子设计、标签质检能力，熟练掌握“实车——评估——归因——数据改进”感知数据迭代闭环。
* **工程与仿真栈**：精通 C++/Python、PyTorch、ROS2（Topic/Node）、MuJoCo 仿真开发；熟练使用 Linux、Git、Docker、CUDA 多机多卡分布式训练。

---

### 💼 实习经历

**CARIZON（大众×地平线合资智驾）** | 感知数据部门 算法实习生 `2026.06 - 至今`
* **道路视觉感知与细粒度属性识别**：基于 Grounding DINO + 专用 VLM（Qwen-VL）构建开放词汇检测与细粒度属性识别链路，结合视觉 Embedding 检索召回相似样本，构建“检测——属性理解——召回”闭环，直接支撑感知模型训练库建设。
* **长尾/Corner Case 数据挖掘**：
  * 融合车载多路传感器回灌数据设计规则挖掘算子，在 3 万条路口数据中挖掘红绿灯转向 Tagger 样本 3,000 条，验收准确率达 90%。
* **VLM 细粒度识别后训练**：负责感知细粒度识别模型优化，围绕闸机杆 / 限高杆五分类清洗与校验图像，基于 Qwen3-VL-8B + LLaMA-Factory 在 H20 8GPU 环境完成 13 轮 SFT 实验迭代与全参数 SFT，测试集准确率 0.8984 创历史新高。

**优奇智能科技有限公司（优必选子公司）** | 人形机器人感知算法实习生 `2026.02 - 2026.05`
* **Jetson AGX Orin 端侧部署与 TensorRT 推理加速**：
  * 主导 R11 机器人感知模型在 Jetson AGX Orin 上的端侧部署，完成 YOLO11s-seg 模型的 ONNX 导出、TensorRT 结构优化与 INT8/FP16 量化。
  * 实现了检测与分割实时推理链路，**推理延迟显著降低，FPS 大幅提升**，对齐精度与时延需求，打通端到端工程闭环。
* **感知模型训练与数据闭环**：基于 PyTorch/Ultralytics 构建多场景数据质检与训练管线，针对误检/漏检 bad case 进行归因分析，驱动感知模型的持续版本迭代。

---

### 🔬 科研与项目经历

**面向具身智能的多传感器融合感知与控制系统** | 项目负责人 `2025.02 - 2026.02`
* **多传感器时序融合与同步**：搭建视觉 + 多通道物理传感器实验平台，完成多通道数据高频采集、时间戳微秒级对齐与任务标注。
* **时序特征建模与 Sim-to-Real 部署**：基于 PyTorch 构建 Transformer/Conformer 时序融合模型；在 MuJoCo 中搭建仿真场景，通过 ROS2 模块化接口打通仿真与真机联调，构建 Sim-to-Real 闭环。

---

### 📄 论文与研究成果
1. **IEEE Sensors Journal (JCR一区, 已接收)**: *Liang B., et al. Sensing Technologies for Hand Gesture Recognition in Human-Robot Interaction: A Review. 2025.*（系统梳理多模态感知与传感器融合路线）
2. **Biomedical Signal Processing and Control (JCR一区Top, 在修)**: *SEMG Conformer: Synergizing Local-Global Spatiotemporal Features for Continuous Force Estimation.*（提出卷积局部+自注意力全局时序建模网络）
3. **Dexterous Hand Grasping Control... (JCR一区, 在投)**: 负责构建结合强化学习与多模态传感融合的分层控制与实时部署系统。