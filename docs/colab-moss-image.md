# Google Colab + ComfyUI + MOSS-Image（免费 T4 实测）

核实日期：2026-11-14
环境：Google Colab 免费版 Tesla T4（15360 MiB）、ComfyUI 0.45.0、PyTorch 2.14.1+cu130、Python 3.13.16
教程来源：X 上 @huang18825 的置顶教程《Google Colab 部署 ComfyUI + MOSS-Image 完整实战教程》

## 结论

免费 T4 可以跑通 MOSS-Image 文生图。实测 1024×1024、27 步、约 166 秒出一张图，无爆显存。

## 关键坑（都是实测踩出来的）

1. **提示词增强器（TextGenerate 节点）在 T4 上必须关闭。**
   开启时报错：`Expected query, key, and value to have the same dtype, but got query.dtype: BFloat16 key.dtype: float`。
   原因是 T4 没有原生 BF16 支持，增强器的生成代码路径里 dtype 写死不一致。
2. **加 `--force-fp16` 启动参数不能修复上面的报错。**
   实测：重启后模型确实以 float16 加载，但增强器报同样的错。正确做法是关掉模板里的 `enable_prompt_enhancer` 开关，主体管线即可正常出图。
3. 权重约 12.2 GB（Comfy-Org/MOSS-Image 三件套），免费环境下载约 4 分钟；免费会话会过期，隧道地址随会话变化。

## 成功参数（实测）

- 提示词：`a cat sitting on a windowsill, sunlight, photorealistic`
- 1024×1024，27 steps，er_sde，cfg 2.5，约 5.08 s/it，总耗时 166.65 s
- 显存峰值在 T4 的 14913 MiB 可用范围内，无 OOM

## 安全提醒

- 截图分享 Colab 界面时，注意不要露出 ngrok Authtoken 等令牌行；泄露后应到对应后台重新生成。
