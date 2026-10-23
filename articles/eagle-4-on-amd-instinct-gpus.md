## 背景：EAGLE-4 发布当天能在 AMD   GPU 上跑

- 日期：2026 年 10 月 20 日
- EAGLE 团队最新的草稿模型 EAGLE-4，从发布当天起就可以在 AMD   GPU 上运行，SGLang 和 ATOM 都已支持。以 Qwen3.8-27B、BF16 为例，单张 GPU 处理一条请求时，吞吐大约是 300 token/s，约为普通解码的 3.5 倍，也是该模型自带 MTP（多 token 预测）的 1.6 倍。并发从 1 路增加到 32 路，这个提升仍然保持；扩到四张 GPU，也同样成立。用法和其他硬件一样：同一份权重，同一套启动参数。
<img width="1200" height="600" alt="image" src="https://github.com/user-attachments/assets/4cc7066d-59b2-4107-a506-615dd66ccf19" />
                                                    图 1：

## EAGLE-4 新在哪里

推测解码把生成拆成两步：让一个小模型先往前猜几个 token，再由大模型用一次前向把这些猜测全部验证完。猜对的留下，当作正式输出；猜错的从第一个错处截断，后面改由大模型自己写。MTP 是 Qwen 模型自己带的多 token 预测头，猜得比较短。EAGLE-4 是单独的一层草稿模型，单请求时会用一棵又宽又深的猜测树，所以同一次核验里能留下的正确 token 更多。EAGLE-4 在原有 EAGLE 系列上加了三件事。

**同一套草稿模型，适配不同的 batch 大小和 GPU。** EAGLE-4 每次只起草一个 token，所以每一步都可以停下来，把猜测树加深或加宽。EAGLE-Harness 在启动时测一次：在当前这台机器上，起草和验证各要花多少开销。服务过程中，它会选单位开销下接受 token 最多的那棵树。只有一个用户时，用又宽又深的树；负载高了，就改成短链，不再把树展开。

**草稿模型的输出头更省。** 以前草稿模型的大部分开销都花在词表投影上。现在先用低秩头从完整词表里筛出候选，再用完整的头只给这些候选打分。草稿开销能降低 40% 以上，词表也不用截断。

**草稿模型很小。** 它只有一层 Transformer，放在主模型旁边只占很少显存，GPU 和端侧设备都能放下。

## 在 AMD   GPU 上的首日支持

在 GPU 上，EAGLE-4 使用的权重和启动参数与其他平台相同。

为了让它在发布当天就跑得够快，我们做了三件事：

1. 在 GPU 上测量 EAGLE-Harness 的开销表，让树的大小符合这张卡的实际开销。
2. 给 Qwen3.8 的线性注意力层（Gated DeltaNet）加上树形验证。整棵猜测树一次前向就能查完，只保留被接受那条分支的状态。
3. 核对结果是否无损。EAGLE-4 没有改大模型的答案。我们拿推测解码的输出和普通解码对过，每个输出 token 都与普通解码一致；只有两个候选的得分几乎打平时，才可能因为 BF16 舍入而不同。

### 结果

测试模型是 Qwen3.8-27B，权重为 BF16，硬件是单张 GPU。基准是 SPEED-Bench，也就是 EAGLE-4 论文里的那套测试https://github.com/eagle-hq/EAGLE-4/blob/main
表中数字的单位是 token/s。

| 并发对话数 | 1 | 4 | 16 | 32 |
|---|---:|---:|---:|---:|
| 普通解码（SGLang） | 82 | 292 | 872 | 1,285 |
| 自带 MTP（SGLang） | 178 | 617 | 1,743 | 2,551 |
| EAGLE-4，SGLang | 289 | 941 | 2,167 | 2,784 |
| EAGLE-4，ATOM | 309 | 976 | 2,309 | 3,243 |
<img width="1200" height="600" alt="image" src="https://github.com/user-attachments/assets/23f7e62d-9e73-4f61-8098-08d2a09b5c98" />
                                                    图 2：EAGLE-4 随负载的变化

- 单请求时，EAGLE-4 每验证一次平均大约能接受 4.7 个 token，自带 MTP 平均大约是 2.9 个。前面的 1.6 倍就来自这里。
- 负载升高、GPU 逐渐占满之后，EAGLE-4 会把猜测缩短，所以到 32 路并发时仍然快于 MTP。
- 使用张量并行时，单请求在两张 GPU 上（ATOM）达到 403 token/s，在四张上达到 463–489 token/s（SGLang / ATOM）。

