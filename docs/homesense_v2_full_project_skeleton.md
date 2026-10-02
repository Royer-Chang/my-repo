# HomeSense v2：完整智慧家庭專案骨架  
**(CoreS3 Thread BR / include CO2 + VOC + UI + Alert logic)**

> 目標：提供可直接擴充的 firmware 架構，讓你從 demo 快速走向競賽可展示版本。  
> 主題功能：溫濕度 + PM2.5 + CO2 + VOC + UI + Alert + Relay 控制。  
> 平台：M5Stack CoreS3 Thread BR（Arduino / PlatformIO 可用）。

---

## 1) 專案目錄結構

```text
project/
├── src/
│   ├── main.cpp
│   ├── types.h
│   ├── config/
│   │   ├── pin_config.h
│   │   └── app_config.h
│   ├── utils/
│   │   ├── filters.h
│   │   └── filters.cpp
│   ├── sensors/
│   │   ├── env_sensor.h
│   │   ├── env_sensor.cpp
│   │   ├── pm_sensor.h
│   │   ├── pm_sensor.cpp
│   │   ├── co2_sensor.h
│   │   ├── co2_sensor.cpp
│   │   ├── voc_sensor.h
│   │   └── voc_sensor.cpp
│   ├── logic/
│   │   ├── comfort_logic.h
│   │   ├── comfort_logic.cpp
│   │   ├── alert_logic.h
│   │   └── alert_logic.cpp
│   ├── control/
│   │   ├── relay_manager.h
│   │   ├── relay_manager.cpp
│   │   ├── alarm_manager.h
│   │   └── alarm_manager.cpp
│   └── ui/
│       ├── ui_manager.h
│       └── ui_manager.cpp
├── platformio.ini
└── README.md
```

---

## 2) `platformio.ini`（CoreS3 Thread BR 基礎版）

```ini
[env:m5stack-cores3]
platform = espressif32
board = m5stack-cores3
framework = arduino
monitor_speed = 115200

lib_deps =
  m5stack/M5CoreS3
  adafruit/Adafruit SHT31 Library
  sparkfun/SparkFun SGP30 Arduino Library
```

> 若你改用 M5Unified 或不同 CO2/VOC 模組，`lib_deps` 會不同。

---

## 3) 共用型別與設定

### `src/types.h`

```cpp
#pragma once
#include <Arduino.h>

struct EnvironmentData {
  float temperatureC = NAN;
  float humidity = NAN;
  float pressure = NAN;

  uint16_t pm1_0 = 0;
  uint16_t pm2_5 = 0;
  uint16_t pm10 = 0;

  uint16_t co2ppm = 0;   // CO2
  uint16_t tvoc = 0;     // VOC (ppb)
  uint16_t eco2 = 0;     // equivalent CO2 from VOC sensor

  float comfortScore = 0.0f;
};

enum class AirLevel {
  Good,
  Moderate,
  Poor
};

struct AlertState {
  bool tempHigh = false;
  bool humidityHigh = false;
  bool pm25High = false;
  bool co2High = false;
  bool vocHigh = false;

  bool anyAlert = false;
  String message;
};
```

### `src/config/pin_config.h`

```cpp
#pragma once

// CoreS3 Thread BR pin mapping
static const int PIN_I2C_SDA = 21;
static const int PIN_I2C_SCL = 22;

// PM2.5 UART2
static const int PIN_PM_RX = 16;
static const int PIN_PM_TX = 17;

// CO2 UART1 (for MH-Z19B)
static const int PIN_CO2_RX = 26;
static const int PIN_CO2_TX = 14;

// Outputs
static const int PIN_RELAY  = 32;
static const int PIN_BUZZER = 33;
static const int PIN_LED    = 25;
```

### `src/config/app_config.h`

```cpp
#pragma once

// Sampling intervals
static const unsigned long ENV_INTERVAL_MS = 2000;
static const unsigned long PM_INTERVAL_MS  = 1000;
static const unsigned long CO2_INTERVAL_MS = 2000;
static const unsigned long VOC_INTERVAL_MS = 2000;
static const unsigned long UI_INTERVAL_MS  = 1000;

// Thresholds (可依需求調整)
static const float TH_TEMP_HIGH_C = 30.0f;
static const float TH_HUMI_HIGH   = 70.0f;
static const float TH_PM25_HIGH   = 35.0f;
static const uint16_t TH_CO2_HIGH = 1000;
static const uint16_t TH_TVOC_HIGH = 400;

// Filter
static const float EMA_ALPHA = 0.30f;
```

