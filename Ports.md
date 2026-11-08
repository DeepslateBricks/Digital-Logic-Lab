# Xilinx EDA 创新实验教学系统引脚分配表

## 开关（SW，LVCMOS33）
| 信号 | 引脚 | 信号 | 引脚 | 信号 | 引脚 | 信号 | 引脚 |
|---|---|---|---|---|---|---|---|
| SW1 | N15 | SW5 | R16 | SW9 | U17 | SW13 | V18 |
| SW2 | P20 | SW6 | P15 | SW10 | U18 | SW14 | V19 |
| SW3 | P16 | SW7 | N17 | SW11 | P19 | SW15 | U20 |
| SW4 | R17 | SW8 | P17 | SW12 | R19 | SW16 | V20 |

## LED（LED1–LED16）
W20, Y22, Y21, U21, T21, W21, W22, T20, AB18, AA18, AA19, AB20, AA20, AA21, AB22, AB21

## 按键 FB1–FB12（LVCMOS15）
Y8, Y7, AA6, Y6, V7, W7, AB7, AB6, V8, V9, AA8, AB8

## 矩阵键盘（5×4）
- 行 KP_R0–R4：M20, N19, N18, M18, L18
- 列 KP_C0–C3：N20, L21, M21, K18

## 数码管
- 段选 SEG_A~G、DP：K17, J17, L16, K16, L14, L15, M15, M16
- 位选 SEG_H0–H7：L20, L19, L13, M13, K14, K13, J14, H14

## LED 点阵（16×16）
- 行 ROW0–ROW15：J16, F15, G17, E21, D21, E22, D22, G21, G22, G20, H20, H22, J22, K21, K22, M22
- 列 COL0–COL3：E13, E14, F13, F14

## 并口 LCD 1602
D0–D7：G13, J21, J20, H19, J19, G16, G15, G18；E=H13, R/W=J15, RS=H15

## VGA 接口
RED=H18, GREEN=N22, BLUE=K19, VS=H17, HS=V17

## TF 卡座
MISO=R14, MOSI=P14, SCK=N14, TF_CS=N13

## 扩展模块接口（40 针插座）
| 插座 | 引脚 | 插座 | 引脚 |
|---|---|---|---|
| 1 | VCC | 2 | VCC |
| 3 | F16 | 4 | E16 |
| 5 | E17 | 6 | D16 |
| 7 | C14 | 8 | B15 |
| 9 | C15 | 10 | B16 |
| 11 | A13 | 12 | A15 |
| 13 | A14 | 14 | A16 |
| 15 | D17 | 16 | B17 |
| 17 | C17 | 18 | B18 |
| 19/21 | GND | 20/22 | GND |
| 23 | E19 | 24 | C18 |
| 25 | D19 | 26 | C19 |
| 27 | B20 | 28 | F18 |
| 29 | A20 | 30 | E18 |
| 31 | F19 | 32 | A18 |
| 33 | F20 | 34 | A19 |
| 35 | C22 | 36 | D20 |
| 37 | B22 | 38 | C20 |
| 39 | VCC | 40 | VCC |

## 其它外设
| 模块 | 信号 → 引脚 |
|---|---|
| 直流电机 | Ctrl_A=Y19, Ctrl_B=Y18, SPEED=C13 |
| 步进电机 | A=V22, B=U22, C=T18, D=R18 |
| JTAG | TCK=V12, TDO=U13, TDI=R13, TMS=T13 |
| 时钟/复位 | SYS_CLK=W19 (50MHz), RESET=B13 |
| 蜂鸣器/三色LED | BUZZER=A21, WS2812B=B21 |
| UART | TXD=D15, RXD=D14 |
| RTC | SDA=M17, SCL=F21 |
| XADC | VN=M9, VP=L10 |

**XDC 约束语法示例**：
```tcl
set_property -dict {PACKAGE_PIN D19 IOSTANDARD LVCMOS33} [get_ports A]
```

⚠️ 注意：**仅按键（FB1–FB12）电平设为 LVCMOS15，其余均为 LVCMOS33**。