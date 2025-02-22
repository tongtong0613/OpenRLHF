# OpenRLHF x Ascend

我们在 OpenRLHF 上增加对华为昇腾设备的支持，在华为昇腾设备上使用 OpenRLHF 与在英伟达 GPU 上使用几乎相同。

## 硬件支持

* Atlas 800T A2

## 安装

### 环境准备

| 软件      | 版本        |
| --------- | ----------- |
| Python    | >= 3.10     |
| torch     | == 2.5.1    |
| torch_npu | >= 2.5.1rc1 |
| CANN      | >= 8.1.RC1 (**Not Released**)   |

> 当前版本在 CANN 8.1.RC1 上进行测试，该版本通常预计在三月底正式发布。  
> 为了保证能够正常使用 vLLM，我们建议上述配套软件的安装遵循 vllm-ascend 的[安装教程](https://vllm-ascend.readthedocs.io/en/v0.7.1rc1/installation.html)。

### 源码安装

```shell
git clone https://github.com/zhuo97/OpenRLHF.git
cd OpenRLHF
TARGET_DEVICE=NPU pip install -e .
```

### vLLM

为了保证能够在 OpenRLHF 上正常使用 vLLM，需要安装 vLLM Ascend 插件（`vllm-ascend`）。关于在华为昇腾上支持的 vLLM 版本以及和 vLLM Ascend 的配套关系请参考[安装教程](https://vllm-ascend.readthedocs.io/en/v0.7.1rc1/installation.html)。

> hybrid engine 功能依赖 vLLM >= 0.7.2，当前 vllm-ascend 不支持该版本，该功能尚未支持。

### Ray

可通过如下方式在华为昇腾设备上启动 Ray:
```shell
# launch the master node of ray in container
ray start --head --node-ip-address 0.0.0.0
```

训练脚本提交方式与英伟达 GPU 相同。

### 其他第三方库说明

| 软件            | 说明             |
| --------------- | ---------------- |
| flash_attn      | 不支持           |
| ring_flash_attn | 不支持           |
| bitsandbytes    | 部分支持，待验证 |

## 支持的算法

### 精度对比

根据经验，我们期望在相同配置下，在华为昇腾设备上的 Loss 与英伟达 GPU 的 Loss 平均误差小于 2%，具体计算方式如下：

![loss_comparison](./images/loss_comparison.png)

其中，N 表示训练的步数。更多信息请参考[精度计算说明](https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/LMaccuracy_0001.html)。

### 进展

| 算法                   | 进展                                                                                                         | 与GPU误差 | 详细结果                                                     |
| ---------------------- |------------------------------------------------------------------------------------------------------------|--------| ------------------------------------------------------------ |
| SFT                    | 已支持                                                                                                        | 0.19%  | [测试结果](https://github.com/OpenRLHF/OpenRLHF/pull/605#issuecomment-2567488539) |
| DPO                    | 功能正常，已有初步验证结果，具体见[测试结果](https://github.com/OpenRLHF/OpenRLHF/pull/605#issuecomment-2567488539)，当前基于默认配置测试中 | 1.81%  | [测试结果](https://github.com/OpenRLHF/OpenRLHF/pull/605#issuecomment-2567488539) |
| IPO                    | 即将开展                                                                                                       |        |                                                              |
| cDPO                   | 即将开展                                                                                                       |        |                                                              |
| KTO                    | 已支持                                                                                                        | 0.37%  | [测试结果](https://github.com/OpenRLHF/OpenRLHF/pull/605#issuecomment-2642104300) |
| RM                     | 已支持                                                                                                        | 0.85%  | [测试结果](https://github.com/OpenRLHF/OpenRLHF/pull/605#issuecomment-2642104300) |
| PRM                    | 已支持                                                                                                        | 1.61%  | [测试结果](https://github.com/OpenRLHF/OpenRLHF/pull/605#issuecomment-2642104300) |
| Iterative DPO          | 即将开展                                                                                                       |        |                                                              |
| PPO                    | 测试中                                                                                                        |        |                                                              |
| REINFORCE++            | 基于 `train_reinforce_llama_ray.sh` 功能打通，精度测试中                                                               |        |                                                              |
| RLOO                   | 即将开展                                                                                                       |        |                                                              |
| REINFORCE Baseline     | 即将开展                                                                                                       |        |                                                              |
| Rejection  Sampling    | 即将开展                                                                                                       |        |                                                              |
| Knowledge Distillation | 即将开展                                                                                                       |        |                                                              |
| Conditional SFT        | 即将开展                                                                                                       |        |                                                              |
| Continue Pretrain      | 即将开展                                                                                                       |        |                                                              |

> 补充说明：
>
> 1. 由于不支持 flash_attn 第三方库，在脚本中需要删除 --flash_attn 参数。
> 2. 当前使用 --adam_offload 参数可能存在卡死情况，待解决。