---

## 4) Utils：濾波

### `src/utils/filters.h`

```cpp
#pragma once
float ema(float prev, float current, float alpha);
```

### `src/utils/filters.cpp`

```cpp
#include "filters.h"
#include <math.h>

float ema(float prev, float current, float alpha) {
  if (isnan(prev)) return current;
  return prev * (1.0f - alpha) + current * alpha;
}
```

---

## 5) Sensors 模組

## 5.1 ENV（SHT31）

### `src/sensors/env_sensor.h`

```cpp
#pragma once
#include <Arduino.h>

class EnvSensor {
public:
  bool begin();
  bool update();
  float temperatureC() const { return t_; }
  float humidity() const { return h_; }

private:
  float t_ = NAN;
  float h_ = NAN;
  bool ok_ = false;
};
```

### `src/sensors/env_sensor.cpp`

```cpp
#include "env_sensor.h"
#include <Wire.h>
#include <Adafruit_SHT31.h>
#include "../config/pin_config.h"

static Adafruit_SHT31 sht31 = Adafruit_SHT31(0x44);

bool EnvSensor::begin() {
  Wire.begin(PIN_I2C_SDA, PIN_I2C_SCL);
  ok_ = sht31.begin(0x44);
  Serial.println(ok_ ? "[ENV] SHT31 ready" : "[ENV] SHT31 not found");
  return ok_;
}

bool EnvSensor::update() {
  if (!ok_) return false;
  float t = sht31.readTemperature();
  float h = sht31.readHumidity();
  if (isnan(t) || isnan(h)) return false;
  t_ = t;
  h_ = h;
  return true;
}
```

---

## 5.2 PM2.5（PMSA003I）

### `src/sensors/pm_sensor.h`

```cpp
#pragma once
#include <Arduino.h>

struct PMData {
  uint16_t pm1_0 = 0;
  uint16_t pm2_5 = 0;
  uint16_t pm10 = 0;
  bool valid = false;
};

class PMSensor {
public:
  bool begin(int8_t rxPin, int8_t txPin);
  bool update();
  PMData data() const { return data_; }

private:
  HardwareSerial serial_{2};
  PMData data_;
};
```

### `src/sensors/pm_sensor.cpp`

```cpp
#include "pm_sensor.h"

bool PMSensor::begin(int8_t rxPin, int8_t txPin) {
  serial_.begin(9600, SERIAL_8N1, rxPin, txPin);
  Serial.println("[PM] UART ready");
  return true;
}

bool PMSensor::update() {
  const uint8_t LEN = 32;

  while (serial_.available() >= LEN) {
    if (serial_.peek() != 0x42) {
      serial_.read();
      continue;
    }

    uint8_t buf[LEN];
    serial_.readBytes(buf, LEN);

    if (buf[0] != 0x42 || buf[1] != 0x4D) continue;

    uint16_t sum = 0;
    for (int i = 0; i < 30; i++) sum += buf[i];
    uint16_t checksum = (uint16_t(buf[30]) << 8) | buf[31];
    if (sum != checksum) return false;

    data_.pm1_0 = (uint16_t(buf[10]) << 8) | buf[11];
    data_.pm2_5 = (uint16_t(buf[12]) << 8) | buf[13];
    data_.pm10  = (uint16_t(buf[14]) << 8) | buf[15];
    data_.valid = true;
    return true;
  }
  return false;
}
```

---

## 5.3 CO2（MH-Z19B, UART）

### `src/sensors/co2_sensor.h`

```cpp
#pragma once
#include <Arduino.h>

class CO2Sensor {
public:
  bool begin(int8_t rxPin, int8_t txPin);
  bool update();
  uint16_t co2ppm() const { return ppm_; }

private:
  HardwareSerial serial_{1};
  uint16_t ppm_ = 0;
  bool ok_ = false;
};
```

### `src/sensors/co2_sensor.cpp`

