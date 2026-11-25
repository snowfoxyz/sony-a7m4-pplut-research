# sony-a7m4-pplut-research

对 Sony A7M4 6.02 PPLUT 与 Log Shooting 行为的只读研究记录。

[English](README.md) | **中文**

## 当前状态

```text
A7M4 官方 6.02 没有 PPLUT1–4
A7C2 官方支持电影模式 PPLUT1–4
A7M4 仍有 PPLUT 标签、renderer、sequence_set_lut_data、SetLutData
A7M4 缺 UI/model LUT 成员和 0x4154–0x4159 handler
sequence_set_lut_data 注册为 type ID 0x1BC，
但动态 sequence loader/dispatcher 尚未从静态固件中闭合
```

单改产品/菜单门控只能让 PPLUT 出现，但不能真正使用。当前最需要的是
A7C2/FX2 正样本运行时证据，或可回滚的调试固件观察。

## 文档

- [研究状态（2026-11-22）](RESEARCH-STATUS-2026-11-22.md)
- [静态分析摘要（2026-11-22）](docs/STATIC-ANALYSIS-2026-11-22.md)
- **[2026-11-25 更新：发现、sequence 注册表、未解问题](docs/2026-11-25/README.md)**
- [贡献 / 证据请求](CONTRIBUTING.md)

## 范围：照片 JSON LUT vs 视频 PPLUT/CUBE LUT

本仓库研究的是 **视频模式 PPLUT1–4 / CUBE LUT**，不是 Erlla 的照片向
Custom LUTS 实现。

```text
Erlla Custom LUTS：
  自定义菜单
  camera-look v2 原生 JSON 参数
  照片向颜色路径
  第三方 appFw + CPAPP 修改

本仓库研究：
  Movie Picture Profile 中的 Sony PPLUT1–4
  CUBE/用户 LUT 进入 PPLUT
  官方 provider/candidate/commit/sequence 绑定
```

Erlla 的自定义菜单即使出现在视频菜单里，也不等于实现了 PPLUT。

## 范围与安全

- 仅做静态、只读分析。
- 不包含固件镜像、分区数据、密钥、设备标识或刷机步骤。
- 文中的地址和键值只是研究线索，不是可直接应用的补丁。
- 未进行相机 I/O、属性写入、固件修改或刷机。

## 许可证

MIT — 见 [LICENSE](LICENSE)。
