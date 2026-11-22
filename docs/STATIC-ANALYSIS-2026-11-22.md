# A7M4 6.02 PPLUT / Log Shooting 静态分析状态

**日期：** 2026-11-22  
**范围：** Sony ILCE-7M4（A7M4）官方 6.02 `appFw.so` 的只读静态分析  
**结论：** 未找到可靠补丁点；没有修改固件，也没有刷机。

---

## 1. 结论摘要

- 官方 6.02 固件中 PPLUT1–4 的名称、显示逻辑、PP 值表都存在。
- `view_list_pictureprofile` 的候选缓存位于 `+0x2258`，数量在 `+0x22B4`。
- 该缓存的**写入者/provider 尚未找到**，因此无法建立：
  `候选 provider → 行有效性 → PP 选择提交` 的证据链。
- `0x1185` 经核实不是产品门控，而是 **PP 参数 getter 的 resolver key**。
- 没有提出补丁，没有改固件，没有设备写入。
- 相机保持官方 6.02。

---

## 2. 分析对象与校验

### 2.1 官方 AP

```text
文件：components-official/appFw.so
SHA-256：1e1b9202eaa69198f20df03dbfdab2f103b206cc4ebcd2e7bd817e6a710018a2
```

### 2.2 对照材料

| 材料 | 用途 |
|---|---|
| A7C2 2.01 `appFw.so` / `CPAPP-B.BIN` | 官方支持 PPLUT 的对照 |
| FX2 `appFw.so` / `CPAPP-B.BIN` | 同代 Cinema 功能对照 |
| v0.1.2 自定义 AP/CP | 有 PPLUT 菜单但灰显的对照 |
| `feature-manager-all-products-602.json` | 43 项 × 产品类矩阵 |

### 2.3 只读审计

7 个只读审计脚本全部重跑通过，覆盖：

- PP 选择 `+0x1B8` → `0x03010000` → class 6 事件
- 刷新 `+0x1C8` → `0x0201000C` → `0x183FDAC -> 0x183FD30`，与选择共用队列但不是 class 6
- `view_list_pictureprofile` vtable/refresh mode gate
- BKID `0x1185` 身份纠正
- 条件树根引用 census
- `0x1185` seed/recompute 生命周期
- App 事件队列 `0x5503E0` 刷新边界
- VPP 父子刷新与主列表候选边界

---

## 3. `view_list_pictureprofile` 构造链

### 3.1 构造函数

```text
0x18A0E50   view_list_pictureprofile 构造函数
```

构造函数从 ctor param 读取：

| ctor param | 写入 widget | 用途 |
|---|---|---|
| `+0x00` | `+0x1B8` | 选择/事件回调 |
| `+0x08` | `+0x1C0` | 第二个回调 |
| `+0x10` | `+0x1C8` | refresh context 对象 |
| `+0x20` | `+0x1D0` | ID/标志 |
| `+0x28` | `+0x1D4` | ID/标志 |

构造函数还初始化：

- `+0x260` 起的分段对象
- `+0x22C8`、`+0x27E8` 子对象
- 候选缓存 `+0x2258`
- 候选计数 `+0x22B4 = 0`

### 3.2 唯一调用者

```text
0x1866F78   s_allocate_list_pictureprofile(...)
```

符号字符串：

```text
i_view_widget* s_allocate_list_pictureprofile(
    uint32_t, char*, uint32_t&,
    const ds_view_widget_ctor_param&,
    const view_common_param_t&)
```

它是 **widget 分配器/工厂**，不是候选 provider。

### 3.3 通用构建器

```text
0x186A51C   通用 view 构建器
0x584E090   view 类 vtable（slot 3 = 0x186A51C）
```

构建器按 **widget type ID** 查表：

```text
allocator 表基址：0x50CD1E8
每项：8 字节
s_allocate_list_pictureprofile 表项：0x50CD308
type ID：36
```

定义记录：

```text
stride：0x34
+0x00  type ID
+0x04  flags
+0x08/+0x0C/+0x10/+0x14  布局
+0x18  ID/标志
+0x26/+0x28/+0x2C/+0x2E/+0x30  短整型
```

### 3.4 父对象回调

`0x1869E30` 写父对象：

```text
+0x3EA8 = x5
+0x3EB0 = x6
+0x3EB8 = x7
+0x3EC0 = stack arg
```

构建器把这些字段填入 ctor param，构造函数再写入 widget：

```text
+0x1B8
+0x1C0
+0x1C8
```

### 3.5 回调来源是 App 接口

全局槽：

```text
0x756E678
0x756E680
0x756E688
0x756E690
```

由 `0x1869940` 初始化，来源是：

```text
0x584DFC8   App 接口对象（.data.rel.ro）
```

字段：