```cpp
#include "co2_sensor.h"

// MH-Z19B command: FF 01 86 00 00 00 00 00 79
static const uint8_t CMD_READ_CO2[9] = {0xFF,0x01,0x86,0,0,0,0,0,0x79};

bool CO2Sensor::begin(int8_t rxPin, int8_t txPin) {
  serial_.begin(9600, SERIAL_8N1, rxPin, txPin);
  ok_ = true;
  Serial.println("[CO2] MH-Z19B UART ready");
  return true;
}

bool CO2Sensor::update() {
  if (!ok_) return false;

  while (serial_.available()) serial_.read(); // clear stale

  serial_.write(CMD_READ_CO2, 9);
  delay(20);

  uint8_t resp[9];
  if (serial_.readBytes(resp, 9) != 9) return false;
  if (resp[0] != 0xFF || resp[1] != 0x86) return false;

  ppm_ = (uint16_t(resp[2]) * 256) + resp[3];
  return true;
}
```

---

## 5.4 VOC（SGP30, I2C）

### `src/sensors/voc_sensor.h`

```cpp
#pragma once
#include <Arduino.h>

class VOCSensor {
public:
  bool begin();
  bool update();
  uint16_t tvoc() const { return tvoc_; }
  uint16_t eco2() const { return eco2_; }

private:
  uint16_t tvoc_ = 0;
  uint16_t eco2_ = 0;
  bool ok_ = false;
};
```

### `src/sensors/voc_sensor.cpp`

```cpp
#include "voc_sensor.h"
#include <Wire.h>
#include <SparkFun_SGP30_Arduino_Library.h>
#include "../config/pin_config.h"

static SGP30 sgp30;

bool VOCSensor::begin() {
  Wire.begin(PIN_I2C_SDA, PIN_I2C_SCL);
  ok_ = sgp30.begin();
  Serial.println(ok_ ? "[VOC] SGP30 ready" : "[VOC] SGP30 not found");
  return ok_;
}

bool VOCSensor::update() {
  if (!ok_) return false;
  if (!sgp30.measureAirQuality()) return false;
  tvoc_ = sgp30.TVOC;
  eco2_ = sgp30.CO2;
  return true;
}
```

---

## 6) Logic 模組

## 6.1 Comfort logic

### `src/logic/comfort_logic.h`

```cpp
#pragma once
#include "../types.h"

class ComfortLogic {
public:
  static float computeScore(const EnvironmentData& d);
  static AirLevel classifyAir(const EnvironmentData& d);
};
```

### `src/logic/comfort_logic.cpp`

```cpp
#include "comfort_logic.h"

float ComfortLogic::computeScore(const EnvironmentData& d) {
  // 簡化分數（0~100）
  float score = 100.0f;

  if (!isnan(d.temperatureC)) {
    if (d.temperatureC > 30) score -= 20;
    else if (d.temperatureC < 18) score -= 10;
  }

  if (!isnan(d.humidity)) {
    if (d.humidity > 70) score -= 20;
    else if (d.humidity < 30) score -= 10;
  }

  if (d.pm2_5 > 35) score -= 20;
  if (d.co2ppm > 1000) score -= 20;
  if (d.tvoc > 400) score -= 15;

  if (score < 0) score = 0;
  return score;
}

AirLevel ComfortLogic::classifyAir(const EnvironmentData& d) {
  if (d.pm2_5 <= 15 && d.co2ppm <= 800 && d.tvoc <= 220) return AirLevel::Good;
  if (d.pm2_5 <= 35 && d.co2ppm <= 1000 && d.tvoc <= 400) return AirLevel::Moderate;
  return AirLevel::Poor;
}
```

## 6.2 Alert logic

### `src/logic/alert_logic.h`

```cpp
#pragma once
#include "../types.h"

class AlertLogic {
public:
  static AlertState evaluate(const EnvironmentData& d);
};
```

### `src/logic/alert_logic.cpp`

