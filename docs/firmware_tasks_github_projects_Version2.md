# Firmware 任務管理表（GitHub Projects / Notion）

## 欄位定義
- **任務名稱**
- **狀態**
- **優先級**
- **預估工時**
- **依賴關係**
- **驗收條件**

## 任務表

| 任務名稱 | 狀態 | 優先級 | 預估工時 | 依賴關係 | 驗收條件 |
|---|---|---|---:|---|---|
| 建立 PlatformIO/Arduino 專案 | Todo | 高 | 1.0 | 無 | 專案可成功編譯與燒錄 |
| 初始化 M5Stack Core2 執行環境 | Todo | 高 | 1.5 | 建立 PlatformIO/Arduino 專案 | 開機畫面可正常顯示，主迴圈穩定執行 |
| 建立 `pin_config.h` 與 `sensor_config.h` | Todo | 高 | 1.0 | 建立 PlatformIO/Arduino 專案 | 所有必要的接腳與常數已定義並可重複使用 |
| 實作 logger / debug 輸出 | Todo | 中 | 1.0 | 初始化 M5Stack Core2 執行環境 | 序列埠可輸出感測值與系統狀態 |
| 建立 UI 假資料模式 | Todo | 中 | 1.0 | 初始化 M5Stack Core2 執行環境 | 沒有實體感測器時 UI 仍可正常運作 |
| 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert） | Todo | 高 | 2.0 | 初始化 M5Stack Core2 執行環境 | 頁面切換可穩定運作 |
| 硬體接線與供電穩定性驗證 | Todo | 高 | 2.0 | 建立 `pin_config.h` 與 `sensor_config.h` | 所有模組可正常通電，且共地無重啟問題 |
| 整合 ENV III（溫度/濕度/氣壓） | Todo | 高 | 2.0 | 硬體接線與供電穩定性驗證 | 即時數值可每 1–2 秒更新一次 |
| 整合 PM2.5 感測器（PMSA003I） | Todo | 高 | 3.0 | 硬體接線與供電穩定性驗證 | PM2.5 數值可正確解析並顯示 |
| 整合 VOC 感測器（SGP30/CCS811） | Todo | 高 | 3.0 | 硬體接線與供電穩定性驗證 | VOC/eCO2 數值可穩定讀取 |
| 整合 CO2 感測器（SCD30/MH-Z19B，可選） | Todo | 中 | 3.0 | 硬體接線與供電穩定性驗證 | CO2 ppm 數值可正確顯示 |
| 整合 BH1750 光照感測器 | Todo | 中 | 1.5 | 硬體接線與供電穩定性驗證 | Lux 數值可顯示並即時更新 |
| 加入感測資料平滑濾波（EMA） | Todo | 高 | 2.0 | 整合 ENV III（溫度/濕度/氣壓） | 顯示數值更穩定，抖動減少 |
| 加入感測超時 / 錯誤處理 | Todo | 高 | 2.0 | 整合 PM2.5 感測器（PMSA003I） | 感測器異常時系統不當機，並顯示 fallback 狀態 |
| 定義健康判斷門檻 | Todo | 高 | 2.0 | 整合 ENV III（溫度/濕度/氣壓）；整合 PM2.5 感測器（PMSA003I）；整合 VOC 感測器（SGP30/CCS811） | 系統可穩定輸出 Good / Moderate / Poor |
| 實作舒適度分數引擎 | Todo | 高 | 3.0 | 定義健康判斷門檻；整合 BH1750 光照感測器 | 舒適度分數可被計算並顯示於 UI |
| 實作空氣品質評估引擎 | Todo | 高 | 2.0 | 定義健康判斷門檻；整合 CO2 感測器（SCD30/MH-Z19B，可選） | AQ 狀態與建議文字可正確更新 |
| 定義節能提醒規則 | Todo | 中 | 2.5 | 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert） | 針對長時間開啟設備與夜間照明的提醒規則已實作 |
| 實作告警狀態機 | Todo | 高 | 3.0 | 實作舒適度分數引擎；實作空氣品質評估引擎 | 告警可依條件觸發與解除，且狀態轉移明確 |
| 實作告警訊息傳遞流程 | Todo | 中 | 1.5 | 實作告警狀態機 | 告警文字可正確顯示於 Alert 頁面與 log |
| Relay GPIO 基本控制測試 | Todo | 高 | 2.0 | 硬體接線與供電穩定性驗證 | Relay 可由 firmware 安全切換 |
| 實作設備控制邏輯（燈/風扇） | Todo | 高 | 2.0 | Relay GPIO 基本控制測試 | 設備狀態可切換並維持在執行期 |
| 將控制狀態同步到 UI | Todo | 高 | 2.0 | 實作設備控制邏輯（燈/風扇）；建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert） | UI 按鈕狀態與硬體狀態一致 |
| 加入防誤觸機制（debounce / cooldown） | Todo | 中 | 1.5 | 實作設備控制邏輯（燈/風扇） | 連續點擊不會造成錯誤切換 |
| 控制動作歷史紀錄 | Todo | 低 | 1.0 | 實作 logger / debug 輸出 | 控制事件可寫入時間戳 log |
| 實作 Home 總覽頁面 | Todo | 高 | 3.0 | 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert）；整合 ENV III（溫度/濕度/氣壓） | Home 頁可顯示核心環境數據與系統狀態 |
| 實作 Air Quality 頁面 | Todo | 高 | 2.5 | 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert）；實作空氣品質評估引擎 | Air 頁可顯示 PM2.5/VOC/CO2 與 AQ 狀態 |
| 實作 Energy 頁面 | Todo | 中 | 2.5 | 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert）；定義節能提醒規則 | 可顯示節能建議與長時間開啟提醒 |
| 實作 Control 頁面 | Todo | 高 | 2.5 | 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert）；將控制狀態同步到 UI | 可在控制頁面切換硬體設備並更新狀態 |
| 實作 Alert 頁面 | Todo | 高 | 2.0 | 建立基礎 UI 導航骨架（Home/Air/Energy/Control/Alert）；實作告警訊息傳遞流程 | 可列出目前啟用中的告警與處置建議 |
| UI 回應速度與流暢度優化 | Todo | 中 | 2.0 | 實作 Home/Air/Energy/Control/Alert 頁面 | 頁面切換流暢且可讀性良好 |
| Wi-Fi 連線管理器 | Todo | 中 | 2.0 | 初始化 M5Stack Core2 執行環境 | AP 中斷後可自動重連 |
| MQTT 發送遙測資料 | Todo | 低 | 3.0 | Wi-Fi 連線管理器 | 遙測 topic 可成功發送 |
| MQTT 訂閱遠端控制命令 | Todo | 低 | 3.0 | MQTT 發送遙測資料；實作設備控制邏輯（燈/風扇） | 遠端命令可成功觸發本地動作 |
| 端到端流程整合測試 | Todo | 高 | 3.0 | 實作 Home/Air/Energy/Control/Alert 頁面 | 完整流程可順利執行且不阻塞或當機 |
| 長時間穩定性測試（2–4 小時） | Todo | 高 | 4.0 | 端到端流程整合測試 | 系統可長時間穩定執行，不重啟、不當機 |
| 撰寫 3–5 分鐘 Demo 腳本 | Todo | 高 | 2.0 | 端到端流程整合測試 | 腳本涵蓋感測、警示、控制與價值說明 |
| 最終除錯與 Demo 修飾 | Todo | 高 | 4.0 | 長時間穩定性測試（2–4 小時）；撰寫 3–5 分鐘 Demo 腳本 | Demo 預演成功率 > 90% |
| 發布 v1.0 Demo Firmware | Todo | 高 | 1.5 | 最終除錯與 Demo 修飾 | 韌體映像 / tag / release notes 已準備完成 |