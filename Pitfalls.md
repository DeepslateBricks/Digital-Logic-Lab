# Pitfalls——数字逻辑实验踩坑记录与工作流参考

**本文档由 AI 生成，在工作流中仅供参考，不作为强制约束。**

---

## 零、定位 Vivado 软件位置（为直接调用做准备）

使用

```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* , HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Where-Object { $_.DisplayName -like "*Vivado*" } | Select-Object DisplayName, InstallLocation, UninstallString
```

定位 Vivado 安装位置，例如返回的安装位置为 `E:\Vivado`，则 Vivado 二进制文件可能位于：`E:\Vivado\Vivado\2023.2\bin`（示例）。

## 一、出板前检查清单（参考）

按顺序过一遍，多数返工可以避免：

- [ ] **仿真**：波形显示的是具体数值而非 `X` / `Z`（`X`/`Z` 通常意味着信号没驱动或端口没连）
- [ ] **xdc 每条约束都有 PACKAGE_PIN + IOSTANDARD**——缺 IOSTANDARD 会触发 DRC NSTD-1，直接禁止生成位流（GUI 里 I/O Ports 面板那列显示 `default (LVCMOS18)` 就是缺失的信号）
- [ ] **每个引脚都能在 `Ports.md` 里查到**——凭印象编引脚号是最常见的错误来源
- [ ] **xdc 里的端口名与 Verilog 端口名逐字一致**（含大小写、位宽写法 `{in1[3]}`）
- [ ] xdc 开头有 `CFGBVS VCCO` / `CONFIG_VOLTAGE 3.3` 两行（消除 CFGBVS-1 警告）
- [ ] 综合、实现全程无 ERROR；**bit 文件的修改时间晚于最后一次源文件/约束修改**（防止拿着旧检查点以为改过了）
- [ ] Hardware Manager → Auto Connect 能枚举到 `xc7a35t`，Program Device 有成功提示

---

## 二、工作流参考

### 1. 代码与仿真（实验前）

模块名 / Testbench 写法参考 SKILL.md 的示例。几个容易忽略的点：

- 仿真里 `reg` 驱动输入、`wire` 接输出；`$finish` 前把要覆盖的取值都遍历到（如 sel 的 000~111 全部八种）。
- 波形窗口显示数字而非 X/Z 才算仿真通过；用 Fit zoom 缩放到合适尺度再截图。
- 输出位宽不足会静默截断（如把 8 位结果赋给 4 位端口），综合警告里会提，值得扫一眼。

### 2. 引脚分配（实验中）

推荐直接以 `Ports.md` 为唯一引脚来源逐条写 xdc。骨架：

```tcl
set_property CFGBVS VCCO [current_design]
set_property CONFIG_VOLTAGE 3.3 [current_design]

set_property -dict {PACKAGE_PIN <引脚> IOSTANDARD LVCMOS33} [get_ports <端口名>]
# 总线端口写法：[get_ports {in1[3]}]
# LED / 数码管段选等直驱输出可加 DRIVE 12
```

- 开关输入、LED/段选输出一律 `LVCMOS33`；**仅按键 FB1–FB12 用 `LVCMOS15`**。
- 想清楚位序约定再动手：推荐"开关编号递增 ↔ 端口下标递减"（SW2=in1[3] 最高位），这样拨码读出的二进制与数值一致；写进报告前在表格里体现出来。

### 3. 下载前批量验证（可选，但很省时间）

不开 GUI 也能完整验证"代码 + 约束能否出位流"，用批处理模式建临时工程跑一遍：

```tcl
# check.tcl —— 用法: vivado -mode batch -source check.tcl
create_project -force chk ./chk_proj -part xc7a35tfgg484-2
add_files -norecurse <顶层.v>
add_files -fileset constrs_1 -norecurse <约束.xdc>
set_property top <顶层模块名> [current_fileset]
update_compile_order -fileset sources_1

launch_runs impl_1 -to_step write_bitstream -jobs 4
wait_on_run impl_1
puts "STATUS: [get_property STATUS [get_runs impl_1]]"

# 逐端口读回落位，与 Ports.md 对照
open_run impl_1
foreach p [get_ports] {
    puts "$p -> [get_property PACKAGE_PIN $p] / [get_property IOSTANDARD $p]"
}
```

`STATUS: write_bitstream Complete!` 且每个端口落位正确，再去实验台上下载，基本一次过。临时工程用完即删。

### 4. 下载（实验中）

Run Synthesis → Run Implementation → Generate Bitstream → Open Hardware Manager → Open Target → Auto Connect → Program Device。

- 术语是"下载"（下载到 FPGA），不是"上传"。
- **严禁带电插拔实验箱导线**（课件原话）。

---

## 三、踩坑记录（真实案例）

每条按「现象 → 原因 → 处理」记录，出处都是本学期实际发生过的。

### 1. 位流生成失败：NSTD-1

