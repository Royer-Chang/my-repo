# CoreS3 Thread BR + ENV III + PM2.5 完整範例程式

> 檔案建議：`cores3_thread_br_env3_pm25_full_example.ino`  
> 用途：展示智慧家庭最小可運作版本（溫濕度 + PM2.5 + 告警邏輯 + 簡單輸出控制）

## 功能包含
- M5CoreS3 初始化
- ENV III（SHT30）讀取溫濕度（I2C）
- PMSx003（PMSA003I）讀取 PM1.0/PM2.5/PM10（UART）
- 螢幕顯示即時資料
- PM2.5 告警邏輯（超過閾值觸發）
- LED / Buzzer / Relay 的示範控制
- 基本 EMA 平滑濾波

## 相依套件
- `M5CoreS3`（或 M5Unified 對應 CoreS3）
- `Adafruit SHT31 Library`

> ENV III 常見溫濕度晶片為 SHT30/SHT31，相容此 library。

## 程式碼

```cpp
#include <M5CoreS3.h>
#include <Wire.h>
#include <Adafruit_SHT31.h>

// =========================
// Pin Mapping (CoreS3 Thread BR)
// =========================
static const int PIN_I2C_SDA = 21;
static const int PIN_I2C_SCL = 22;

static const int PIN_PM_RX   = 16; // UART2 RX <- PMS TX
static const int PIN_PM_TX   = 17; // UART2 TX -> PMS RX (optional)

static const int PIN_RELAY   = 32;
static const int PIN_BUZZER  = 33;
static const int PIN_LED     = 25;

// =========================
// PM2.5 sensor (PMSx003)
// =========================
HardwareSerial PMSerial(2);

struct PMSData {
  uint16_t pm1_0 = 0;
  uint16_t pm2_5 = 0;
  uint16_t pm10  = 0;
  bool valid = false;
};

PMSData pmsData;

// PMS frame is 32 bytes for PMS5003/PMSA003I
bool readPMSFrame(PMSData& out) {
  const uint8_t FRAME_LEN = 32;

  // Wait for header 0x42 0x4D
  while (PMSerial.available() >= FRAME_LEN) {
    if (PMSerial.peek() == 0x42) {
      uint8_t buffer[FRAME_LEN];
      PMSerial.readBytes(buffer, FRAME_LEN);

      if (buffer[0] != 0x42 || buffer[1] != 0x4D) {
        continue;
      }

      // checksum
      uint16_t sum = 0;
      for (int i = 0; i < 30; i++) sum += buffer[i];
      uint16_t checksum = (uint16_t(buffer[30]) << 8) | buffer[31];
      if (sum != checksum) {
        return false;
      }

      // Atmospheric environment values:
      // PM1.0: buffer[10..11], PM2.5: buffer[12..13], PM10: buffer[14..15]
      out.pm1_0 = (uint16_t(buffer[10]) << 8) | buffer[11];
      out.pm2_5 = (uint16_t(buffer[12]) << 8) | buffer[13];
      out.pm10  = (uint16_t(buffer[14]) << 8) | buffer[15];
      out.valid = true;
      return true;
    } else {
      PMSerial.read(); // discard one byte
    }
  }
  return false;
}

// =========================
// ENV III (SHT3x)
// =========================
Adafruit_SHT31 sht31 = Adafruit_SHT31(0x44);

// =========================
// App State
// =========================
float tempC = NAN;
float humi  = NAN;

// EMA filtered PM2.5
float pm25Filtered = 0.0f;
bool pm25Initialized = false;
const float EMA_ALPHA = 0.30f;

// Alert threshold
const float PM25_ALERT_THRESHOLD = 35.0f;

// Timing
unsigned long lastReadEnv = 0;
unsigned long lastReadPM  = 0;
unsigned long lastUI      = 0;

const unsigned long ENV_INTERVAL_MS = 2000;
const unsigned long PM_INTERVAL_MS  = 1000;
const unsigned long UI_INTERVAL_MS  = 1000;

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

const char* pmStatus(float pm25) {
  if (pm25 <= 15) return "Good";
  if (pm25 <= 35) return "Moderate";
  return "Poor";
}

void drawStaticUI() {
  M5.Display.fillScreen(BLACK);
  M5.Display.setTextColor(WHITE);
  M5.Display.setTextSize(2);
  M5.Display.setCursor(10, 10);
  M5.Display.println("HomeSense Demo");

  M5.Display.setTextSize(1);
  M5.Display.setCursor(10, 45);
  M5.Display.println("CoreS3 Thread BR + ENV III + PM2.5");
}

void drawDataUI() {
  M5.Display.fillRect(10, 70, 300, 150, BLACK);
  M5.Display.setTextColor(WHITE);
  M5.Display.setTextSize(2);
  M5.Display.setCursor(10, 70);
  M5.Display.printf("T: %.1f C\n", tempC);
  M5.Display.printf("H: %.1f %%\n", humi);
  M5.Display.printf("PM2.5: %.1f\n", pm25Filtered);

  M5.Display.setTextSize(1);
  M5.Display.setCursor(10, 150);
  M5.Display.printf("PM1.0: %u  PM10: %u", pmsData.pm1_0, pmsData.pm10);

  M5.Display.setCursor(10, 170);
  M5.Display.printf("Air: %s", pmStatus(pm25Filtered));

  M5.Display.setCursor(10, 190);
  M5.Display.println("Touch to toggle Relay");
}

void applyAlertLogic() {
  bool alert = (pm25Initialized && pm25Filtered > PM25_ALERT_THRESHOLD);

  if (alert) {
    setLed(true);
    setBuzzer(true);
    setRelay(true); // demo: auto turn on fan relay when PM2.5 high
  } else {
    setLed(false);
    setBuzzer(false);
    // relay stays user/manual controlled if needed; here we auto-off for demo:
    setRelay(false);
  }
}

void setup() {
  auto cfg = M5.config();
  CoreS3.begin(cfg);
  Serial.begin(115200);
  delay(200);

  // GPIO
  pinMode(PIN_RELAY, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_LED, OUTPUT);
  setRelay(false);
  setBuzzer(false);
  setLed(false);

  // I2C
  Wire.begin(PIN_I2C_SDA, PIN_I2C_SCL);

  // ENV sensor
  if (!sht31.begin(0x44)) {
    Serial.println("[ERR] SHT31 not found at 0x44");
  } else {
    Serial.println("[OK] SHT31 ready");
  }

  // PM UART
  PMSerial.begin(9600, SERIAL_8N1, PIN_PM_RX, PIN_PM_TX);
  Serial.println("[OK] PMS UART2 ready");

  drawStaticUI();
  drawDataUI();

  Serial.println("[BOOT] Demo started");
}

void loop() {
  M5.update();
  unsigned long now = millis();

  // Read ENV
  if (now - lastReadEnv >= ENV_INTERVAL_MS) {
    lastReadEnv = now;
    float t = sht31.readTemperature();
    float h = sht31.readHumidity();

    if (!isnan(t) && !isnan(h)) {
      tempC = t;
      humi  = h;
      Serial.printf("[ENV] T=%.2fC H=%.2f%%\n", tempC, humi);
    } else {
      Serial.println("[WARN] ENV read failed");
    }
  }

  // Read PM
  if (now - lastReadPM >= PM_INTERVAL_MS) {
    lastReadPM = now;
    PMSData tmp;
    if (readPMSFrame(tmp) && tmp.valid) {
      pmsData = tmp;

      float pm = (float)pmsData.pm2_5;
      if (!pm25Initialized) {
        pm25Filtered = pm;
        pm25Initialized = true;
      } else {
        pm25Filtered = (1.0f - EMA_ALPHA) * pm25Filtered + EMA_ALPHA * pm;
      }

      Serial.printf("[PM] PM1.0=%u PM2.5=%u PM10=%u Filtered=%.1f\n",
                    pmsData.pm1_0, pmsData.pm2_5, pmsData.pm10, pm25Filtered);
    } else {
      Serial.println("[WARN] PM frame not ready/invalid");
    }
  }

  // Alert logic
  applyAlertLogic();

  // UI update
  if (now - lastUI >= UI_INTERVAL_MS) {
    lastUI = now;
    drawDataUI();
  }

  // Touch: manual relay toggle for demo
  if (M5.Touch.getCount() > 0) {
    static bool relayManual = false;
    relayManual = !relayManual;
    setRelay(relayManual);
    Serial.printf("[TOUCH] Manual Relay=%s\n", relayManual ? "ON" : "OFF");
    delay(200); // debounce
  }
}
```

## 驗收方式
1. 開機後顯示 `HomeSense Demo` 畫面
2. Serial 可看到 `[ENV]` 與 `[PM]` 日誌
3. 螢幕顯示溫度、濕度、PM2.5、空氣等級
4. PM2.5 > 35 時，LED/Buzzer 觸發，Relay 自動 ON
5. 觸控螢幕可手動切換 Relay（Demo 用）

## 佈署注意事項
- PMSA003I 請確保 5V 供電穩定，並共地
- 若 SHT31 位址不是 `0x44`，可能是 `0x45`
- 若 PM 資料讀不到，先確認 UART TX/RX 是否接反
- 若 Buzzer 過吵，可把 `setBuzzer(true)` 改成短脈衝方式