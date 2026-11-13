# Proposal

## Why

目前 API 無法提供裝置清單，使用者也無法依裝置狀態查詢。需要一個可替換資料來源的唯讀 API，先以記憶體假資料支援開發與驗證。

## What Changes

- 新增 `GET /api/devices`，回傳裝置欄位並支援可選的狀態篩選。
- 對不合法的 `status` 回傳 HTTP 400 與 ASP.NET Core ProblemDetails。
- 啟動時建立 20 台記憶體假裝置，並透過 `IDeviceService` 隔離資料存取。
- 不包含新增、修改、刪除裝置、登入或前端畫面。

## Capabilities

### New Capabilities

- `device-list`: 提供裝置清單、狀態篩選及查詢錯誤回應。

### Modified Capabilities

無。

## Impact

- 影響 `api/` 的控制器、裝置模型、服務註冊與資料存取實作。
- 新增唯讀 HTTP API 及其測試；既有 WeatherForecast API 不變。