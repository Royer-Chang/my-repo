# A. M5Stack CoreS3 硬體接線圖 / 接腳配置

> 目標：以 CoreS3 為主控，保持與 Core2 相容的感測器與控制邏輯設計，適合進行智慧家庭 demo。  
> 這版偏向「穩定、可實作、可展示」的接線方式。

---

## A-1. CoreS3 建議硬體組合（穩定版）

- 主控：M5Stack CoreS3
- I2C 感測：
  - ENV III（溫度 / 濕度 / 氣壓）
  - SGP30（VOC / eCO2）
  - BH1750（光照）
- UART 感測：
  - PMSA003I（PM2.5）
  - MH-Z19B（CO2，可選）
- 控制輸出：
  - 1CH Relay（燈 / 風扇）
- 告警：
  - Buzzer（可選）
  - LED（可選）

---

## A-2. 接線拓撲（CoreS3 版）

```text
                +----------------------+
                |   M5Stack CoreS3     |
                |      (ESP32-S3)      |
                |                      |
                | I2C SDA (GPIO21) ----+---- ENV III
                | I2C SCL (GPIO22) ----+---- SGP30
                |                      |---- BH1750
                |                      |
                | UART2 RX (GPIO16) <--+---- PMSA003I TX
                | UART2 TX (GPIO17) ---+---- PMSA003I RX (optional)
                |                      |
                | UART1 RX (GPIO26) <--+---- MH-Z19B TX (optional)
                | UART1 TX (GPIO14) ---+---- MH-Z19B RX (optional)
                |                      |
                | GPIO32 (OUT) --------+---- Relay IN
                | GPIO33 (OUT) --------+---- Buzzer IN (optional)
                |                      |
                | 5V ------------------+---- Relay VCC / PMSA003I VCC
                | 3V3 -----------------+---- I2C sensors VCC (if needed)
                | GND -----------------+---- All module GND
                +----------------------+