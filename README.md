# AMD AI 开发者问题反馈

欢迎 AMD GPU 上的 AI 开发者在此提交问题。我们持续建设开发者生态，希望把散落在各平台的零散反馈，收敛到这里统一跟踪和解决。

我们经常在 Reddit、知乎、B 站、小红书、Discord、微信群等渠道看到过类似留言——

> 「Radeon + llama.cpp + ROCm 跑 LLM 没 Vulkan 快」  
> 「Geneformer 官方示例全是 CUDA，AMD ROCm + Linux 能跑吗？」  
> 「AI Max 395 上 vLLM main 部署 Qwen3，某算子不支持，自动 fallback 到 CPU」

这类信息对我们很有价值，但**往往缺少复现所需的关键细节**，在评论区追问常常回复不全或等待很久。

所以**请大家在本仓库提 Issue**，按下方模板提供完整信息，我们才能更快定位、复现和修复。

---

## 如何提交

1. 点击仓库顶部的 **[Issues](https://github.com/AMD-DEVELOPERS/AI-issues/issues)** → **New issue**
2. 选择 **AMD AI 问题反馈** 模板并点击 **Get started**
3. 按下方 **Issue 标题规范** 填写标题
4. 按自动加载的模板逐项填写（不确定的字段请写「未测试 / 未知」，不要留空）
5. 附上 **错误日志** 和 **截图**（可直接拖拽到 Issue 编辑器）
6. 如有复现脚本或最小示例，请一并附上或提供 GitHub Gist / 仓库链接

---

## Issue 标题规范

请使用以下格式，便于检索和分类：

```
[框架名] [模型名] 简短问题描述 — 硬件型号
```

**示例：**

- `[vLLM] [Qwen3-8B] 某算子不支持导致 fallback CPU — AI Max 395`
- `[llama.cpp] [Llama-3-8B] ROCm 后端性能低于 Vulkan — Radeon RX 7900 XTX`
- `[PyTorch] [Geneformer] ROCm 环境下训练/推理报错 — Instinct MI300X`

---

## Issue 正文模板

提交 Issue 时，以下模板会自动加载，请逐项填写：

```markdown
## 问题概述
<!-- 用 1～3 句话描述你遇到的问题和期望行为 -->

## 环境信息

| 字段 | 内容 |
|------|------|
| **硬件** | <!-- 例：AMD AI Max 395 / Radeon RX 7900 XTX / Instinct MI300X --> |
| **CPU** | <!-- 例：AMD Ryzen 9 9950X --> |
| **内存** | <!-- 例：128 GB DDR5 --> |
| **操作系统** | <!-- 例：Ubuntu 22.04 / RHEL 9.4 --> |
| **ROCm 版本** | <!-- `rocminfo` 或 `hipconfig` 输出中的版本 --> |
| **驱动版本** | <!-- `rocm-smi` 或 ` amdgpu-pro --version` --> |

## 软件栈

| 字段 | 内容 |
|------|------|
| **推理/训练框架** | <!-- 例：vLLM / llama.cpp / PyTorch / TensorRT-LLM / SGLang --> |
| **框架版本** | <!-- 例：vLLM 0.6.3 / llama.cpp b4123 / PyTorch 2.4.0+rocm6.1 --> |
| **后端** | <!-- 例：ROCm / HIP / Vulkan / CPU --> |
| **相关依赖版本** | <!-- 例：transformers 4.44.0, flash-attn 2.6.0 --> |
| **安装方式** | <!-- 例：pip / docker / 源码编译 / 官方 wheel --> |

## 模型信息

| 字段 | 内容 |
|------|------|
| **模型名称** | <!-- 例：Qwen3-8B-Instruct / Geneformer / Llama-3-8B --> |
| **模型来源** | <!-- HuggingFace 链接或本地路径 --> |
| **量化方式** | <!-- 例：FP16 / BF16 / INT8 / AWQ / GGUF Q4_K_M / 无量化 --> |
| **模型规模** | <!-- 例：8B / 70B --> |

## 问题详情

### 复现步骤
<!-- 按顺序列出操作步骤，越具体越好 -->
1. 
2. 
3. 

### 实际行为
<!-- 描述实际发生了什么 -->

### 期望行为
<!-- 描述你期望发生什么 -->

### 相关算子 / API（如有）
<!-- 例：FlashAttention / paged_attention / custom_op::xxx / torch.compile -->
<!-- 如有报错中提到的 kernel 名、算子名、Python 函数名，请一并写出 -->

## 错误信息

### 完整错误日志
<!-- 请粘贴完整 traceback 或终端输出，不要只截一行 -->
```
在此粘贴日志
```

### 截图
<!-- 拖拽上传终端报错、rocm-smi、性能对比等截图 -->

## 复现脚本（强烈建议）

<!-- 提供最小可复现示例（MWE），可以是：-->
<!-- - 完整的 Python 脚本 -->
<!-- - Docker Compose / Dockerfile -->
<!-- - 启动命令 + 配置文件 -->
<!-- - GitHub 仓库或 Gist 链接 -->

```bash
# 启动命令示例
python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen3-8B ...
```

## 补充信息（可选）

- **对比环境**：如在 NVIDIA GPU 上是否正常？Vulkan 后端是否正常？
- **性能数据**：如有 benchmark 数字（tokens/s、latency），请附上
- **已尝试的 workaround**：换版本、换后端、降精度等
- **相关链接**：原帖链接、上游 Issue、文档链接
```

---

## 必填 vs 选填

### 必填（缺少则难以处理）

| 信息 | 说明 |
|------|------|
| 问题概述 | 一句话说清「什么坏了」 |
| 硬件型号 | 具体 AMD GPU 型号，不要只写「AMD 显卡」 |
| 操作系统 + ROCm/驱动版本 | 版本差异是多数问题的根因 |
| 框架名称 + 版本 | 含 commit hash 更佳（如 `vLLM main @ abc1234`） |
| 模型名称 + 来源 | HuggingFace 链接或确切 checkpoint 名 |
| 复现步骤 | 他人能按步骤重现 |
| 完整错误日志 | 完整 traceback，不要只截关键一行 |
| 后端 | ROCm / Vulkan / HIP / CPU 等 |

### 强烈建议

| 信息 | 说明 |
|------|------|
| 算子 / kernel 名称 | 报错或日志中出现的 op 名、kernel 名 |
| 错误截图 | 终端、监控面板、性能对比 |
| 最小复现脚本 | 单行命令或 10 行以内 Python 脚本 |
| 量化方式 | FP16/BF16/INT8/GGUF 等 |

### 可选但有助于加速

- 在 NVIDIA 或 Vulkan 上的对比结果  
- `rocminfo`、`rocm-smi` 输出  
- Docker 镜像名和 tag  
- 上游仓库已有 Issue 链接  

---

## 好 Issue vs 差 Issue

### ❌ 信息不足（难以处理）

> Radeon + llama.cpp + ROCm 跑流行 LLM 没 Vulkan 快

缺少：具体 GPU 型号、llama.cpp 版本、测试模型、量化格式、测试命令、性能数字、ROCm 版本。

> 我想跑 Geneformer，官方示例全是 CUDA，AMD ROCm 能实现吗？

缺少：目标场景（训练/推理/微调）、PyTorch 版本、具体报错、已尝试步骤。

> vLLM main 部署 Qwen3，算子不支持，fallback CPU

缺少：GPU 型号、vLLM commit、完整报错、复现命令、哪个算子。

### ✅ 信息完整（可快速跟进）

> **[vLLM] [Qwen3-8B] paged_attention_v2 算子不支持，fallback CPU — AI Max 395**
>
> - 硬件：AMD AI Max 395，128GB 统一内存  
> - 系统：Ubuntu 24.04，ROCm 6.2.0，驱动 6.2.0-66  
> - vLLM：`main` @ `a1b2c3d`（2025-08-20），pip 安装  
> - 模型：`Qwen/Qwen3-8B-Instruct`，BF16  
> - 命令：`python -m vllm.entrypoints.openai.api_server --model Qwen/Qwen3-8B-Instruct --dtype bfloat16`  
> - 报错：`RuntimeError: Operator 'paged_attention_v2' not implemented for ROCm`  
> - 附件：完整 traceback + rocm-smi 截图  

---

## 常见问题类型

提交时可参考以下标签（如仓库已启用 Labels）：

| 类型 | 说明 | 示例 |
|------|------|------|
| `bug` | 功能异常、崩溃、fallback | 算子不支持、OOM、Segfault |
| `performance` | 能跑但性能不达预期 | ROCm 比 Vulkan 慢、吞吐低 |
| `compatibility` | 框架/模型在 AMD 上不可用 | 仅支持 CUDA 的项目 |
| `documentation` | 缺少 AMD 平台文档或示例 | 官方 README 无 ROCm 说明 |
| `feature-request` | 希望支持某模型/算子/后端 | 请求 vLLM 支持某 custom op |

---

## 提交前自查

- [ ] 标题包含框架、模型、硬件
- [ ] 硬件型号具体（非「AMD GPU」）
- [ ] 框架版本可复现（版本号或 commit hash）
- [ ] 模型有明确来源（HF 链接或 checkpoint 名）
- [ ] 复现步骤他人可逐步执行
- [ ] 错误日志完整（非单行摘要）
- [ ] 如有算子名/kernel 名已填写
- [ ] 已搜索 [已有 Issues](https://github.com/AMD-DEVELOPERS/AI-issues/issues)，避免重复

---

## 我们会如何响应

1. **确认收到**：通常在 2～3 个工作日内回复  
2. **信息补全**：若关键字段缺失，我们会留言请补充  
3. **复现与定位**：信息完整的问题会进入复现队列  
4. **进展同步**：修复、workaround 或上游跟进会在 Issue 中更新  

---

## 其他渠道

本仓库是 **AMD AI 问题的官方汇总入口**。你也可以在其他社区讨论，但请**同时或最终在本仓库提 Issue**，以便跟踪和避免信息丢失。

---

## 技术文章译文

下面是提交到本仓库的 AMD 技术文章中文译文，方便中文读者查阅。复现时请以原文的命令和数字为准。

- [发布当天，EAGLE-4 就能在 AMD Instinct GPU 上跑](articles/eagle-4-on-amd-instinct-gpus.md)（2026-10-20，[原文](https://www.amd.com/en/developer/resources/technical-articles/2026/eagle-4-on-amd-instinct-gpus.html)）

---

## License

本仓库用于问题收集与跟踪，Issue 内容遵循 GitHub 默认条款。提交 Issue 即表示你同意问题描述可能被用于内部复现与修复。