- **现象**：`write_bitstream ERROR`，日志里 `DRC NSTD-1: 15 out of 15 logical ports use I/O standard 'DEFAULT'`。
- **原因**：xdc 里只写了 `PACKAGE_PIN`，一条 `IOSTANDARD` 都没有。
- **处理**：每条约束补上 `IOSTANDARD LVCMOS33`（或按键 FB 用 LVCMOS15）。GUI 的 I/O Ports 面板里 I/O Std 列显示红色 `default (LVCMOS18)` 即为缺失项。

### 2. 改了 xdc，DRC 错误依旧

- **现象**：明明把 IOSTANDARD 补进 xdc 了，重新 Generate Bitstream 还是报老错误。
- **原因**：Vivado 复用了旧的实现检查点，没有真正重跑。
- **处理**：Design Runs 里 Reset `impl_1`（连带清掉旧 bitstream）再重新 Implement → Generate Bitstream。改任何源文件/约束后都建议核对 bit 文件时间戳。

### 3. xdc 端口名与代码不一致

- **现象**：xdc 里写 `[get_ports en]`，代码里的端口实际叫 `en_xhx`——约束不生效或综合报找不到端口。
- **原因**：两个文件分开改、不同步。
- **处理**：写完 xdc 逐条与模块端口声明对照；用上面 check.tcl 的端口读回，一眼就能看出哪个端口没落位。

### 4. 编造引脚号

- **现象**：约束里出现 `PACKAGE_PIN D0`，引脚表里根本没有这个脚。
- **原因**：凭记忆/猜测填引脚。
- **处理**：每个引脚先在 `Ports.md` 查到再写。引脚表是唯一事实来源。

### 5. "下载后全黑"——其实是没连上

- **现象**：位流正常生成、Program Device 走完，数码管一片黑；反复怀疑代码和约束，改了一堆没用的。
- **原因**：物理连接问题（板子没上电 / USB 插在 UART 口而不是 JTAG 口 / 线只供电不传数据）。
- **处理**：**失败时先检查物理设备是否连接成功**——电源灯、JTAG 口、Auto Connect 能否枚举到 `xc7a35t`、Program Device 是否真的弹出成功提示。"没下载进去"和"逻辑写错了"的现象一模一样，先排除前者。

### 6. 数码管显示异常的排查顺序

- 全黑：先查位选（每位需要驱动，共阳位选通常给高电平；本实验台位选经 74LVC245 同相缓冲，`DIR` 接 3.3V、`/OE` 接地）
- 字形错乱：查段码位序——约定 `F[7]→SEG_A`、…、`F[0]→SEG_DP`（段码 a~g 按 `{a,b,c,d,e,f,g}` 排列时，a 在最高位）
- 亮但不对：共阳极段码 0 亮 1 灭；小数点是 `dp` 取反后送 h 段
- 多位同显/串扰：本实验台 DG1、DG2 的段脚是并联的，两模块永远显示同样内容；想让各位显示不同数字需要位选扫描（分频 + 轮流选通），那是后续实验的事

### 7. 报告里的引脚号笔误

- **现象**：报告实验现象一节写了"拨动开关 SW9~SW11"，但 sel 实际接在 SW10~SW12——与同文档的引脚分配表自相矛盾。
- **处理**：引脚号写进报告（或 xdc、或任何地方）之前，对照 `Ports.md` 核一遍。这类错误很隐蔽，因为读者默认作者不会写错自己的引脚表。

### 8. 外部修改 xdc 被 GUI 覆盖

- **现象**：脚本/编辑器刚写入的 xdc，几分钟后变成 0 字节或内容回退。
- **原因**：Vivado GUI 里开着该文件的编辑标签，其空/旧缓冲区被保存回去。
- **处理**：用外部工具改 xdc 时，先在 Vivado 里关掉该文件标签（提示保存时选"否"）；或者干脆只在一边改。

### 9. 两个 Vivado 进程同时开一个工程

- **现象**：run 状态互相打架、文件被另一个实例覆盖。
- **处理**：任务管理器确认只有一个 `vivado.exe` 在操作当前工程；批处理验证用独立的临时工程目录，不碰 GUI 正开着的工程。

---

## 四、硬件速查（配合 Ports.md）

- **平台**：ZDXH-EDA7A35，FPGA 为 xc7a35tfgg484-2（Artix-7），50MHz 时钟 SYS_CLK=W19。
- **拨动开关 SW1–SW16**：上=1，下=0，带 10K 上拉。
- **LED1–LED16**：FPGA 引脚经 1K 电阻直驱，高电平点亮。
- **数码管**：动态扫描八段数码管模块，共阳极；段选/位选均经 74LVC245 同相缓冲（无反相）；DG1 与 DG2 段选并联。
- **电平标准**：仅按键 FB1–FB12 为 LVCMOS15，其余一律 LVCMOS33。
- **下载口**：JTAG（USB 线插 JTAG 口，不是 UART 口）。
