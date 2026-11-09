# kicad_integrating_box_light_source

胶片翻拍用「积分盒」四色 LED 光源 —— KiCad 工程。

## 功能

- **R / G / B / IR 四个独立通道**，每通道 4 颗 3535 灯珠串联，恒流 333 mA
- 恒流驱动 **AL8862SP-13**（滞环降压，`I_OUT = 0.1 V / R_SET`），CTRL 端 0.4–2.5 V **DC（模拟）调光**
- 调光由 **MCP4728**（I2C 四路 12 位 DAC）给出，CH1–CH4 对应 R/G/B/IR
- 光强闭环：**BPW34 + OPA333 跨阻 + ADS1110**（16 位 ΔΣ ADC，同一 I2C 总线）
- 电源：24 V 输入 → 2 A 保险丝 + SMBJ30CA TVS + SS54 防反接 → **LM317 线性**产生 3V3（模拟电源用线性，避开开关噪声）

## 图纸结构（层次式）

| 图纸 | 内容 |
|---|---|
| `integrating_box_light_source.kicad_sch` | 顶层：J1(24V)、J2(I2C)、I2C 上拉、7 个层次图 |
| `power_protection.kicad_sch` | 24 V 输入防护 + LM317 3V3 |
| `led_cc_driver.kicad_sch` / `_g` / `_b` / `_ir` | 四路恒流驱动（红/绿/蓝/红外） |
| `dac_with_external_vref.kicad_sch` | MCP4728 四路 DAC |
| `light_sensor.kicad_sch` | 光强检测前端 |

每张图内都有中文的 **参数计算**、**LAYOUT 要点**（引用器件手册原文），红通道图上嵌有 AL8862 数据手册裁出的 SO-8EP 推荐焊盘图。

## 关键计算

| 项目 | 值 | 依据 |
|---|---|---|
| LED 电流（100%） | 333 mA | `I_OUT = 0.1 V / R_SET`，R_SET = R30 |
| 电感 | 100 µH | `t_OFF = L·ΔI/(V_LED+V_D+I·(R_SET+R_L))`；满载 570–700 kHz（手册推荐 33–100 µH，高压取大） |
| 输入电容 | 10 µF/50V | 手册推荐 4.7–10 µF，X7R |
| 3V3 | 3.28 V | `1.25×(1+390/240)`，TO-252，耗散 0.19 W |
| 跨阻增益 | 47 kΩ ∥ 10 pF | 带宽 339 kHz，噪声 ≈20 µV rms，折算 1.33 nA/LSB |

## 灯珠与立创编号

| 通道 | 位号 | 型号 | 立创编号 |
|---|---|---|---|
| 红 | LED_R1–R4 | XL-HD3535SURC-A2 | C3646952 |
| 绿 | LED_G1–G4 | JNJ-LEHG0113W120 | C19185875 |
| 蓝 | LED_B1–B4 | XL-HD3535UBC-A2 | C3646953 |
| 红外 | LED_IR1–IR4 | XL-HD3535HIR-T4A2 | C5221244 |
| 驱动 IC | U4–U7 | AL8862SP-13 | C526360 |

灯珠立创编号写在符号的 `LCSC Part` 字段里，`BOM_立创编号.csv` 由 KiCad 网表导出生成，可直接用于下单核对。

## 打开方式

KiCad 10 打开 `integrating_box_light_source.kicad_pro`。工程自带两个库表：
- `sym-lib-table` → `LCSC.kicad_sym`（AL8862 符号）
- `fp-lib-table` → `integrating_box_light_source.pretty`（自建 3535 灯珠封装）

## 状态

- 原理图：整机 ERC **0 error / 0 warning**
- PCB：已建板，布局进行中
- 尚未完成：其余阻容/连接器的立创编号待补齐
