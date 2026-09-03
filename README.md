# STM32 单节锂电池 BMS 原型

这是一个基于 STM32F103C8T6 的单节锂电池管理系统原型，用于展示电池电压采样、SOC 电量估算和 OLED 实时显示。

项目定位是学习型 BMS 原型，不是车规量产 BMS。它验证了 BMS 的核心流程：

```text
电池电压采样 -> ADC 转换 -> SOC 估算 -> OLED 显示 -> 状态判断
```

## 成果展示

### 实物整体

![BMS demo full view](docs/images/bms-demo-full.jpg)

### OLED 显示结果

![OLED result](docs/images/oled-result.jpg)

当前显示结果：

```text
1S Li BMS
V: 3.32V
SOC: 000%
Status: OK
```

SOC 为 0% 是因为当前测试电池电压较低。给电池充电到 3.7V 以上后，SOC 会显示更高百分比。

## 硬件清单

| 硬件 | 用途 |
|---|---|
| STM32F103C8T6 Blue Pill | 主控 |
| 0.96 英寸 OLED I2C 屏 | 显示电压、SOC 和状态 |
| 18650 锂电池 + 电池座 | 被测电池 |
| 10k 电阻 x2 | 电池电压分压 |
| 面包板和杜邦线 | 快速搭建原型 |

## 引脚连接

### OLED

| OLED | STM32 |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | PB8 |
| SDA | PB9 |

### 电池电压采样

电池电压不能直接接入 STM32 ADC。STM32 的 ADC 输入范围是 0 到 3.3V，而单节锂电池满电约 4.2V，所以使用两个 10k 电阻分压。

```text
电池正极 -> 10k R1 -> 采样点 -> 10k R2 -> GND
采样点 -> PA0/A0
电池负极 -> GND
```

接线示意：

![Wiring diagram](docs/images/wiring-diagram.png)

## 软件功能

- 使用 PA0/A0 读取电池分压后的 ADC 数值。
- 将 ADC 数值换算为真实电池电压。
- 通过 OCV-SOC 查表法估算电池电量百分比。
- OLED 实时显示电压、SOC 和状态。
- 根据电压判断状态：

| 电压范围 | 显示状态 |
|---|---|
| >= 4.18V | FULL |
| <= 3.20V | LOW |
| 其他 | OK |

## 核心代码

主程序位于：

```text
src/User/main.c
```

完整 Keil 工程位于：

```text
keil/Project.uvprojx
```

如果需要复现实验，可以直接用 Keil 打开 `keil/Project.uvprojx`，编译后下载到 STM32F103C8T6。

核心计算：

```c
Voltage = ADValue * 660 / 4095;
```

这里使用 `660` 是因为两个 10k 电阻把电池电压分成了一半，程序中需要乘回 2 倍。

SOC 估算采用分段查表：

```c
if (Voltage >= 420) return 100;
if (Voltage >= 410) return 95;
if (Voltage >= 400) return 90;
if (Voltage >= 392) return 80;
if (Voltage >= 385) return 70;
if (Voltage >= 380) return 60;
if (Voltage >= 375) return 50;
if (Voltage >= 370) return 40;
if (Voltage >= 365) return 20;
if (Voltage >= 360) return 10;
if (Voltage >= 350) return 5;
return 0;
```

## 项目亮点

- 将锂电池的电压采样、SOC 估算和状态判断整合到一个可运行的 STM32 原型中。
- 使用电阻分压解决 ADC 输入电压限制问题。
- 用 OLED 做实时显示，便于现场展示。
- 项目可以继续扩展温度采样、继电器保护、多串电芯采样、CAN 通信和均衡控制。

## 面试说明

可以这样介绍：

```text
我做了一个基于 STM32F103 的单节锂电池 BMS 原型。
硬件上使用两个 10k 电阻对电池电压分压，PA0 读取 ADC 采样值。
软件上将 ADC 值还原成真实电池电压，并用 OCV-SOC 查表法估算电量。
OLED 会实时显示电压、SOC 和状态。
当前电池电压是 3.32V，所以 SOC 接近 0%，系统状态为 OK。
后续可以扩展到多串电芯采样、温度管理、继电器保护和 CAN 通信。
```