```text
+0x10 -> 0x183FC9C   选择事件回调（+0x1B8，0x03010000，30 字节，class 6）
+0x58 -> 0x183FC8C   另一个回调入口，不能当作 +0x1C8
+0x78 -> 0x183FDAC   刷新接口（+0x1C8）
+0x90 -> 0x183FC78   另一个回调入口
```

**判断：这些是 App/view 事件层回调，不是候选列表 provider。**

### 3.6 refresh context

`+0x1C8` 是刷新接口。

`0x18A1094` / `0x18A1E18` 会：

```asm
ldr x0, [x19, #0x1C8]
ldr x1, [x0]
ldr x2, [x1]
...
mov w1, #0x0201000C
blr x2
```

刷新路径：

```text
0x183FDAC -> 0x183FD30 -> 0x183FBD0
event 0x0201000C，24 字节
queue 0x005503E0
```

它与 PP 选择共用队列，但**不是 class 6**，也不经过选择事件的 `0x008F013A` 路径。

---

## 4. 候选缓存 `+0x2258 / +0x22B4`

### 4.1 已知

- `+0x2258`：候选 DWORD 缓存
- `+0x22B4`：候选数量
- `+0x22B0/+0x22B8`：当前选择/索引相关
- `0x18A0F38`：在最多 11 个缓存 DWORD 中查找当前值
- `0x18A1860`：读取选中项并发送事件
- `0x18A135C`：按 widget mode 和 cache count 切换显示模式

### 4.2 扫描结果

已做：

- 全 `.text` 精确立即数 `+0x2258` / `+0x22B4` 扫描
- 拆分偏移扫描：`+0x2000/+0x2B4`、`+0x2200/+0xB4`
- 直接 `STR` / `STP` / `MOV` 上下文检查
- 其他类同偏移写入排除

结果：

- 构造函数只做初始化
- 父 view 没有直接写
- 没有找到 PP 类的候选 count writer
- `0x323CFF8` 的 `str w0,[x19,#0x22B4]` 属于镜头信息序列模型，不是这个 widget

**结论：候选缓存 writer 仍未定位。**

---

## 5. `0x1185` 的真实语义

### 5.1 通用 resolver

```text
0x2EF3BFC   通用属性 resolver
```

它按 key/subkey 查回调，表基址：

```text
0x89BCF00
```

有 **1706 个调用点**，是通用属性 getter 分发器，不是 PPLUT 专用规则。

### 5.2 静态注册表

```text
注册函数：0x17A9E00
静态表：0x571FB38
大小：0x1038
条目：0x18 字节
条目数：173
```

条目格式：

```text
+0x00  key（如 0x3978）
+0x04  0
+0x08  0x50001
+0x10  回调函数指针
```

表中 keys 为 `0x3978…`，**没有 `0x1185`**。

### 5.3 唯一 `0x1185` 调用点

```text
0x3130110
```

正确上下文是连续的两个属性槽：

```asm
; 0x02CF0E95 -> 0x1185
0x31300f0: mov w0, #0x0e95
0x31300f4: movk w0, #0x2cf, lsl #16
0x31300f8: bl  0x2FBFE08
0x31300fc: tst w0, #0xff
0x3130100: b.eq 0x3130114
0x3130104: add x2, x21, #0xbd8
0x3130108: mov w1, w19
0x313010c: mov w0, #0x1185
0x3130110: bl  0x2EF3BFC

; 0x02CF04AE -> 0x1186
0x3130114: mov w0, #0x04ae
0x3130118: movk w0, #0x2cf, lsl #16
0x313011c: bl  0x2FBFE08
0x3130120: tst w0, #0xff
0x3130124: b.eq 0x3130138
0x3130128: add x2, x21, #0xbd8
0x313012c: mov w1, w19
0x3130130: mov w0, #0x1186
0x3130134: bl  0x2EF3BFC
```

后续还有 `0x1191`、`0x119c`、`0x11a7`… 按 PP 参数递增。

**结论：`0x1185` 是 PP 参数 getter 的 resolver key，不是产品门控。**

---

## 6. 其他已排除路径

### 6.1 PP 值表

```text
表：0x37D64D8
PP1–11 = 1..11
OFF = 0
PPLUT1–4 = 0x0C–0x0F
```

读取者：

```text
0x185A160   取值 getter
0x185A1B4   调用者，PP 参数结构拷贝
```

**结论：只用于参数取值，不是菜单候选生成。**

### 6.2 主 PP 枚举表

```text
描述符：0x5D570E8
表：0x57F99C8
```

读取者：

```text
0x37297BC   通用字符串/查表 helper
```

**结论：不是候选列表 provider。**

### 6.3 PPLUT 文案 ID

```text
0x1E00A499
```

