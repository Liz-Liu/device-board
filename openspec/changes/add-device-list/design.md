# Design

## Context

目前 `api/` 是 .NET 10 的 ASP.NET Core 控制器式 API，於 `Program.cs` 註冊控制器，尚無裝置資料存取層或測試專案。需求及範圍見 proposal 和 device-list spec。

## Goals / Non-Goals

**Goals:**
- 將 HTTP 驗證與資料存取分離，使替換資料來源只需更換 `IDeviceService` 實作及其註冊。
- 在整個應用程式生命週期維持同一批啟動假資料，並可注入測試資料驗證邊界情況。

**Non-Goals:**
- 不設計持久化或資料庫遷移，不改變現有 WeatherForecast 端點。

## Decisions

1. 使用 `DevicesController` 提供明確的 `api/devices` 路由，延續現有控制器架構；不另引入 Minimal API。控制器檢查原始查詢參數是否存在，並要求單一且精確符合小寫合法值，避免空字串被模型繫結轉成 `null` 後誤判為未提供。重複、空白、大小寫不符及其他值均視為非法；這是目前未指定細節的約定。
2. 以 `IDeviceService` 提供裝置清單查詢，透過 DI 注入控制器；記憶體實作負責以有效狀態篩選。相較控制器直接持有靜態資料，此介面讓其他資料來源能獨立替換。
3. 註冊單例記憶體服務，並在應用程式啟動時初始化一次 20 筆資料，而非每次請求重新產生。每筆使用唯一數字 ID、有效狀態及 `DateTimeOffset`，以 JSON camelCase 和 ISO 8601 輸出。假資料僅供本機用途，重啟後不保證保留。
4. 使用 ASP.NET Core 的 `ProblemDetails` 回應非法查詢，保持狀態碼及內容一致；不使用自訂錯誤格式。以整合測試注入可控制的 `IDeviceService` 驗證篩選、空結果及錯誤，另驗證預設服務啟動資料與序列化。

## Risks / Trade-offs

- [假資料非持久化且重啟可能變動] → 視為暫時資料來源，後續以新服務實作取代。
- [單例服務若暴露可變集合可能被意外修改] → 只提供唯讀查詢結果，不公開可變儲存集合。
- [沒有現成測試專案] → 新增適用 .NET 10 的 API 測試專案，使用可替換服務驗證 HTTP 行為。

## Migration Plan

部署新版 API 即可啟用新端點，無需資料遷移；回滾至前一版會移除該端點，不影響既有端點或持久資料。