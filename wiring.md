# 接线说明

## OLED

```text
OLED VCC -> STM32 3.3V
OLED GND -> STM32 GND
OLED SCL -> STM32 PB8
OLED SDA -> STM32 PB9
```

## 电池采样

```text
电池正极 -> 10k R1 -> 采样点 -> 10k R2 -> GND
采样点 -> STM32 PA0/A0
电池负极 -> GND
```

注意：电池正极不能直接接 A0。