### 四张 GPU
四张 GPU 做张量并行，基准与上面相同。单位仍是 token/s。
<img width="1200" height="600" alt="image" src="https://github.com/user-attachments/assets/73d40aa3-b69e-4d16-a304-63e2df03e660" />
                                                    图 3：四卡上的 EAGLE-4

| 并发对话数 | 1 | 4 | 16 | 32 |
|---|---:|---:|---:|---:|
| 普通解码（SGLang） | 156 | 539 | 1,655 | 2,331 |
| 自带 MTP（SGLang） | 303 | 1,036 | 2,711 | 3,916 |
| EAGLE-4，SGLang | 463 | 1,360 | 3,339 | 4,474 |
| EAGLE-4，ATOM | 489 | 1,347 | 3,381 | 4,796 |
用 SGLang 时，四张 GPU 上的单条请求，EAGLE-4 是普通解码的 3.0 倍，是 MTP 的 1.5 倍；到 32 路并发，仍分别是 1.9 倍和 1.14 倍。

### 目标

希望把 EAGLE-4 用到更多模型上，并把它和单请求 megakernel 合在一起，把延迟再降下来https://rocm.blogs.amd.com/software-tools-optimization/megakernel-intro/README.html?_gl=1*p6clqv*_gcl_au*NjI2Nzc3NjkwLjE3ODYwOTA2OTkuLS4tLjE3ODkxMTM1NjcuMTQ1MzYzMzYwOC4xNzkyNzM2MDI5LjE3OTI3MzYxNTQ.*_ga*MzcxMDczMTU3LjE3NzgzMDkxMzU.*_ga_5KTY6V8M8J*czE3OTI3NDExNDckbzIwMiRnMCR0MTc5Mjc0MTE0NyRqNjAkbDAkaDA.

### 测试配置

- **硬件：** AMD GPU。张量并行度为 1。软件版本是 AMD ROCm™ 7.2.4。
- **模型：** `Qwen/Qwen3.8-27B`，BF16。草稿权重是 Qwen3.8-27B 对应的 EAGLE-4 权重https://huggingface.co/EAGLE-Corp/Qwen3.8-27B-EAGLE4
- **基准：** SPEED-Bench 的 qualitative 子集。每个类别每隔 10 段对话取一段，一共 88 段，按轮次回放。开启思考，temperature 为 0。token/s 等于生成的 token 数除以墙钟时间(指一条回答的生成 token 数除以它跑完的秒数。)。
- **引擎：** SGLang 用的是 EAGLE-4 fork 的 main，提交 `2af5570`，并把 ROCm 的 PR rebase 到这个提交上。ATOM 用的是 `proaditya/ATOM` 的 `eagle4_day0_support` 分支，基线是 ATOM main 的 `2479251d3`，镜像是 `rocm/atom-dev:nightly_202610131505`（ROCm 7.2.4，包含 aiter 和 FlyDSL）。表里的每一档并发，SGLang 都使用 EAGLE-Harness 选出来的树。ATOM 在 16 路及以下按 batch 大小使用固定的树，到 32 路改为链。
eagle4_day0_support:https://github.com/proaditya/ATOM/tree/eagle4_day0_support
### 怎样复现这些数字

| | SGLang | ATOM |
|---|---|---|
| EAGLE-4，Qwen3.8-27B | 已提供ROCm PR:https://github.com/eagle-hq/sglang | 已提供（`eagle4_day0_support`） |

两个引擎用的是同一套 SPEED-Bench 客户端和数据，都在 EAGLE-4 仓库里：`inference/scripts/client_speedbench.py`，以及 `inference/dataset/speed_bench/question_multiturn.jsonl`。ATOM 的客户端从响应里的 `usage` 字段读取 token 数。客户端按轮次回放每段对话（开启思考，reasoning effort 为 `xhigh`，temperature 为 0）。文中报告的是 `wall_tput`，也就是生成 token 数除以墙钟时间。`--every 10` 就是表里那 88 段对话；去掉这个参数，则跑全部 880 段。