```cpp
#include "alert_logic.h"
#include "../config/app_config.h"

AlertState AlertLogic::evaluate(const EnvironmentData& d) {
  AlertState a;
  a.tempHigh = (!isnan(d.temperatureC) && d.temperatureC > TH_TEMP_HIGH_C);
  a.humidityHigh = (!isnan(d.humidity) && d.humidity > TH_HUMI_HIGH);
  a.pm25High = (d.pm2_5 > TH_PM25_HIGH);
  a.co2High = (d.co2ppm > TH_CO2_HIGH);
  a.vocHigh = (d.tvoc > TH_TVOC_HIGH);

  a.anyAlert = a.tempHigh || a.humidityHigh || a.pm25High || a.co2High || a.vocHigh;

  if (!a.anyAlert) {
    a.message = "Air normal, no alert.";
    return a;
  }

  String msg = "Alert: ";
  if (a.pm25High) msg += "PM2.5 high; ";
  if (a.co2High) msg += "CO2 high; ";
  if (a.vocHigh) msg += "VOC high; ";
  if (a.tempHigh) msg += "Temp high; ";
  if (a.humidityHigh) msg += "Humidity high; ";
  a.message = msg;
  return a;
}
```

---

## 7) Control 模組

### `src/control/relay_manager.h`

```cpp
#pragma once
#include <Arduino.h>

class RelayManager {
public:
  explicit RelayManager(int pin): pin_(pin) {}
  void begin();
  void set(bool on);
  bool state() const { return on_; }

private:
  int pin_;
  bool on_ = false;
};
```

### `src/control/relay_manager.cpp`

```cpp
#include "relay_manager.h"

void RelayManager::begin() {
  pinMode(pin_, OUTPUT);
  set(false);
}
void RelayManager::set(bool on) {
  on_ = on;
  digitalWrite(pin_, on ? HIGH : LOW);
}
```

### `src/control/alarm_manager.h`

```cpp
#pragma once
#include <Arduino.h>

class AlarmManager {
public:
  AlarmManager(int buzzerPin, int ledPin): buzzerPin_(buzzerPin), ledPin_(ledPin) {}
  void begin();
  void setAlert(bool on);

private:
  int buzzerPin_;
  int ledPin_;
};
```

### `src/control/alarm_manager.cpp`

```cpp
#include "alarm_manager.h"

void AlarmManager::begin() {
  pinMode(buzzerPin_, OUTPUT);
  pinMode(ledPin_, OUTPUT);
  setAlert(false);
}
void AlarmManager::setAlert(bool on) {
  digitalWrite(ledPin_, on ? HIGH : LOW);
  digitalWrite(buzzerPin_, on ? HIGH : LOW);
}
```

---

## 8) UI 模組

### `src/ui/ui_manager.h`

```cpp
#pragma once
#include "../types.h"

class UIManager {
public:
  void begin();
  void render(const EnvironmentData& d, const AlertState& a, bool relayOn);
};
```

### `src/ui/ui_manager.cpp`

```cpp
#include "ui_manager.h"
#include <M5CoreS3.h>

static const char* airLevelText(AirLevel lv) {
  switch (lv) {
    case AirLevel::Good: return "Good";
    case AirLevel::Moderate: return "Moderate";
    case AirLevel::Poor: return "Poor";
  }
  return "Unknown";
}

void UIManager::begin() {
  M5.Display.fillScreen(BLACK);
  M5.Display.setTextColor(WHITE);
  M5.Display.setTextSize(2);
  M5.Display.setCursor(10, 10);
  M5.Display.println("HomeSense v2");
  M5.Display.setTextSize(1);
}

void UIManager::render(const EnvironmentData& d, const AlertState& a, bool relayOn) {
  M5.Display.fillRect(0, 40, 320, 200, BLACK);
  M5.Display.setCursor(10, 45);
  M5.Display.printf("Temp: %.1f C\n", d.temperatureC);
  M5.Display.printf("Humi: %.1f %%\n", d.humidity);
  M5.Display.printf("PM2.5: %u  CO2: %u\n", d.pm2_5, d.co2ppm);
  M5.Display.printf("TVOC: %u  eCO2: %u\n", d.tvoc, d.eco2);
  M5.Display.printf("Comfort: %.1f\n", d.comfortScore);

  M5.Display.printf("Relay: %s\n", relayOn ? "ON" : "OFF");

  M5.Display.setTextColor(a.anyAlert ? RED : GREEN);
  M5.Display.printf("%s\n", a.message.c_str());
  M5.Display.setTextColor(WHITE);
}
```

---

## 9) `main.cpp`（整合主程式）