在以下 AP 中都没有直接 `u32` 引用：

- A7M4
- A7C2
- FX2

**结论：菜单文案不是裸文本 ID 直接引用，不能靠搜 ID 定位门控。**

### 6.4 RTTI 对照

名称：

```text
24view_list_pictureprofile
```

A7M4 / A7C2 都有，但它位于**连续名称表**，不是标准 Itanium RTTI：

- 没有 raw 指针
- 没有 relocation 指向它
- 无法自动解析 typeinfo/vtable

**结论：不能靠 RTTI 自动做跨机型 vtable 对照。**

---

## 7. 已排除 / 未解决

### 已排除

| 路径 | 结论 |
|---|---|
| 构造函数直接初始化 | 只清空/初始化，不填候选 |
| 父 view 直接写入 | 未发现 |
| App 事件/refresh 回调 | 是事件层，不是 provider |
| `0x1185` resolver | PP 参数 getter，不是产品门控 |
| PP 值表 `0x37D64D8` | 参数取值，不是候选生成 |
| PP 枚举表 `0x57F99C8` | 通用查表，不是候选生成 |
| PPLUT 文本 ID | 三机型均无直接引用 |
| RTTI 自动对照 | 名称表不是 RTTI |
| 拆分偏移写入 | 未发现 PP 类 writer |

### 未解决

1. `view_list_pictureprofile` 候选缓存 `+0x2258/+0x22B4` 的写入者
2. 候选 provider / 行有效性规则
3. PP 选择事件到属性 setter 的提交路径
4. PPLUT 在 LS 产品类下的最小可用门控

---

## 8. 为什么没有补丁

要提出补丁，至少需要证明：

```text
候选 provider
  → 候选缓存写入
  → 每行 PPLUT 有效性
  → 选中事件
  → PP 属性/LUT 提交
```

目前只确认了：

- widget 构造/显示路径
- App 事件回调层
- PP 参数 getter
- 候选缓存读取

缺少最关键的 **provider → cache writer** 证据。

因此不能安全地给出：

- 改哪个字节
- 改了之后菜单是否真的出现
- 选中后是否真的应用 LUT
- 回滚是否完整

**所以没有补丁，也没有刷机。**

---

## 9. 动态验证环境现状

已安装：

- WSL2 + Ubuntu 24.04
- QEMU 11.1.2
- Sony QEMU patch
- A7M4 SCU 模型

已验证：

```text
QEMU emulator version 11.1.2
```

但运行 A7M4 UI 还需要：

- `nflasha7`
- `nflasha3`
- `nflasha15`
- `nflasha8`
- `firmware.tar`

当前只有 `nflasha15`、`nflasha8`。

公开 `fwtool.py` 对官方 6.02 DAT 报：

```text
No decrypter found
```

所以无法从官方 DAT 生成缺失分区。

---

## 10. 继续所需

任选其一：

1. **Fastboot 全分区备份中的 `nflasha7`、`nflasha3`**
   - 复制文件即可，不需要在备份电脑上运行 QEMU

2. **已解密的 `firmware.tar` / 分区**
   - 或 A7M4 6.02 可用的 decrypter

3. **现成 QEMU 运行镜像**

拿到后可以继续：

- 启动原厂 UI
- 在 `view_list_pictureprofile+0x2258/+0x22B4` 下写观察点
- 找候选 provider
- 验证行有效性和提交路径
- 再讨论最小补丁

---

## 11. 安全与边界

- 本文只包含静态分析结论和地址线索，不包含固件二进制、分区数据、密钥或设备标识。
- 地址和键值只是研究线索，不是可直接应用的补丁说明。
- 没有进行相机 I/O、属性写入、固件修改或刷机。
- 相机保持官方 6.02。
- 任何后续补丁都必须：
  - 有明确证据链
  - 可回滚
  - 先做 Fastboot 全分区备份
  - 经用户明确确认后才实机写入

---

## 附录：外部 TC 静态对照（简要）

- A7M4 固件中存在外部 TC 代码：
  - `sequence_sys_set_timecode_set_ext_tcin`
  - `sequence_sys_set_timecode_set_tcin_accessory_connection_det`
  - `EXT_TC_SIGNAL_INPUTSIGNAL`
  - `EXT_TC_LOCK_STATUS_*`
- FX2 也有同类代码，二者高度相似。
- 未找到类似 PPLUT 的明确 `TYPE_LS` 产品排除规则。
- 因此外部 TC 更像**配件检测/硬件链路门控**，不是简单菜单开关。
- C23 接的是相机侧边 **Multi-Terminal**，不是 MI 热靴。
- Deity 官方兼容列表：FX3、FX30、A1（fw 2.01+）、A7S III（fw 3.0+）；不含 A7M4。
