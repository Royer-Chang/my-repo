# CoreS3 Thread BR Arduino 初始化骨架程式

> 檔案建議：`cores3_thread_br_arduino_skeleton.ino`  
> 用途：快速驗證 CoreS3 Thread BR 的 I2C/UART/GPIO 初始化、主迴圈節奏、基本 UI 輸出。

## 功能包含
- M5CoreS3 初始化
- I2C 初始化（SDA=21, SCL=22）
- UART2 初始化（PM2.5 預留）
- Relay / Buzzer / LED GPIO 初始化
- 週期性 heartbeat log
- 簡易螢幕狀態顯示

## 程式碼

```cpp
#include <M5CoreS3.h>
#include <Wire.h>

// =========================
// Pin Mapping (CoreS3 Thread BR)
// =========================
static const int PIN_I2C_SDA = 21;
static const int PIN_I2C_SCL = 22;

static const int PIN_PM_RX   = 16; // UART2 RX <- PM sensor TX
static const int PIN_PM_TX   = 17; // UART2 TX -> PM sensor RX (optional)

static const int PIN_RELAY   = 32;
static const int PIN_BUZZER  = 33;
static const int PIN_LED     = 25;

// =========================
// UART instance for PM sensor
// =========================
HardwareSerial PMSerial(2);

// =========================
// Timing
// =========================
unsigned long lastHeartbeat = 0;
const unsigned long HEARTBEAT_INTERVAL_MS = 1000;

// =========================
// Helpers
// =========================
void setRelay(bool on) {
  digitalWrite(PIN_RELAY, on ? HIGH : LOW);
}

void setBuzzer(bool on) {
  digitalWrite(PIN_BUZZER, on ? HIGH : LOW);
}

void setLed(bool on) {
  digitalWrite(PIN_LED, on ? HIGH : LOW);
}

void drawStatusScreen(const char* line1, const char* line2) {
  M5.Display.fillScreen(BLACK);
  M5.Display.setTextColor(WHITE);
  M5.Display.setTextSize(2);
  M5.Display.setCursor(20, 30);
  M5.Display.println("CoreS3 Thread BR");
  M5.Display.setTextSize(1);
  M5.Display.setCursor(20, 70);
  M5.Display.println(line1);
  M5.Display.setCursor(20, 90);
  M5.Display.println(line2);
}

void setup() {
  auto cfg = M5.config();
  CoreS3.begin(cfg);

  Serial.begin(115200);
  delay(200);

  // GPIO init
  pinMode(PIN_RELAY, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_LED, OUTPUT);

  setRelay(false);
  setBuzzer(false);
  setLed(false);

  // I2C init
  Wire.begin(PIN_I2C_SDA, PIN_I2C_SCL);

  // UART2 init for PM sensor
  PMSerial.begin(9600, SERIAL_8N1, PIN_PM_RX, PIN_PM_TX);

  drawStatusScreen("Init done", "I2C/UART/GPIO ready");

  Serial.println("[BOOT] CoreS3 Thread BR skeleton init complete");
  Serial.printf("[I2C] SDA=%d SCL=%d\n", PIN_I2C_SDA, PIN_I2C_SCL);
  Serial.printf("[UART2] RX=%d TX=%d\n", PIN_PM_RX, PIN_PM_TX);
  Serial.printf("[GPIO] RELAY=%d BUZZER=%d LED=%d\n", PIN_RELAY, PIN_BUZZER, PIN_LED);
}

void loop() {
  M5.update();

  unsigned long now = millis();
  if (now - lastHeartbeat >= HEARTBEAT_INTERVAL_MS) {
    lastHeartbeat = now;

    static bool ledState = false;
    ledState = !ledState;
    setLed(ledState);

    Serial.printf("[HEARTBEAT] uptime=%lu ms, LED=%s\n", now, ledState ? "ON" : "OFF");

    M5.Display.fillRect(20, 120, 280, 40, BLACK);
    M5.Display.setCursor(20, 120);
    M5.Display.printf("Uptime: %lu ms", now);
    M5.Display.setCursor(20, 140);
    M5.Display.printf("LED: %s", ledState ? "ON" : "OFF");
  }

  // Demo touch area: tap screen to toggle relay
  if (M5.Touch.getCount() > 0) {
    static bool relayState = false;
    relayState = !relayState;
    setRelay(relayState);

    Serial.printf("[TOUCH] Relay toggled: %s\n", relayState ? "ON" : "OFF");

    M5.Display.fillRect(20, 170, 280, 30, BLACK);
    M5.Display.setCursor(20, 170);
    M5.Display.printf("Relay: %s", relayState ? "ON" : "OFF");

    delay(200); // basic debounce
  }
}
```

## 驗收方式
1. 上電後螢幕顯示 `Init done`
2. Serial 每秒輸出 heartbeat
3. 螢幕觸控可切換 Relay 狀態
4. LED 每秒閃爍