```cpp
#include <M5CoreS3.h>
#include "types.h"
#include "config/pin_config.h"
#include "config/app_config.h"
#include "utils/filters.h"

#include "sensors/env_sensor.h"
#include "sensors/pm_sensor.h"
#include "sensors/co2_sensor.h"
#include "sensors/voc_sensor.h"

#include "logic/comfort_logic.h"
#include "logic/alert_logic.h"

#include "control/relay_manager.h"
#include "control/alarm_manager.h"

#include "ui/ui_manager.h"

// Modules
EnvSensor env;
PMSensor pm;
CO2Sensor co2;
VOCSensor voc;

RelayManager relay(PIN_RELAY);
AlarmManager alarm(PIN_BUZZER, PIN_LED);
UIManager ui;

// State
EnvironmentData data;
AlertState alert;

// Timers
unsigned long tEnv = 0, tPM = 0, tCO2 = 0, tVOC = 0, tUI = 0;

// filtered PM2.5
float pm25Filtered = NAN;

void setup() {
  auto cfg = M5.config();
  CoreS3.begin(cfg);
  Serial.begin(115200);
  delay(200);

  env.begin();
  pm.begin(PIN_PM_RX, PIN_PM_TX);
  co2.begin(PIN_CO2_RX, PIN_CO2_TX);
  voc.begin();

  relay.begin();
  alarm.begin();
  ui.begin();

  Serial.println("[MAIN] HomeSense v2 start");
}

void loop() {
  M5.update();
  unsigned long now = millis();

  // ENV
  if (now - tEnv >= ENV_INTERVAL_MS) {
    tEnv = now;
    if (env.update()) {
      data.temperatureC = env.temperatureC();
      data.humidity = env.humidity();
    }
  }

  // PM
  if (now - tPM >= PM_INTERVAL_MS) {
    tPM = now;
    if (pm.update()) {
      auto p = pm.data();
      data.pm1_0 = p.pm1_0;
      data.pm10 = p.pm10;
      pm25Filtered = ema(pm25Filtered, (float)p.pm2_5, EMA_ALPHA);
      data.pm2_5 = (uint16_t)pm25Filtered;
    }
  }

  // CO2
  if (now - tCO2 >= CO2_INTERVAL_MS) {
    tCO2 = now;
    if (co2.update()) {
      data.co2ppm = co2.co2ppm();
    }
  }

  // VOC
  if (now - tVOC >= VOC_INTERVAL_MS) {
    tVOC = now;
    if (voc.update()) {
      data.tvoc = voc.tvoc();
      data.eco2 = voc.eco2();
    }
  }

  // Logic
  data.comfortScore = ComfortLogic::computeScore(data);
  alert = AlertLogic::evaluate(data);

  // Actuation policy:
  // if PM2.5 or CO2 high => turn on relay (e.g. fan)
  bool autoRelay = alert.pm25High || alert.co2High;
  relay.set(autoRelay);
  alarm.setAlert(alert.anyAlert);

  // Touch override demo (optional)
  if (M5.Touch.getCount() > 0) {
    relay.set(!relay.state());
    delay(150);
  }

  // UI
  if (now - tUI >= UI_INTERVAL_MS) {
    tUI = now;
    ui.render(data, alert, relay.state());
  }
}
```

---

## 10) 開發與驗收清單（v2）

- [ ] 能讀到溫濕度（ENV）
- [ ] 能讀到 PM2.5（PMSA003I）
- [ ] 能讀到 CO2（MH-Z19B）
- [ ] 能讀到 TVOC/eCO2（SGP30）
- [ ] UI 每秒刷新且不閃爍嚴重
- [ ] 告警觸發時 LED/Buzzer 反應
- [ ] PM2.5/CO2 高時 Relay 自動開啟
- [ ] 無感測器時不當機（可逐步拔插測試）

---

## 11) 你可以馬上做的下一步

1. 先只上 `ENV + PM2.5 + UI`，確認主流程  
2. 再加 `CO2`，測試告警觸發  
3. 最後加 `VOC`，完成完整評分與告警文案  
4. 若時間夠，再加 MQTT（加分）

---

## 12) 備註

- 不同感測器實際 I2C 位址與 UART 行為可能略有差異，第一次上板請先逐模組驗證。  
- 若 Thread BR 的擴充底板有固定接口，優先用官方 Unit/Grove 連接可降低接線風險。  
- 本骨架是「競賽可落地」版本：先穩定可跑，再做漂亮與進階功能。
