> **Update (2026-11-25):** a newer evidence layer is in
> [docs/2026-11-25/README.md](docs/2026-11-25/README.md).
> The 2026-11-22 text below is retained as historical context.

# Sony A7M4 6.02 PPLUT / Log Shooting 研究状态

**状态日期：** 2026-11-22  
**范围：** 对 Sony ILCE-7M4 6.02 固件进行只读分析，重点关注 PPLUT1–4 是否能出现在视频 Picture Profile 选择菜单，以及 Log Shooting 的软件门控路径。

## 结论摘要

固件中存在 PPLUT1–4 的名称，以及共用的 Picture Profile LUT 显示逻辑。另一条机型差异路径中，也存在一个带条件判断、涉及 Picture Profile 相关键值的查询。但目前证据**尚未把这两条路径与视频 Picture Profile 主菜单的候选项列表连起来**。尚未找到可靠补丁点，也没有验证过的固件修改方案。

## 已从官方 6.02 应用镜像确认

- Picture Profile 值注册表共有 16 项；序号 12–15 对应 PPLUT1–4 名称。
- A7M4 应用镜像中存在共用的 Picture Profile 显示渲染逻辑。这只能证明固件编译了相关显示能力，不能证明 PPLUT1–4 可在视频 PP 菜单中选择。
- 主 `view_list_pictureprofile` 控件从内部缓存读取候选值并发送选择事件。在已检查的构造、刷新和选择路径范围内，尚未找到写入该缓存的代码。
- 一条生成式机型差异路径会先检查条件 `0x02CF0E95`，再调用通用解析器并传入裸键值 `0x1185`。仅凭低位数值相同，无法确定该裸键所属的命名空间或语义；目前也没有证据证明这条路径会影响 PP 候选项生成或菜单行显隐。
- A7M4、A7C2 与 FX2 的对比显示，PP 列表类结构和 PPLUT 显示映射有共同特征；但这不能证明它们使用相同的候选项提供者或具有相同行为。

## 证据与边界

| 判断 | 证据 | 目前不能据此证明 |
|---|---|---|
| 固件中有 PPLUT1–4 名称 | 静态检查值表和名称指针 | 不能证明菜单会显示或允许选择 |
| 固件中有 PP/LUT 显示代码 | 静态检查显示路径和数据流 | 不能证明候选项生成或 LUT 实际生效 |
| 机型差异条件可门控一次通用解析器调用 | 静态检查指令和条件树 | 裸键值的命名空间、运行时注册情况及其 UI 影响仍未确定 |
| 主 PP 菜单使用缓存候选列表 | 静态检查菜单控件 | 缓存写入者和键值提交路径仍未找到 |
| 软件或产品分类门控是一个合理方向 | 公开机型资料与编译后的条件路径 | 这仍是待验证假设，并非已证明的因果关系 |

## 外部项目交叉核对

