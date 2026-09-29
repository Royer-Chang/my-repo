# M5Stack CoreS3 Thread BR 硬體接線圖 / 接腳配置

> 目標：以 M5Stack CoreS3 Thread BR 為主控，完成智慧家庭 demo。  
> 這份方案以「穩定、容易 debug、適合競賽展示」為優先，並保留 M5Stack CoreS3 通用的 GPIO 設計邏輯。

---

## 1. 基本前提

M5Stack CoreS3 Thread BR 大多數情況屬於 ESP32-S3 類型主控，且與 CoreS3 的通用平台邏輯相近。  
因此，在實作智慧家庭感測器與控制模組時，最穩定的設計方式是：

- I2C 感測器：走 `GPIO21 / GPIO22`
- UART 感測器：走 `GPIO16 / GPIO17` 或 `GPIO26 / GPIO14`
- Relay / Buzzer / LED：走獨立 GPIO
- 若使用 M5Stack 專屬 shield / Grove 擴充板：優先使用官方接口，不直接裸接高壓設備

這樣可以降低接線錯誤與賽場突發狀況。

---

## 2. 建議硬體組合（穩定版）

- 主控：M5Stack CoreS3 Thread BR
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

## 3. 接線拓撲（文字版）

```text
                +--------------------------------------+
                |     M5Stack CoreS3 Thread BR        |
                |             (ESP32-S3)               |
                |                                      |
                | I2C SDA (GPIO21) -------+---- ENV III
                | I2C SCL (GPIO22) -------+---- SGP30
                |                         |---- BH1750
                |                         |
                | UART2 RX (GPIO16) <----+---- PMSA003I TX
                | UART2 TX (GPIO17) ----->+---- PMSA003I RX (optional)
                |                         |
                | UART1 RX (GPIO26) <----+---- MH-Z19B TX (optional)
                | UART1 TX (GPIO14) ----->+---- MH-Z19B RX (optional)
                |                         |
                | GPIO32 (OUT) -----------+---- Relay IN
                | GPIO33 (OUT) -----------+---- Buzzer IN (optional)
                |                         |
                | 5V --------------------+---- Relay VCC / PMSA003I VCC
                | 3V3 -------------------+---- I2C sensors VCC (if needed)
                | GND -------------------+---- All module GND
                +--------------------------------------+