**ATOM。** 一条命令按配方里的参数把每个并发档都跑一遍，再跑客户端，并把结果和原文的数字并排打出来。README 里写了可选项（TP 大小、并发数），配方则给出这些选项对应的具体参数。使用前把 `EAGLE4` 设成 EAGLE-4 仓库路径，把 `DRAFT` 设成草稿权重路径：

```bash
EAGLE4= DRAFT= bash blog_benchmark/eagle4/build_and_repro.sh
```
Readme:https://github.com/proaditya/ATOM/blob/eagle4_day0_support/blog_benchmark/eagle4/README.md
recipe:https://github.com/proaditya/ATOM/blob/eagle4_day0_support/recipes/Qwen3.8-27B-EAGLE-4.md
script:https://github.com/proaditya/ATOM/blob/eagle4_day0_support/blog_benchmark/eagle4/build_and_repro.sh

**SGLang。** 用的是 EAGLE-4 仓库里的 SGLang，跑在面向 GPU 的 ROCm SGLang 镜像上。使用前需要准备这些变量：`$EAGLE4` 是 EAGLE-4 仓库路径，`$MODEL` 是主模型路径，`$DRAFT` 是草稿权重路径，`$COST_ROOT` 是开销表输出目录，`$C` 是并发对话数。
SGLang（用的是 EAGLE-4 仓库里的 SGLang- https://github.com/eagle-hq/sglang   ，跑在ROCm SGLang 镜像上）：

```bash
export PYTHONPATH=$EAGLE4/sglang-v0.5.20/python
export FLYDSL_RUNTIME_CACHE_DIR=/tmp/flydsl_cache     # writable; aiter compiles new GEMM shapes here
# 1. Once per GPU and TP size: measure EAGLE-Harness's cost tables (about 10-20 minutes)
cd $EAGLE4/inference/scripts
python engine_cost.py --base-model-path $MODEL --fast-model-path $DRAFT \
    --draft-tree-path $COST_ROOT --environment-str qwen38-eagle4-GPU-tp1 \
    --speculative-model EAGLE4 --compute-intensity high_intensity --model-data-type bfloat16 \
    --attention-backend triton --mm-attention-backend triton_attn --page-size 1 --mamba-ssm-dtype bfloat16 \
    --speculative-draft-attention-backend fa3 --speculative-draft-model-quantization unquant \
    --tp-size 1 --start-bs 1 --max-draft-batch-size 32 --context-step 256 --context-stop 32768 \
    --enable-cuda-graph --enable-fast-measurement --mem-fraction-static 0.8
# 2. Serve (C = concurrent conversations)
SGLANG_GDN_TREE_RECORDS=1 SGLANG_GDN_TREE_CHUNK=1 SGLANG_GDN_TREE_CHUNK_FLYDSL=1 \
python -m sglang.launch_server --model-path $MODEL --tp-size 1 --dtype bfloat16 --trust-remote-code \
    --mem-fraction-static 0.80 --context-length 36864 --chunked-prefill-size 16384 \
    --attention-backend triton --mm-attention-backend triton_attn --page-size 1 --mamba-ssm-dtype bfloat16 \
    --disable-radix-cache --reasoning-parser qwen3 --random-seed 721 \
    --max-running-requests $C --cuda-graph-max-bs-decode $C \
    --disable-overlap-schedule --disable-cuda-graph-padding --disable-prefill-cuda-graph --cuda-graph-backend-decode full \
    --speculative-algorithm EAGLE4 --speculative-draft-model-path $DRAFT --speculative-draft-model-quantization unquant \
    --speculative-draft-attention-backend fa3 --speculative-attention-mode prefill --speculative-exec-mode 0 \
    --speculative-num-steps 12 --speculative-eagle-topk 4 --speculative-num-draft-tokens 16 \
    --speculative-compute-intensity high_intensity --speculative-data-path $COST_ROOT/qwen38-eagle4-GPU-tp1 --port 30000
```

三个 `SGLANG_GDN_TREE_*` 开关用来打开线性注意力层的 FlyDSL 树形验证，测开销表时也要同样打开。TP 大于 1 时，两条命令都改成 `--tp-size N`，并且每个 TP 大小各测一张开销表。普通解码就是同一条启动命令，去掉全部 `--speculative-*` 参数。自带 MTP 则改为：

```bash
--speculative-algorithm EAGLE --speculative-draft-model-path $MODEL --speculative-num-steps 3 --speculative-eagle-topk 1 --speculative-num-draft-tokens 4
```
