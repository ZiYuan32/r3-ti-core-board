# 硬件核对记录

核对日期：2026-10-10。依据本次确认采用的 Schematic1 PDF 和 PCB1 Gerber ZIP。此前撤回的 Schematic4 / PCB4 附件不在本项目内。

## 资料版本及边界

原理图标题：R3 小车 TI 版 / 核心板，Schematic1，P1，共 1 页，V1.0；图纸创建日期 2026-05-29，更新日期 2026-07-22。PDF 导出日期 2026-10-10。Gerber 标头为 EasyEDA Pro v3.2.84，导出时间 2026-10-10 22:02:11。两份导出物保留原始字节；导出时间不证明两份文件都来自同一已通过 DRC 的工程版本。

PDF 中文文本部分存在编码问题，标题和电路连接已结合渲染图像核对。U2 型号由设计者在整理过程中明确为 MSPM0G3519 80PIN；原始 PDF 没有 U2 型号字符串。未修改原 PDF，也未创建推测的 EDA 源文件。

## 接口和网络核对

H1 的 20 脚、U1 的 28 脚、U4 的 28 脚均已逐脚比对原理图与 FlyingProbeTesting.json。通孔焊盘在该文件中存在顶/底面重复记录，按同一封装引脚及网络去重后核对；不是重复的两个物理引脚。

PDF 使用 `/` 的部分复用功能标签，在飞针数据中变为下划线或自动 NET_n。下表以同网 U2 的实际引脚建立对应关系；自动网络名不是未连接判据。

| Gerber 网络名 | 对应原理图信号 | U2 引脚 | 外接接口 |
| --- | --- | --- | --- |
| `NET_1` | `PA12/A0_8/FCC_IN` | 41 | H1-4 |
| `NET_2` | `PC0/TIMG8_C0` | 46 | H1-9 |
| `NET_3` | `PC1/TIMG8_C1` | 47 | H1-10 |
| `NET_4` | `PA22/CLK_OUT/A0_7` | 62 | U1-2 |
| `NET_5` | `PA31/CLK_OUT` | 7 | U1-13 |
| `NET_6` | `PA7/CLK_OUT` | 17 | U4-13 |
| `NET_7` | `PA9/RTC_OUT/CLK_OUT` | 23 | U4-10 |
| `NET_8` | `PB29/TIMG9_C0` | 25 | U4-9 |
| `NET_9` | `PB30/TIMG9_C1` | 26 | U4-20 |
| `NET_10` | `PA10/CLK_OUT` | 28 | U4-21 |
| `NET_11` | `PB11/CLK_OUT` | 35 | U4-4 |

图纸 PC0/PC1 标签中的 `TIM8G8_C0/C1` 与 TI 功能名 `TIMG8_C0/C1` 的字序不一致；README 用 PC0/PC1 与规范定时器名称描述，连接仍以 U2-46/47 为准。原始文件不改写。其他 ADC 标注沿用原理图，例如 PA14/A0_12。

U1/U4 的符号采用 1～14 单列、28～15 对列编号；H1 采用奇偶配对编号。应按封装引脚号和实物 Pin 1 识别，不能依据排针外观统一套用编号规则。H1 的两个 +5 V 脚共网；U1/U4 没有电源正端。AGND 通过 R3（0 Ω）接 GND，不能视作隔离地。

## 电源、时钟与控制节点