- [leogenot/sony-a7iv-feature-research](https://github.com/leogenot/sony-a7iv-feature-research) 记录了与 Log Shooting 和 Picture Profile 相关的 A7M4 产品分类及机型差异排除信息。这支持继续调查软件门控，但没有追踪到主 PP 候选缓存。
- [chengong-art/a7m4-lut-log-shooting](https://github.com/chengong-art/a7m4-lut-log-shooting) 发布了一个离线 LUT 初始化候选方案。仓库自身将其标记为研究用途，并说明原生菜单行为、CP 对 MLUT 的消费以及成像管线输出仍未验证。其启动阶段或 LUT 修改不能独立证明 PPLUT 菜单行可用。
- [16480000/sony-a7iv-qemu](https://github.com/16480000/sony-a7iv-qemu) 提供基于 QEMU 的原始 UI 研究环境。仓库没有记录 PPLUT 或 Log Shooting 菜单验证；现有证据覆盖部分 UI 路径，尚未实现完整的原始取景器、自动对焦、ISP 与 JPEG 链路。

## 当前判断

软件菜单或产品分类门控仍然是合理假设。现有证据既不能证明存在硬件限制，也不能排除硬件限制。候选项生成器、单项有效性规则，以及用户选中后的提交路径仍未查明。根据当前证据，不建议制作或应用补丁。

## 研究安全与复现范围

- 分析仅对本地持有的 6.02 材料进行静态只读检查；本文不包含固件镜像、分区数据、设备标识或写入步骤。
- 未进行相机 I/O、属性写入、固件或升级包修改，也未刷写相机。
- 文中的地址和键值只是研究线索，不是可直接应用的补丁说明。

## 后续需要取得的证据

1. 找到主 PP 候选缓存的运行时写入者或提供者。
2. 追踪机型差异条件结果是否传递到候选提供者或单项有效性逻辑。
3. 追踪 PP 选择事件到实际属性设置器，并确认键值所属命名空间。
4. 只有完成上述调用链验证后，才考虑提出可回滚且经过独立核对的补丁方案。



# Sony A7M4 6.02 PPLUT / Log Shooting Research Status

**Status date:** 2026-11-22  
**Scope:** Read-only analysis of Sony ILCE-7M4 firmware 6.02, focused on PPLUT1–4 availability in the Movie Picture Profile picker and the Log Shooting software gate.

## Executive result

The firmware contains PPLUT1–4 labels and shared Picture Profile LUT-display logic. A separate model-difference path also contains a condition-gated query involving a Picture Profile-related key. However, the evidence does **not** yet connect either path to the main Movie Picture Profile menu's candidate list. No reliable patch point or verified firmware modification has been identified.

## Verified from the official 6.02 application image

- The Picture Profile value registry has 16 entries; ordinals 12–15 resolve to PPLUT1–4 labels.
- A shared PP display renderer is present in the A7M4 application image. This confirms compiled display capability, not that PPLUT1–4 are selectable in the Movie PP menu.
- The main `view_list_pictureprofile` widget reads candidate values from an internal cache and emits a selection event. Its bounded constructor, refresh, and selection paths do not identify a writer for that cache.
- A generated model-difference path checks condition `0x02CF0E95` before invoking a generic resolver with bare key `0x1185`. The meaning/namespace of that bare key is not established by identical low bits alone. This path has not been connected to PP candidate generation or row visibility.
- The A7M4/A7C2/FX2 comparison shows a conserved PP-list class shape and PPLUT display mapping. It does not establish identical candidate providers or behavior across models.

## Evidence classification

| Claim | Evidence | Limit |
|---|---|---|
| PPLUT1–4 labels exist | Static value-table and label-pointer inspection | Does not prove menu exposure |
| PP/LUT display code exists | Static call/data-flow inspection | Does not prove candidate generation or LUT application |
| A model-difference condition can gate a resolver call | Static instruction and condition-tree inspection | Bare key namespace, runtime registration, and UI effect remain unresolved |
| Main PP menu uses a cached candidate list | Static widget inspection | Cache writer and key-commit path remain unresolved |
| A software/product-class gate is plausible | Public model metadata and compiled condition path | Hypothesis, not a proven causal edge |

## External project cross-checks

- [leogenot/sony-a7iv-feature-research](https://github.com/leogenot/sony-a7iv-feature-research) documents A7M4 product-class/model-difference exclusions relevant to Log Shooting and Picture Profile. This supports investigating a software gate, but does not trace the rule to the main PP candidate cache.
- [chengong-art/a7m4-lut-log-shooting](https://github.com/chengong-art/a7m4-lut-log-shooting) publishes an offline LUT-initialization candidate. Its own status marks it research-only and leaves natural menu behavior, CP MLUT consumption, and image-pipeline output unverified. Its app startup/LUT edits do not independently validate PPLUT menu rows.
- [16480000/sony-a7iv-qemu](https://github.com/16480000/sony-a7iv-qemu) contains a QEMU-based original UI research environment. It does not document PPLUT or Log Shooting menu verification; its current evidence covers selected UI paths, while the full original viewfinder/AF/ISP/JPEG chain is not implemented.

## Current conclusion

A software menu/product gate remains plausible. A hardware limitation is neither established nor excluded. The candidate generator, per-row availability rule, and selected-value commit path are still unresolved. No patch is recommended from the current evidence.

## Research safety and reproducibility

- Analysis was static and read-only against locally held 6.02 artifacts; this status contains no firmware image, partition data, device identifier, or write procedure.
- No camera I/O, property write, firmware/package modification, or flashing was performed.
- Addresses and key values above are research leads, not a ready-to-apply patch specification.

## Next evidence needed

1. Locate the runtime writer/provider for the main PP candidate cache.
2. Trace whether the condition/model-difference result reaches that provider or per-row validity logic.
3. Trace the PP selection event to the actual property setter and confirm the key namespace.
4. Only then consider a reversible, independently checked patch proposal.