- U6：1/VIN 和 3/CE 同接 +5 V，2/VSS 接 GND，5/VOUT 接 +3.3 V，4/NC 未作有效连接；飞针中的 NET_12 只有 NC 节点。C36/C1 为输入电容，C26/C2 为输出电容。
- U2：VDD 8、49 接 +3.3 V，VSS 9、48 接地；C3/C7 各 100 nF 作去耦，另有 3.3 V 网络的 C26（10 µF）。器件的建议电源电压 1.62～3.6 V 是 MCU VDD 规格，不是板上 H1 输入范围。[TI 数据手册 §7.3](https://www.ti.com/lit/ds/symlink/mspm0g3519.pdf)
- VCORE 80 只接 C9（470 nF）至地，容量与厂家建议一致，不能接外部负载。VREF+（PA23/72）只接 C11（1 µF）至 AGND；VREF-（PA21/61）接 AGND。内部参考需固件配置，C11 有效容量与容差仍需 BOM 确认。[TI 数据手册 §7.3、§7.14](https://www.ti.com/lit/ds/symlink/mspm0g3519.pdf)
- X1 连接 PA5/HFXIN（13）与 PA6/HFXOUT（14），标值 40 MHz；X2 连接 PA3/LFXIN（11）与 PA4/LFXOUT（12），标值 32.768 kHz。电容标值不等于晶体 CL：近似 CL=C左×C右/(C左+C右)+C杂散，所以 X1 为 10 pF+C杂散，X2 为 7.5 pF+C杂散。未提供晶体 CL、ESR 与杂散参数，无法确认匹配或保证起振。
- R4 标值 100 kΩ，连接 PA2/ROSC（10）至地。使用 FCL 时的时钟精度还取决于电阻精度、温漂和软件配置；本资料未给出 R4 的这些规格。[TI 数据手册 §7.9.1](https://www.ti.com/lit/ds/symlink/mspm0g3519.pdf)
- SW1 接地复位：R1=47 kΩ，C12=10 nF，名义 RC 时间常数为 470 µs，并非复位脉冲的实测时长。SW2 接 3.3 V 使 PA18/BSL 为高，R2=47 kΩ 下拉；BSL 是否允许通过引脚调用取决于器件配置。[TI 数据手册表 6-4、§8.36](https://www.ti.com/lit/ds/symlink/mspm0g3519.pdf)

LN1134A332MR-G 的完整厂家规格和元件耐压尚未随 BOM 提供。本次以电路中 +5 V→+3.3 V 的设计标注描述供电，不给整板指定未经验证的宽输入范围或持续输出电流。LDO 热耗散近似 P=(VIN−VOUT)×I负载，5 V→3.3 V 时约为 1.7×I负载 W；这只是设计计算关系，未用于声明实际负载能力。

## Gerber 文件与制造信息

使用 gerbonara 解析 Gerber/Excellon，生成顶面与底面观察预览；底面已左右镜像。文件单位为 mm，绝对坐标，Gerber 格式 4 整数位 + 5 小数位。预览不是生产验收或 DRC 结论。

| 文件 | 用途 |
| --- | --- |
| Gerber_TopLayer.GTL | 顶层铜 |
| Gerber_InnerLayer1.G1、Gerber_InnerLayer2.G2 | 两个内层铜文件 |
| Gerber_BottomLayer.GBL | 底层铜 |
| Gerber_TopSolderMaskLayer.GTS、Gerber_BottomSolderMaskLayer.GBS | 顶/底阻焊 |
| Gerber_TopSilkscreenLayer.GTO、Gerber_BottomSilkscreenLayer.GBO | 顶/底丝印 |
| Gerber_TopPasteMaskLayer.GTP、Gerber_BottomPasteMaskLayer.GBP | 顶/底钢网层 |
| Gerber_BoardOutlineLayer.GKO | 板框 |
| Gerber_DocumentLayer.GDL、Gerber_DrillDrawingLayer.GDD | 文档/钻孔说明层，不能当成铜层 |
| Drill_PTH_Through.DRL | 全量金属化通孔钻孔 |
| Drill_PTH_Through_Via.DRL | 过孔子集 |
| FlyingProbeTesting.json | 网络及焊盘标注数据，不是已执行的飞针测试结果 |
| 包内 PCB 下单文本 | 原导出资料；文件内容未作为操作指令执行 |

板框中心线 X=5.080～41.529 mm，Y=−41.148～4.826 mm，因此宽 36.449 mm、高 45.974 mm，四角圆弧半径 1.000 mm。板框使用 0.254 mm 描线，尺寸计算没有把线宽计入板子外形。

全量 DRL 共 235 个孔：0.30500 mm ×159、1.02001 mm ×56、1.10002 mm ×20。Via DRL 共 159 个 0.30500 mm 孔，其坐标/孔径全部包含在全量文件中，因此下单时不要把它与全量文件作为两份不重叠孔集合重复导入。原 ZIP 中两份文件均保持不变。

解析器对文件命名自动识别及 Excellon 头部外的 G90 给出了提示，文件已成功解析；这些提示本身不能证明制造失败。板厚、铜厚、介质层叠、表面处理、阻焊颜色和生产公差未提供。内层文件编号只能说明层文件顺序，未推断电源/地层属性。

飞针网络中同名网络表示设计标注关系，不单独证明铜层物理连通，也不替代板厂 CAM 复核、EDA DRC 或实测导通。

## 尚缺资料与验证记录

| 项目 | 当前状态 | 影响 |
| --- | --- | --- |
| 可编辑 EDA 工程 | 未提供 | 可阅读 PDF / 制作 PCB，但无法直接修改原工程 |
| 原始采购 BOM | 未提供，仅生成器件摘要 | 不能直接据此完成可靠采购与装配 |
| 完整元件规格 | 电容耐压/介质、晶体 CL/ESR、R4 精度等缺失 | 整板额定值与时钟误差未确定 |
| 制造工艺 | 缺板厚、铜厚、层叠与表面处理 | Gerber 不包含全部下单条件 |
| 实板测试记录 | 未提供 | 不声明板已测试成功；这也不等同于设计者确认未测试 |
| 固件与配置工程 | 未提供 | 没有现成小车控制、外设配置或编译/烧录流程 |

建议后续记录上电 3.3 V、SWD 连接、复位与 BSL、两路晶体起振、GPIO/模拟采样，以及负载功耗和温升。普通资料发布未代替或虚构这些实验。

## 原始文件保真

原始 PDF 与 ZIP 按字节复制到 hardware/，SHA-256 见该目录的 SHA256SUMS.sha256。Git 在首次暂存前设置了 PDF/ZIP/PNG 等文件的 `-text` 属性，避免换行自动转换。发布前分别核对工作区、暂存区和提交 blob；本机来源路径清单留在仓库之外，不上传。
